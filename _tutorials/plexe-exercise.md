---
layout: toclay
title: "Getting Started with Plexe"
description: "Command vehicles in a platoon, use inter-vehicle communication, and implement a leave-platoon maneuver in OMNeT++/Plexe."
tags: [Plexe, OMNeT++, SUMO, CACC, C++]
permalink: /tutorials/plexe-exercise/
plexewebsite: https://plexe.car2x.org
---

# Getting Started with Plexe
{: .no_toc }

This exercise is proposed to start getting acquainted with Plexe and its [API]({{ page.plexewebsite }}/api){:target="_blank"}.

In this exercise we want to *command* a vehicle in a platoon to perform an action, and also to communicate
with the other vehicles in the platoon. In particular, we want to send the commands necessary to
let the last vehicle *abandon* the platoon, leaving the formation, and then we want to inform all
remaining vehicles that the platoon formation has changed.

Solving this guided exercise you will learn:

- The Plexe project folder structure;
- How to change the behavior (e.g., speed) of a vehicle;
- How to use inter-vehicle communication in Plexe.

![Demo of the Getting Started Exercise]({{ site.url }}{{ site.baseurl }}/assets/res/gettingStartedPlexe.gif){: width="800" }

## Exercise outline

We will implement the solution of this exercise as a modification to the *Platooning[^1]* example, i.e.,
one of the [examples](https://github.com/michele-segata/plexe/tree/master/examples) made available by Plexe that you can run
issuing these commands from your terminal.

[^1]: The *Platooning* scenario is one of the Scenarios included in the [Platooning example](https://github.com/michele-segata/plexe/tree/master/examples/platooning) project of Plexe.

```bash
cd $HOME/src/plexe
source setenv
cd examples/platooning
plexe_run -u Cmdenv -c Platooning -r 2
```

The *Platooning* example shows a platoon made up of 8 cars controlled by a Cooperative Adaptive Cruise Controller (CACC) that travels on a straight piece of Highway for 60 seconds. While on travel, cars target an inter-vehicle distance of 5m.

We will customize this initial example proceeding according to this list of steps:

1. At some given instant of time, we want the last vehicle to start braking, so to safely increase its distance from the rest of the platoon before abandoning the formation. We will be satisfied when the gap will have grown from 5m to 15m.
2. While braking, we need a mechanism too keep monitoring the width of the gap, and stop braking when the last car has reached the target 15m safety-distance from the platoon.
3. When this safety requirement is satisfied, the last vehicle should turn off the Cooperative-ACC (CACC) and switch to a plain ACC, essentially detaching from the platoon.
4. This last vehicle should also inform the platoon leader that he has just left the current formation, the leader should propagate
this information back to all other vehicles.


## Step 1: Scenario customization

The [Plexe Documentation]({{ page.plexewebsite }}/documentation){:target="_blank"} illustrates the content of the directories used to organize the source code of Plexe.
We now focus on the [scenarios folder]({{ page.plexewebsite }}/documentation/#scenarios-folder){:target="_blank"}. In general, in every simulation the behavior of vehicles
depends on many factors, for example, on the cooperative-driving Application that the vehicle is running (check docs about [apps folder]({{ page.plexewebsite }}/documentation/#apps-folder){:target="_blank"}) or on the [mobility]({{ page.plexewebsite }}/documentation/#mobility-folder){:target="_blank"} properties of the vehicle as governed by SUMO. 
An experimenter may wish to add an external stimulus to further drive the behavior of a simulation, for example, in the *Sinusoidal*[^2] configuration of the [Platooning example](https://github.com/michele-segata/plexe/blob/plexe-3.1/examples/platooning){:target="_blank"} the car leading a platoon is forced to keep speeding and braking so to exhibit a sinusoidal speed pattern.
The Plexe folder organization suggests to code this external stimuli as *Scenarios*.

[^2]: The [first Plexe Tutorial]({{ page.plexewebsite }}/tutorial){:target="_blank"} teaches you how to run the *Sinusoidal* scenario.

In our exercise we want to start from the [SimpleScenario](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/scenarios/SimpleScenario.cc){:target="_blank"} used while selecting the `Platooning` config defined in the [omnetpp.ini](https://github.com/michele-segata/plexe/blob/plexe-3.1/examples/platooning/omnetpp.ini#L193-L197){:target="_blank"}.

```c++
#            TraCIScenarioManager parameters             #
*.manager.moduleType = "org.car2x.plexe.PlatoonCar"
*.manager.moduleName = "node"
...
[Config Platooning]
*.manager.command = "sumo-gui"
#use the base scenario
*.node[*].scenario_type = "SimpleScenario"
```


- The above piece of configuration says that the [TraCIScenarioManager](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/veins/modules/mobility/traci/TraCIScenarioManager.ned){:target="_blank"} will map all vehicles (aka nodes) to [PlatoonCar](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/PlatoonCar.ned#L34-L52){:target="_blank"} modules.
Furthermore, each node (`*.node[*]`) should contain a [SimpleScenario](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/scenarios/SimpleScenario.cc){:target="_blank"} as internal submodule.

![Illustration of a PlatoonCar module]({{ site.url }}{{ site.baseurl }}/assets/res/PlatoonCar.png){: width="200" }

<!--
```c++
module PlatoonCar {
    parameters:
        string scenario_type;
        string helper_type;
        string appl_type;
        string protocol_type;
    submodules:
        helper: <helper_type> like BasePositionHelper
        ...
        scenario: <scenario_type> like BaseScenario
        ...
        appl: <appl_type> like BaseApp
        ...
```

```c++
void SimpleScenario::initialize(int stage) {
    BaseScenario::initialize(stage);
    if (stage == 0)
        // get pointer to application
        appl = FindModule<BaseApp*>::findSubModule(getParentModule());
    if (stage == 2) {
        // average speed
        leaderSpeed = par("leaderSpeed").doubleValue() / 3.6;
        if (positionHelper->isLeader()) {
            // set base cruising speed
            plexeTraciVehicle->setCruiseControlDesiredSpeed(leaderSpeed);
        }
        else {
            // let the follower set a higher desired speed to stay connected
            // to the leader when it is accelerating
            plexeTraciVehicle->setCruiseControlDesiredSpeed(leaderSpeed + 10);
        ...
```
-->

We need to customize the C++ source code contained in [SimpleScenario.cc](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/scenarios/SimpleScenario.cc#L29-L49){:target="_blank"} to add an event, scheduled only for the last vehicle of the platoon, that should be fired at simulation-time 10s; that will trigger the start of the detaching procedure.

#### **Step 1 - Hint**
{: .no_toc }

```c++
void SimpleScenario::initialize(int stage)
[...]
if (positionHelper->isLeader()) {[...]}
        else {
            [...]
            plexeTraciVehicle->setCruiseControlDesiredSpeed(leaderSpeed + 10);
            // if we are the last vehicle, then schedule braking at time 10sec
            std::vector<int> pFormation = positionHelper->getPlatoonFormation();
            // if we are the last vehicle...
            if (positionHelper->getId() == pFormation[pFormation.size() - 1]) {
                // prepare self messages for scheduled operations
                startBraking = new cMessage("Start Braking now!");
                checkDistance = new cMessage("Check Distance now!");
                // ...then schedule Brake operation
                scheduleAt(10, startBraking);
[...]
```

```c++
void SimpleScenario::handleMessage(cMessage* msg)
{
    if (msg == startBraking) {
        // Increase CACC Constant Spacing (set it to 15m)
        plexeTraciVehicle->setCACCConstantSpacing(15.0);
        traciVehicle->setColor(TraCIColor(100, 100, 100, 255));
        // then start checking when we reach that 15m distance
        scheduleAt(simTime() + 0.1, checkDistance);
    }
```

- Beyond the *startBraking* event, we are initializing already also the *checkDistance* event to be used later for solving [Step 2]({{ page.plexewebsite }}/tutorial/example6/#step-2-monitoring-distance). The declaration of these `cMessage*` variables is not shown in the code snippets and should be added to
[SimpleScenario.h](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/scenarios/SimpleScenario.h){:target="_blank"}

- How do we identify the last car of the platoon? By asking information about the current formation of the platoon and checking if our vehicle ID matches with the one of the last vehicle in the formation. For retrieving this kind of information about the position of vehicles in a platoon it is always recommended to rely on the **Position Helper**, as we did with `positionHelper->getPlatoonFormation();`
The position helper is a submodule of each Platoon Car, so it is available in each car object, and its goal
is to offer platooning-related information such as the ID of the platoon leader or of the vehicle in front of us.
Check out the [*Utilities documentation*]({{ page.plexewebsite }}/documentation/#utilities-folder){:target="_blank"}, describing in more details the functionalities offered by the Position Helper/Manager classes.

- When the *startBraking* event is intercepted by the `handleMessagge()` implemented by `SimpleScenario.cc`, we do 3 operations:
    1. We Increase the CACC Constant Spacing (from 5m we set it to 15m) with `setCACCConstantSpacing()`, which
     is one of the many functions documented in the Plexe [API]({{ page.plexewebsite }}/api){:target="_blank"}.
    2. We change the Car color through TraCI, i.e., the [SUMO Traffic Control Interface](https://sumo.dlr.de/docs/TraCI.html){:target="_blank"} with `traciVehicle->setColor(...);`.
    The `traciVehicle` is a further submodule of each Platoon Car object, exposing most of the functionalities defined by the SUMO TraCI interface.
    3. We schedule, in just 0.1s from the current simulation time, a check of the distance from the front vehicle.

[`STEP 1, SOLUTION on GITLAB`](https://tests.ing.unibs.it/ghiro/plexe/-/commit/5160850a35e2024073cb4759a378e920fdbc37e4){:target="_blank"}

## Step 2: Monitoring distance

We concluded the last step by asking to the car that is abandoning the platoon to increase its distance from its front-vehicle to a value of 15m.
By issuing ```setCACCConstantSpacing(15.0)``` our car will start braking, and only after some *actuation time* the car will really be 15m away from the platoon.
How do we understand that this necessary *actuation time* has passed and that we have reached the desired safety-distance?

The solution is to keep checking our distance from the vehicle in front of us relying on ```getRadarMeasurements()```, one further function offered by the Plexe [API]({{ page.plexewebsite }}/api){:target="_blank"}. Notice that, as last operation of the solution to [Step 1]({{ 'plexe_exercise.html#step-1-scenario-customization' | relative_url }}), we started planning an operation of "distance-checking" by scheduling a self-message called `checkDistance`.

#### **Step 2 - Hint**
{: .no_toc }

```c++
void SimpleScenario::handleMessage(cMessage* msg)
{
    if (msg == startBraking) {[...]}
    else if (msg == checkDistance) {
        // Checking current distance with radar
        double distance = nan("nan"), relSpeed = nan("nan");
        plexeTraciVehicle->getRadarMeasurements(distance, relSpeed);
        LOG << "LEAVING VEHICLE now at: " << distance << " meters" << endl;
        if (distance > 14.9) {
            // We are almost at correct distance! Turn to ACC (i.e., abandon platoon...)
            plexeTraciVehicle->setActiveController(ACC);
            plexeTraciVehicle->setACCHeadwayTime(1.2);
            traciVehicle->setColor(TraCIColor(200, 200, 200, 255));
            // send abandon Platoon message to leader
            appl->sendAbandonMessage();
        }
        else scheduleAt(simTime() + 0.1, checkDistance);
    }
}
```

- In the ```handleMessage()``` function, whenever we handle a `checkDistance` message, we use the radar to save into the ```distance``` variable
the current value of our distance from the vehicle in front us, then:
    - if our distance is close to 15m (```> 14.9``` in the code), we do some operations to really abandon the platoon;
    - otherwise we understand that we are still not distant enough, so we reschedule a "distance-check" 0.1s after the current simulation-time
    with ```scheduleAt(simTime() + 0.1, checkDistance);``` 
- When we are actually ready to abandon the platoon, to stop using the Cooperative ACC we:
    1. Switch to a plain ACC with ```plexeTraciVehicle->setActiveController(ACC);```, we do not forget to also initialize a reasonable [Time-Headway](https://en.wikipedia.org/wiki/Headway){:target="_blank"} for an ACC equal to 1.2s;
    2. We send an **Abandon Message** using ```appl```, which represents one application installed in our PlatoonCar. NB: each [PlatoonCar](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/PlatoonCar.ned){:target="_blank"} contains an Application submodule instantiated into an object referenced by the already seen
    `appl` pointer.

[`STEP 2, SOLUTION ON GITLAB`](https://tests.ing.unibs.it/ghiro/plexe/-/commit/38e52356f7169b2880bef387386283a449ae4d24){:target="_blank"}

### Side note "Scenarios VS Apps"

Starting this exercise we commented about the fact that, in a simulation, environmental factors or external stimuli shall be coded as *Scenarios*.
Here instead we see for the first time the use of Applications, that we should consider as software able to exploit all the resources of a modern vehicle,
including radio equipment, to let the vehicle perform some automatic operations.
In *Plexe*, new applications should be coded into the
[apps folder]({{ page.plexewebsite }}/documentation/#apps-folder){:target="_blank"} as extensions to ```BaseApp```.
It is always a good idea to carefully read the [Plexe Documentation]({{ page.plexewebsite }}/documentation){:target="_blank"} that, for instance,
describes the BaseApp this way:

    apps/BaseApp: The aim of the application is mainly to pass wirelessly received data to the automated controllers
    (the application, in our case).
    In addition, the application takes care of logging mobility information such as speed, acceleration, distance, etc. 
    The idea is to have a base application that provides basic functionalities. 
    More sophisticated applications can inherit from this base application.

The provided [apps folder]({{ page.plexewebsite }}/documentation/#apps-folder) makes available to experimenters some sample applications, including:
- the 
[`GeneralPlatooningApp`](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/apps/GeneralPlatooningApp.cc){:target="_blank"},
which will be a source of inspiration to write our application code;
- and the [`SimplePlatooningApp`](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/apps/SimplePlatooningApp.cc){:target="_blank"}
 currently used by in our [`omnetpp.ini`](https://github.com/michele-segata/plexe/blob/plexe-3.1/examples/platooning/omnetpp.ini#L140-L144){:target="_blank"},
 which is a an empty App template not implementing any businesses logic. It is made available as an extension of the [BaseApp](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/apps/BaseApp.cc){:target="_blank"} precisely for the purpose of having a place where to start from for writing a new application.
 For our purpose, we will use it to develop the sending/handling routines for `AbandonPlatoon` messages.

It is time to draw a new action list of what we need to develop for our new application:

1. Define a **new Message Type** (i.e., `AbandonPlatoon.msg` message).
2. Implement, in our application, the **transmission-side** of `AbandonPlatoon` messages, namely, implement the `appl->sendAbandonMessage()` method we used in the solution of [Step 2](/tutorial/example6/#step-2-monitoring-distance).
    * This includes learning how to encapsulate packets inside [802.11p](https://en.wikipedia.org/wiki/IEEE_802.11p){:target="_blank"} frames.
    Check out the [Plexe Documentation about drivers]({{ page.plexewebsite }}/documentation/#driver-folder){:target="_blank"}.
3. Implement also the **receiver-side**.
4. *Bonus Track:* Plexe comes with many predefined messages, let's use them! At the end of the reception of an AbandonPlatoon message let the leader broadcast a `NewFormation` message to inform all remaining vehicle that the platoon formation has been updated.

## Step 3:  New Maneuver Message

#### **Step 3 - Hint**
{: .no_toc }

Create a file called `AbandonMessage.msg` under  `/src/plexe/messages` with this content:

```c++
import ManeuverMessage;
// Message sent by a vehicle to notify the leader that this vehicle left the platoon.
// NB: this message contains also all fields inherited from ManeuverMessage!
// e.g., vehicleId and platoonId 
packet AbandonPlatoon extends ManeuverMessage {
    string msgContent = "Goodbye I'm leaving";
}
```

The solution is straightforward, providing the definition of a new packet of type `AbandonPlatoon` which extends a `ManeuverMessage`, this last
is already available in Plexe and its definition is included in [src/plexe/messages/ManeuverMessage.msg](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/messages/ManeuverMessage.msg){:target="_blank"}

```c++
{% raw  %}cplusplus {{
    /* message type for maneuver messages */
    static const int MANEUVER_TYPE = 12347;
}}{% endraw %}
// General message for an arbitrary maneuver to holds common information.
// Only children of this message should be initialized.
packet ManeuverMessage {
    // id of the originator of this message
    int vehicleId;
    // id of the platoon this message is about
    int platoonId;
    // id of the destination of this message
    int destinationId;
    // sumo external id of the sender
    string externalId;
}
```

- It is most of the time recommended to define new packets as extension to Maneuver Messages, since information such as `vehicleId`, `platoonId` and `destinationId`
are necessary for the correct processing of packets... in 99% of the conceivable scenarios.
- NB: [ManeuverMessage.msg](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/messages/ManeuverMessage.msg){:target="_blank"} contains also the definition of `static const int MANEUVER_TYPE = 12347;`. Assigning to packets that share a similar purpose (in the present case, messages supporting some coordinated maneuver) a unique `'TYPE'` is a good practice to ease the filtering of packets at reception time.
Do not forget that, in Vehicular networks, transmissions are most of the time *broadcast*, so filtering packets can become extremely important.

Now that we have our definition of an `AbandonPlatoon` message, it is time to learn how to send it.
We want to put together all the business logic related to sending/receiving this kind of messages into a single application.
For this purpose we will customize the 
[SimplePlatooningApp](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/apps/SimplePlatooningApp.cc){:target="_blank"}.


<!--
**omnetpp.ini**
```ini
#   Application   #
*.node[*].appl_type = "SimplePlatooningApp"
```
-->

[`STEP 3, SOLUTION ON GITLAB`](https://tests.ing.unibs.it/ghiro/plexe/-/commit/dbba75790c2792ee36beea1c55a868336e844664){:target="_blank"}

## Step 4 - Develop a new Application

In Vehicular Networks, applications usually run on top of a minimal *MAC+PHY* networking stack,
leading to a typical *APP-MAC-PHY* 3-layers architecture which is simpler compared to a full, classic *TCP/IP* one.
As App developer we should take care of interfacing our app with the MAC-sublayer.
Check once more the modular description of a [PlatoonCar](https://github.com/michele-segata/plexe/blob/4a3779e9e3c3786f3a55c3102e3d97a762e4e309/src/plexe/PlatoonCar.ned){:target="_blank"}:
- you should notice an [`appl`](https://github.com/michele-segata/plexe/blob/4a3779e9e3c3786f3a55c3102e3d97a762e4e309/src/plexe/PlatoonCar.ned#L54){:target="_blank"} submodule, which represents an application run by the car;
- then there is also a [`prot`](https://github.com/michele-segata/plexe/blob/4a3779e9e3c3786f3a55c3102e3d97a762e4e309/src/plexe/PlatoonCar.ned#L59){:target="_blank"}
submodule, where `prot` stays for *"protocol"*, which is the module that represents the lower layers of the stack (MAC+PHY).
All messages arriving wirelessly to the car, if correctly decoded by the PHY are passed to the MAC, this last can report them
to applications interested on those messages. This last sentence illustrates the `APP-MAC-PHY` architecture adopting a bottom-up
perspective. Using instead a top-down one, an app that wants to send a message must submit its messages to `prot`, then `prot`
will take care of all the rest, i.e., encapsulating messages into Layer2-frames and transmit them.

Thanks to this architecture Plexe is able to support `multiplexing`, i.e., multiple applications can run together on a
single car, all using the same radio interface.
Coming back to our exercise, to develop our new application we will add some code to the already provided
[`SimplePlatooningApp`](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/apps/SimplePlatooningApp.cc){:target="_blank"},
where the connection between the *APP* and the *MAC* is already implemented in 
[`BaseApp.cc`](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/apps/BaseApp.cc#L69){:target="_blank"}.
In fact, `SimplePlatooningApp` is a child of the very important [**BaseApp**](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/apps/BaseApp.cc){:target="_blank"} class, so inherits from it the ability of exchanging data with the radio sublayer and to log mobility information as reported in the [apps Documentation]({{ page.plexewebsite }}/documentation/#apps-folder){:target="_blank"}.

Let's start defining some new methods inside 
[SimplePlatooningApp.h](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/apps/SimplePlatooningApp.h){:target="_blank"}

```c++
#include "plexe/messages/AbandonPlatoon_m.h"
class SimplePlatooningApp : public BaseApp {

public:
    SimplePlatooningApp() {}

    void sendAbandonMessage();
    virtual void sendUnicast(cPacket* msg, int destination);

protected:
    /** override from BaseApp */
    virtual void initialize(int stage) override;
    virtual void handleLowerMsg(cMessage* msg) override;

    BaseScenario* scenario;

private:
    AbandonPlatoon* createAbandonMessage();
    NewFormation* createNewFormationMessage(const std::vector<int>& newPlatoonFormation);
    void handleAbandonPlatoon(const AbandonPlatoon* msg);
    void handleNewFormation(const NewFormation* msg);
    void sendNewFormationToFollowers(const std::vector<int>& newPlatoonFormation);
};
```

Notice the name of the functions implemented by this class; these are either:
- send operations (`sendAbandonMessage`, `sendNewFormationToFollowers`...);
- receive operations, aka message-handling-routines (`handleAbandonPlatoon`, `handleNewFormation`, `handleLowerMsg`)
- auxiliary functions, in the present case helpers to create new packets.

Before moving to [Step 5 - Sending a Message](/tutorial/example6/#step-5-sending-a-message), we provide the template of the implementation of the
`SimplePlatooningApp`, that should be coded in the `SimplePlatooningApp.cc` file.

```c++
#include "plexe/apps/SimplePlatooningApp.h"
Define_Module(SimplePlatooningApp);

void SimplePlatooningApp::initialize(int stage){ /*TO BE IMPLEMENTED*/ }

AbandonPlatoon* SimplePlatooningApp::createAbandonMessage(){ /*TO BE IMPLEMENTED*/ }

NewFormation* SimplePlatooningApp::createNewFormationMessage(const std::vector<int>& newPlatoonFormation)
{ /*TO BE IMPLEMENTED*/ }

void SimplePlatooningApp::sendAbandonMessage(){ /*TO BE IMPLEMENTED*/ }

void SimplePlatooningApp::sendNewFormationToFollowers(const std::vector<int>& newPlatoonFormation)
{ /*TO BE IMPLEMENTED*/ }

void SimplePlatooningApp::sendUnicast(cPacket* msg, int destination)
{ /*TO BE IMPLEMENTED*/ }

void SimplePlatooningApp::handleAbandonPlatoon(const AbandonPlatoon* msg)
{ /*TO BE IMPLEMENTED*/ }

void SimplePlatooningApp::handleNewFormation(const NewFormation* msg)
{ /*TO BE IMPLEMENTED*/ }

void SimplePlatooningApp::handleLowerMsg(cMessage* msg){ /*TO BE IMPLEMENTED*/ }
```

[`STEP 4, SOLUTION ON GITLAB`](https://tests.ing.unibs.it/ghiro/plexe/-/commit/c9d331a42227e79f050c5e6c04ca7dd7d5c69258){:target="_blank"}

## Step 5 - Sending a Message

In order to send an `AbandonPlatoon` message we have to:

1. Create it;
2. Encapsulate it in a 802.11p frame and transmit it.

The `sendAbandonMessage()` is limited to / makes explicit these 2 operations:

```c++
#include "plexe/messages/AbandonPlatoon_m.h"

void SimplePlatooningApp::sendAbandonMessage() {
    getSimulation()->getActiveEnvir()->alert("Sending an abandon message");
    AbandonPlatoon* abmsg = createAbandonMessage();
    sendUnicast(abmsg, abmsg->getDestinationId());
}
```

- To work with the `AbandonMessages` don't forget to include the header file
generated automatically by the OMNeT++ compiler `#include "plexe/messages/AbandonPlatoon_m.h"`; this directive can go in the `SimplePlatooningApp.h` file.
- `getSimulation()...->alert("SOME TEXT")` is a piece code useful all the times that the user wants some text to be displayed in the OMNeT++ GUI pausing the simulation.
- `sendUnicast(abmsg, abmsg->getDestinationId());` abstracts the operation of encapsulating an Application Messages into a frame before transmission over the air.
The implementation we provide here is taken from the source code of Plexe as also implemented in 
[GeneralPlatooningApp.cc](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/apps/GeneralPlatooningApp.cc#L132-L145){:target="_blank"}.

Now let's see the implementation of `createAbandonMessage()` and of `sendUnicast()`:

```c++
AbandonPlatoon* SimplePlatooningApp::createAbandonMessage() {
    AbandonPlatoon* abmsg = new AbandonPlatoon();
    abmsg->setVehicleId(positionHelper->getId());
    abmsg->setPlatoonId(positionHelper->getPlatoonId());
    abmsg->setDestinationId(positionHelper->getLeaderId());
    abmsg->setExternalId(positionHelper->getExternalId().c_str());
    abmsg->setKind(MANEUVER_TYPE);
    return abmsg;
}
```

```c++
void SimplePlatooningApp::sendUnicast(cPacket* msg, int destination) {
    Enter_Method_Silent();
    take(msg);
    BaseFrame1609_4* frame = new BaseFrame1609_4("BaseFrame1609_4",
            msg->getKind());
    frame->setRecipientAddress(destination);
    frame->setChannelNumber(static_cast<int>(Channel::cch));
    frame->encapsulate(msg);
    // send unicast frames using 11p only
    PlexeInterfaceControlInfo* ctrl = new PlexeInterfaceControlInfo();
    ctrl->setInterfaces(PlexeRadioInterfaces::VEINS_11P);
    frame->setControlInfo(ctrl);
    sendDown(frame);
}
```

- The creation of a message like in `createAbandonMessage()` most of the time requires the initialization of the message's fields using setter methods.
In the present case we retrieve the proper values using, once more, the
    [Position Helper](https://plexe.car2x.org/documentation/#utilities-folder){:target="_blank"}.
    **NB** We also tag the created message with its `KIND` value, set to the constant `MANEUVER_TYPE`.
- The code in `sendUnicast(cPacket* msg, int destination)` takes care of:
    * Encapsulating messages in BaseFrames.
    * Preparing the use of the radio equipment, e.g., configuring the radio driver.
    * Pushing the frame down to the MAC layer.

For this last operation, i.e., pushing/pulling messagges downto/upto the application layer, the application must be correctly
initialized and bound to the MAC. This is done as shown in the `initialize()` function:

```c++
void SimplePlatooningApp::initialize(int stage) {
    BaseApp::initialize(stage);
    if (stage == 1) {
        // connect application to lower layer
        protocol->registerApplication(MANEUVER_TYPE, gate("lowerLayerIn"),
            gate("lowerLayerOut"), gate("lowerControlIn"),
            gate("lowerControlOut"));
        // register to the signal indicating failed unicast transmissions
        findHost()->subscribe(Mac1609_4::sigRetriesExceeded, this);
        scenario = FindModule<BaseScenario*>::findSubModule(getParentModule());
    }
}
```

The Plexe architecture demands the duty of implementing the exchange of messages between the APP and the MAC layer
to what is simply called the `protocol`. 
The Plexe [Documentation about Protocols]({{ page.plexewebsite }}/documentation/#protocols-folder){:target="_blank"}
is again a precious resource to understand the role of the protocol submodule available in each PlatoonCar object:

    - `protocols/BaseProtocol.*`: We implement a base protocol that takes care of loading parameters from configuration file,
    recording network statistics, and providing primitives for sending frames.
    This class defines a virtual messageReceived() method that inheriting classes can override
    to get notified about incoming data frames.
    In addition, BaseProtocol performs application multiplexing.
    Applications can register to BaseProtocol to obtain copies of received frames of a particular type.
    - `protocols/SimplePlatooningBeaconing.*`: This class extends the base protocol and implements a 
    classic periodic beaconing protocol sending a beacon every x milliseconds.

- In our [omnetpp.ini](https://github.com/michele-segata/plexe/blob/plexe-3.1/examples/platooning/omnetpp.ini#L149-L162){:target="_blank"}
we configure all vehicles to use a the [`SimplePlatooningBeaconing`](https://github.com/michele-segata/plexe/blob/plexe-3.1/examples/platooning/omnetpp.ini#L149-L162){:target="_blank"} as protocol submodule.

```ini
*.node[*].protocol_type = "SimplePlatooningBeaconing"
```

- As shown before, while initializing our *APP* with `SimplePlatooningApp::initialize()`, we take care of performing application multiplexing
as explained in the [Documentation about Protocols]({{ page.plexewebsite }}/documentation/#protocols-folder){:target="_blank"} by registering to the reception of `MANEUVER_TYPE`
messages, since the `AbandonMessage` we defined is indeed of type `MANEUVER_TYPE`.

At this stage, the last vehicle of our platoon should be really able to send an Abandon Platoon message
when in the `SimpleScenario` code we do `appl->sendAbandonMessage()`, however, no vehicle is ready to decode/handle
such packets yet.

[`STEP 5, SOLUTION ON GITLAB`](https://tests.ing.unibs.it/ghiro/plexe/-/commit/e90bb0f88133c0f55487f4cb37042e2becd8c57e){:target="_blank"}

## Step 6 - Handling a Message

In this step we implement the handler functions to let the platoon leader parse `AbandonPlatoon` messages
and thus understand that some vehicle is leaving the platoon.
Notice that, however, an application like our `SimplePlatooningApp` may be able to
parse multiple kinds of messages, not just `AbandonPlatoon` messages.
For instance, later we will add the support also to the use of `NewFormation` messages.
Anyhow, all messages arrive to our application from the *MAC/PHY* sublayers.
For this reason, a common practice when implementing the receiver-side of a new application is to:

1. Start from the [`handleLowerMsg()`](https://github.com/michele-segata/plexe/blob/plexe-3.1/src/plexe/apps/GeneralPlatooningApp.cc#L147-L172){:target="_blank"},
 i.e., a function responsible of:
    - Intercepting all kind of packets received by the vehicle radio;
    - Decapsulate packets removing PHY-MAC headers;
    - Identifying the type of packets (e.g., `AbandonPlatoon` or `NewFormation` messages) and activate the dedicated packet handler routine.
2. Implement a specialized handler function for each supported kind of packets. In the present case we will have to implement `handleAbandonPlatoon()`
and `handleNewFormation()`


#### **Step 6 - Hint**
{: .no_toc }

```c++
void SimplePlatooningApp::handleLowerMsg(cMessage* msg) {
    BaseFrame1609_4* frame = check_and_cast<BaseFrame1609_4*>(msg);

    cPacket* enc = frame->getEncapsulatedPacket();
    ASSERT2(enc, "received a BaseFrame1609_4s with nothing inside");

    if (enc->getKind() == MANEUVER_TYPE) {
        ManeuverMessage* mm = check_and_cast<ManeuverMessage*>(
            frame->decapsulate());
        if (AbandonPlatoon* msg = dynamic_cast<AbandonPlatoon*>(mm)) {
            handleAbandonPlatoon(msg);
            delete msg;
        }
        else if (NewFormation* msg = dynamic_cast<NewFormation*>(mm)) {
            handleNewFormation(msg);
            delete msg;
        }
        delete frame;
    }
    else BaseApp::handleLowerMsg(msg);
}
```

- The code shows how cMessages arriving from the lower layer are filtered by Kind `if (enc->getKind() == MANEUVER_TYPE)`.
- Furthermore, packets accepted by the filtering condition are delivered to the proper handler function and then deleted to avoid memory leaks.
- **NB** The BaseApp class makes... a lot of *dirty job under the hood* for us, handling packets that our `SimplePlatooningApp` does
not have to handle directly. This is a why a good `<SOME-APP>::handleLowerMsg()` should dispatch unknown packets to its parent class,
    i.e., to `BaseApp::handleLowerMsg()`.


<!--HANDLING ABANDON MESSAGES-->
```c++
void SimplePlatooningApp::handleAbandonPlatoon(const AbandonPlatoon* msg) {
    if (msg->getPlatoonId() != positionHelper->getPlatoonId())
        return;
    // only leader listens to AbandonMessages
    if (msg->getDestinationId() != positionHelper->getLeaderId())
        return;
    if (msg->getDestinationId() != positionHelper->getId())
        return;

    // Retrieving relevant info from Abandon Message
    int leaderID, leaverID, platoonID;
    leaderID = positionHelper->getId();
    leaverID = msg->getVehicleId();
    platoonID = msg->getPlatoonId();

    // Informing SUMO via Plexe Interface to remove vehicle from platoon
    plexeTraciVehicle->removePlatoonMember(msg->getExternalId());

    // Changing platoon Formation...
    std::vector<int> formation = positionHelper->getPlatoonFormation();

    // Removing the vehicle that abandoned the platoon
    formation.pop_back();
    positionHelper->setPlatoonFormation(formation);

    char text[250];
    sprintf(text, "LEADER[%d]: I'm removing v<%d> from platoon<%d>\n", leaderID, leaverID, platoonID);
    LOG << text << endl;
    getSimulation()->getActiveEnvir()->alert(text);

    sendNewFormationToFollowers(formation);
}
```

- A common practice when coding *handler* functions is to add an initial block of `if` conditions to ensure that the packet
is handled only by the correct, intended recipient of the packet. In the present case, we want `AbandonMessages` to be caught only
by the leader of the platoon we are traveling with, i.e., the message should:
    * report as `PlatoonId` the same `PlatoonId` of the car that is processing the packets and
    * the `ID` of the message recipient should be the `VehicleId` of the vehicle leading the platoon.
- The rest of the code extracts from the message the most relevant information and then uses them to really update the platoon formation.
**NB**: updating the platoon formation is a responsibility of the *positionHelper*, and also SUMO should be informed about the
formation of platoons to correctly implement the dynamics of vehicles under different Cruise Controllers. For this reason, we highlight
in the above snippet of code the use of `plexeTraciVehicle->removePlatoonMember(msg->getExternalId());` and of `positionHelper->setPlatoonFormation(formation);`
- Once the `formation` vector has been updated, with the last line of code the platoon leader informs all other vehicle about the existence of the new formation by calling `sendNewFormationToFollowers(formation);`.

The transmission and reception of NewFormation messages is implemented as shown here below:

<!--SENDING NEW FORMATION MESSAGES-->
```c++
NewFormation* SimplePlatooningApp::createNewFormationMessage(const std::vector<int>& newPlatoonFormation)
{
    NewFormation* nfmsg = new NewFormation();
    nfmsg->setKind(MANEUVER_TYPE);
    nfmsg->setPlatoonFormationArraySize(newPlatoonFormation.size());
    for (unsigned int i = 0; i < newPlatoonFormation.size(); i++) {
        nfmsg->setPlatoonFormation(i, newPlatoonFormation[i]);
    }
    return nfmsg;
}
```

```c++
void SimplePlatooningApp::sendNewFormationToFollowers(const std::vector<int>& newPlatoonFormation)
{
    NewFormation* nfmsg = createNewFormationMessage(newPlatoonFormation);
    int dest;
    // send a copy to each platoon follower
    for (int i = 1; i < newPlatoonFormation.size(); i++) {
        dest = newPlatoonFormation[i];
        NewFormation* dup = nfmsg->dup();
        dup->setDestinationId(dest);
        sendUnicast(dup, dest);
    }
    delete nfmsg;
}
```

<!--HANDLING NEW FORMATION MESSAGES-->

```c++
void SimplePlatooningApp::handleNewFormation(const NewFormation* msg)
{
    std::vector<int> newFormation;
    for (int i = 0; i < msg->getPlatoonFormationArraySize(); i++)
        newFormation.push_back(msg->getPlatoonFormation(i));

    std::string formationString = "[ ";
    for (int i = 0; i < newFormation.size(); i++) {
        formationString += std::to_string(newFormation[i]) + " ";
    }
    formationString += "]";

    char text[250];
    sprintf(text, "v<%d> got newFormation = %s\n", positionHelper->getId(), formationString.c_str());
    getSimulation()->getActiveEnvir()->alert(text);

    positionHelper->setPlatoonFormation(newFormation);
}
```

[`STEP 6, SOLUTION ON GITLAB`](https://tests.ing.unibs.it/ghiro/plexe/-/commit/a6f67d706a88d716851bed5b2d13d59943be9151){:target="_blank"}

## Conclusion
{: .no_toc }

Congratulation, you have completed the *Getting Started with Plexe* tutorial!
Now you should be ready to define new simulation scenarios and write novel cooperative driving
applications based on vehicular communications. Have fun with Plexe!

If you need more help don't hesitate to contact:
- [Michele Segata](mailto:michele.segata@unitn.it) [![]({{ site.url }}{{ site.baseurl }}/assets/res/mailto.png){: width="25" }](mailto:michele.segata@unitn.it)
- [Lorenzo Ghiro](mailto:lorenzo.ghiro@unibs.it) [![]({{ site.url }}{{ site.baseurl }}/assets/res/mailto.png){: width="25" }](mailto:lorenzo.ghiro@unibs.it)


## How to Cite
{: .no_toc }

{% include citePlexe.html %}

<br>
## Footnotes
{: .no_toc }
