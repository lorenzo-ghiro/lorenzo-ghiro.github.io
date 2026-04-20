---
layout: toclay
title: "Exercise with Veins"
description: "Customize the Veins DEMO scenario to implement V2V multihop messaging with TTL-limited propagation, duplicate suppression, and parallel simulation studies."
tags: [Veins, OMNeT++, SUMO, V2V, C++]
permalink: /tutorials/veins-exercise/
---

# {{ page.title }}

The goal of this exercise is to customize the [Veins DEMO Scenario](https://veins.car2x.org/tutorial/#finish){:target="_blank"},
which is well described in the [Veins Tutorial by P. B. Stahorszki]({{ site.url }}{{ site.baseurl }}/assets/res/VEINS_tutorial_PBStahorszki.pdf){:target="_blank"}

Essentially, we just want the cars to communicate to each other, without the need of any Road Side Unit (RSU).
Then, we want to change the *"message propagation strategy"*, i.e., messages should become *multihop*, with the number of hops
defined through a **Time To Live (TTL)** field. Moreover, we want cars to propagate only messages they never propagated before,
which means we want to implement a mechanism to **avoid duplicates**.
Finally, we want to see how much the implemented strategies influence the amount of generated traffic.
Let's start!

## 1. Update SUMO Configuration

As a preliminary step, let's reduce the number of vehicles going around during our simulation, so it will be easier
to track what each car is doing. 

**erlangen.rou.xml**
```xml
<routes>
   ...
   <flow id="flow0" type="vtype0" route="route0" begin="0" period="20" number="5"/>
</routes>
```

Then, similarly to what is done in the [SUMO authobahn tutorial](https://sumo.dlr.de/docs/Tutorials/Autobahn.html#run_the_simulation){:target="_blank"},
define the GUI settings for SUMO, save them in a file called [*guisettings.xml*](https://tests.ing.unibs.it/ghiro/veins/-/blob/demoVNCD/examples/veins/guisettings.xml?ref_type=heads){:target="_blank"} and instruct
[Veins-launchd](https://veins.car2x.org/documentation/sumo-launchd/)
to use them when running the simulation.

**erlangen.sumo.cfg**
```xml
        ...
        <additional-files value="erlangen.poly.xml"/>
        <gui-settings-file value="guisettings.xml"/>
    </input>
```

**erlangen.launchd.xml**
```xml
        ...
    <copy file="erlangen.poly.xml" />
    <copy file="guisettings.xml" />
    <copy file="erlangen.sumo.cfg" type="config" />
</launch>

```

## 2. Simplify Scenario

Let's get rid of the RSU, as we want the communication to happen only among vehicles (V2V), but not towards any infrastructure (V2X).
This is an easy step. 

- Define a new scenario identical to the original [RSUExampleScenario](https://github.com/sommer/veins/blob/veins-5.2/examples/veins/RSUExampleScenario.ned){:target="_blank"}
but remove the RSU declared internally as a submodule. Let's call this file **demoVNCDScenario.ned** and let the network inside be called, again, *DemoVNCDScenario*

**demoVNCDScenario.ned**
```c
network demoVNCDScenario extends Scenario
{
//    submodules:
//        rsu[1]: RSU {
//            @display("p=150,140;i=veins/sign/yellowdiamond;is=vs");
//        }
}

```

- Update the omnetpp.ini. You must change the network name and you can delete all the RSU configuration details

**omnetpp.ini**

~~network = RSUExampleScenario~~
```ini
network = demoVNCDScenario
```

Delete these lines

```ini
##########################################################
#                       RSU SETTINGS                     #
#                                                        #
#                                                        #
##########################################################
*.rsu[0].mobility.x = 2000
*.rsu[0].mobility.y = 2000
*.rsu[0].mobility.z = 3

*.rsu[*].applType = "TraCIDemoRSU11p"
*.rsu[*].appl.headerLength = 80 bit
*.rsu[*].appl.sendBeacons = false
*.rsu[*].appl.dataOnSch = false
*.rsu[*].appl.beaconInterval = 1s
*.rsu[*].appl.beaconUserPriority = 7
*.rsu[*].appl.dataUserPriority = 5
*.rsu[*].nic.phy80211p.antennaOffsetZ = 0 m
```

Delete also these lines as we don't want to handle anymore the channelSwitching and beaconing cases

```ini
[Config WithBeaconing]
*.rsu[*].appl.sendBeacons = true
*.node[*].appl.sendBeacons = true

[Config WithChannelSwitching]
*.**.nic.mac1609_4.useServiceChannel = true
*.node[*].appl.dataOnSch = true
*.rsu[*].appl.dataOnSch = true
``` 

## 3. Meet TraCI in Veins

Now (in theory) our simulation works exactly as in the original [Veins DEMO Scenario](https://veins.car2x.org/tutorial/#finish){:target="_blank"}, just without the assistance of the RSU repeating the received messages with a 2 second delay for those vehicles that approach later the location of the accident.

Unfortunately, in [TraCIDemo11p.cc](https://github.com/sommer/veins/blob/veins-5.2/src/veins/modules/application/traci/TraCIDemo11p.cc){:target="_blank"}
the car color feature  ---cars with accident turning red, while cars that received at least one Wave Short Message turn green---
is implemented only for the Omnet GUI, but not for the SUMO GUI.
Let's start learning how to interact with SUMO via the VEINS traci interface by turning to RED the cars that suffer an accident.

BTW, Did you understand already how accidents are "forced to happen" in the simulation? It's through the use of the *startAccidentMsg*
inside [TraCIMobility.cc](https://github.com/sommer/veins/blob/veins-5.2/src/veins/modules/mobility/traci/TraCIMobility.cc#L135-L150){:target="_blank"}
Let's add there some code to change the vehicle color.

1. We will need first to get the *vehicle command interface*
([there is a standard method to do this](https://github.com/sommer/veins/blob/c96edc2be25266300f73bd701ee33f4acb662631/src/veins/modules/mobility/traci/TraCIMobility.h#L151-L155){:target="_blank"})

2. Then the vehicle command interface supports the method `setColor(TraCIColor(red,green,blue,alpha))`

<!--
<details open="">
    <summary>Solution</summary>
    <p>
            Welcome Cocktail
    </p>
</details>
-->

**SOLUTION**
```c++
void TraCIMobility::handleSelfMsg(cMessage* msg)
{
    if (msg == startAccidentMsg) {
        getVehicleCommandInterface()->setSpeed(0);
        // turn RED the car with accident
        getVehicleCommandInterface()->setColor(TraCIColor(255, 0, 0, 255));
        simtime_t accidentDuration = par("accidentDuration");
        scheduleAt(simTime() + accidentDuration, stopAccidentMsg);
        accidentCount--;
    }
    else if (msg == stopAccidentMsg) {
        getVehicleCommandInterface()->setSpeed(-1);
        // turn back to YELLOW when accident is over
        getVehicleCommandInterface()->setColor(TraCIColor(255, 255, 0, 255));
        if (accidentCount > 0) {
            simtime_t accidentInterval = par("accidentInterval");
            scheduleAt(simTime() + accidentInterval, startAccidentMsg);
...
```

## 4. Customize the Application Logic

If we want to implement a propagation-limit based on TTLs we need to add a TTL field to our messages.
Furthermore, if we want to avoid duplicates we need a mechanism to uniquely identify each "generated packet".
Since every car can generate at least one packet in a given instant of time, the tuple `<senderAddress, timestamp>`
is unique for each packet. Customize the *TraCIDemo11pMessage.msg* so to include the 3 required fields just discussed.

**TraCIDemo11pMessage.msg**
```c++
packet TraCIDemo11pMessage extends BaseFrame1609_4 {
    string demoData;
    LAddress::L2Type senderAddress = -1;
    int TTL = 4;
    omnetpp::simtime_t timestamp;
}
```

Define parameters to control the application logic. Indeed, if later we want to set up a parameter study,
we wish to be able to conveniently define TTL values from the omnetpp.ini file. Similarly, we wish
to toggle on/off the ability of avoiding duplicates. So introduce 2 parameters in the [TraCIDemo11p.ned](https://github.com/sommer/veins/blob/veins-5.2/src/veins/modules/application/traci/TraCIDemo11p.ned){:target="_blank"}
file to be able to customize the Application behavior.

```c++
simple TraCIDemo11p extends DemoBaseApplLayer
{
    parameters:
        @class(veins::TraCIDemo11p);
        @display("i=block/app2");
        bool avoidDuplicates = default(true);
        int TTL = default(2);  
}
```

**Implementation time:**

To implement the *"duplicates avoidance strategy"* we need to keep in memory the
[set](https://en.cppreference.com/w/cpp/container/set){:target="_blank"} of already seen messages.
We also need to know, inside the application code, what is the TTL number configured by the user in the omnetpp.ini.
Introduce the set of received messages, a TTL variable and "avoidDuplicates" flag in [TraCIDemo11p.h](https://github.com/sommer/veins/blob/veins-5.2/src/veins/modules/application/traci/TraCIDemo11p.h){:target="_blank"}

**TraCIDemo11p.h**
```c++
#include <set>
#include <tuple>

namespace veins {
...
protected:
    simtime_t lastDroveAt;
    int currentSubscribedServiceId;
    int TTL;
    bool avoidDuplicates;
    std::set<std::tuple<LAddress::L2Type, omnetpp::simtime_t>> rcvd_messages;
```

and initialize them within the usual `initialize()` function

**TraCIDemo11p.c**
```c++
void TraCIDemo11p::initialize(int stage)
{
    ...
    if (stage == 0) {
        ...
        avoidDuplicates = par("avoidDuplicates").boolValue();
        TTL = par("TTL").intValue();
    }
}

```


Now try to complete the implementation! Essentially you must add some logic to the *onWSM()* function used to
handle the reception of Wave Short Messages and, in particular:

1. if *avoidDuplicates* is `true` then you must check if the received `wsm` is already included in `rcvd_messages`
    * if so, ignore it;
    * otherwise, add it to the set of received messages and then check the TTL.
2. if the *TTL* is not expired, decrease the TTL and propagate the packet.
3. When the car that suffers an accident generates a packet, don't forget to correctly set the `TTL`,
`senderAddress` and the `timestamp` fields.

```c++
void TraCIDemo11p::handlePositionUpdate(cObject* obj)
{
    ...
    wsm->setTimestamp(simTime());
    wsm->setSenderAddress(myId);
    wsm->setTTL(TTL);
    ...
}
```

[SOLUTION](https://tests.ing.unibs.it/ghiro/veins/-/commit/76b2fa165cab2708b5581b73d3afffce304c9278){:target="_blank"}

In the proposed solution the code of `TraCIDemo11p.c` has been polished, removing the code related
to `beacons` and `channelSwitching` modes.

## 5. Set up a Parameter Study

Let's define 2 configurations:

1. One that we call "default" where we avoid duplicates
2. A second where instead duplicates are allowed (called "allowDuplicates")

Beyond this, let's have an iteration variable called "ttl" that loops over the values {1,2,..5}

**omnetpp.ini**
```ini
repeat = 1
[Config Default]
*.node[*].appl.TTL = ${ttl = 1..5 step 1}
*.node[*].appl.avoidDuplicates = true
output-vector-file = ${resultdir}/${configname}_${ttl}_${repetition}.vec
output-scalar-file = ${resultdir}/${configname}_${ttl}_${repetition}.sca

[Config allowDuplicates]
extends = Default
*.node[*].appl.avoidDuplicates = false
output-vector-file = ${resultdir}/${configname}_${ttl}_${repetition}.vec
output-scalar-file = ${resultdir}/${configname}_${ttl}_${repetition}.sca

[Config DefaultNoGui]
extends = Default
*.manager.ignoreGuiCommands = true

[Config allowDuplicatesNoGui]
extends = allowDuplicates
*.manager.ignoreGuiCommands = true

```

- it is a good practice to give a conventional name to `*.vec` and `*.sca` generated by a simulation.
In our example we prefix all filenames with the configuration name and then, separated by `_`, we concatenate
all the iteration variable values we want to track (`${resultdir}/${configname}_${ttl}_${repetition}`).
- we defined also `NoGui` configurations where we disable the support for GUIs, these are configurations suitable
for batch executions in parallel.

Now take a look to the [veins scripts](https://github.com/veins/veins_scripts){:target="_blank"}, which are a collection
of tools that help a researcher in defining simulation batches to be run in parallel thanks to [runmaker](https://github.com/veins/runmaker){:target="_blank"}. We take advantage of these tools to:

1. Define a list of all the simulations we want to run;
2. Run this list of simulations in parallel.

To accomplish our first goal we rely on
[generateRunsFile.pl](https://github.com/veins/veins_scripts#generaterunsfilepl){:target="_blank"}

```console
~/src/veins/examples/veins$  generateRunsFile.pl | grep NoGui
. ./run -u Cmdenv -c DefaultNoGui -r 4
. ./run -u Cmdenv -c allowDuplicatesNoGui -r 3
. ./run -u Cmdenv -c DefaultNoGui -r 1
. ./run -u Cmdenv -c allowDuplicatesNoGui -r 2
. ./run -u Cmdenv -c allowDuplicatesNoGui -r 1
. ./run -u Cmdenv -c DefaultNoGui -r 2
. ./run -u Cmdenv -c allowDuplicatesNoGui -r 4
. ./run -u Cmdenv -c DefaultNoGui -r 3
. ./run -u Cmdenv -c allowDuplicatesNoGui -r 0
. ./run -u Cmdenv -c DefaultNoGui -r 0
```
The script is able to list all possible runs for all configurations listed in the `omnetpp.ini` file, and with `grep` we filter and keep only those that contain the `NoGui` keyword.
The output of `generateRunsFile.pl` is formatted to be compliant with
[runmaker.py](https://github.com/veins/runmaker/blob/master/runmaker4.py){:target="_blank"}, so let's save
the output of `generateRunsFile.pl` in text file that we call `runs.txt`

```console
generateRunsFile.pl | grep NoGui > runs.txt
```

We now just need to let runmaker do the job it is designed for, i.e., spawn in parallel a process for each
simulation listed in some input file (`runs.txt` in the present case).
Before doing so, don't forget to run the [veins_launchd daemon](https://veins.car2x.org/documentation/sumo-launchd){:target="_blank"}.

```console
veins_launchd -vv -c sumo -d
runmaker4.py -j $(nproc) runs.txt
```

Once the simulations will be completed, check if there are new files in your results folder:

```console
~/src/veins/examples/veins$ ls results/
allowDuplicatesNoGui_1_0.sca  allowDuplicatesNoGui_3_0.vec  DefaultNoGui_1_0.vci  DefaultNoGui_4_0.sca
allowDuplicatesNoGui_1_0.vci  allowDuplicatesNoGui_4_0.sca  DefaultNoGui_1_0.vec  DefaultNoGui_4_0.vci
allowDuplicatesNoGui_1_0.vec  allowDuplicatesNoGui_4_0.vci  DefaultNoGui_2_0.sca  DefaultNoGui_4_0.vec
allowDuplicatesNoGui_2_0.sca  allowDuplicatesNoGui_4_0.vec  DefaultNoGui_2_0.vci  DefaultNoGui_5_0.sca
allowDuplicatesNoGui_2_0.vci  allowDuplicatesNoGui_5_0.sca  DefaultNoGui_2_0.vec  DefaultNoGui_5_0.vci
allowDuplicatesNoGui_2_0.vec  allowDuplicatesNoGui_5_0.vci  DefaultNoGui_3_0.sca  DefaultNoGui_5_0.vec
allowDuplicatesNoGui_3_0.sca  allowDuplicatesNoGui_5_0.vec  DefaultNoGui_3_0.vci
allowDuplicatesNoGui_3_0.vci  DefaultNoGui_1_0.sca          DefaultNoGui_3_0.vec
```


## 6. Set up a custom Analysis

Our goal is now to analyse our result files, and draw some plots to show how much traffic has been generated during
the various simulations. We expect this analysis to highlight that, increasing the TTL, more messages
are propagated/sent by cars moreover, avoiding duplicates, we should instead observe less traffic.

The first thing to do is actually to figure out what our `*.sca` and `*.vec` contain. You can explore them
inside the Omnet editor or just give a quick look to some of these files:

```console
cat results/allowDuplicatesNoGui_1_0.sca | grep generated
scalar demoVNCDScenario.node[0].appl generatedWSMs 40
scalar demoVNCDScenario.node[0].appl generatedBSMs 0
scalar demoVNCDScenario.node[0].appl generatedWSAs 0
scalar demoVNCDScenario.node[1].appl generatedWSMs 40
scalar demoVNCDScenario.node[1].appl generatedBSMs 0
scalar demoVNCDScenario.node[1].appl generatedWSAs 0
scalar demoVNCDScenario.node[2].appl generatedWSMs 8
scalar demoVNCDScenario.node[2].appl generatedBSMs 0
scalar demoVNCDScenario.node[2].appl generatedWSAs 0
scalar demoVNCDScenario.node[3].appl generatedWSMs 1
scalar demoVNCDScenario.node[3].appl generatedBSMs 0
scalar demoVNCDScenario.node[3].appl generatedWSAs 0
scalar demoVNCDScenario.node[4].appl generatedWSMs 1
scalar demoVNCDScenario.node[4].appl generatedBSMs 0
scalar demoVNCDScenario.node[4].appl generatedWSAs 0
```

Seems like the `*.sca` files contain the count of how many WSM, BSM and WSA messages have been generated by each car, bingo!
If you are interested in understanding how this stats have been recorded, then check
[DemoBaseApplLayer](https://github.com/sommer/veins/blob/veins-5.2/src/veins/modules/application/ieee80211p/DemoBaseApplLayer.cc#L254),
which is the parent class of the application run by our cars (`TraCIDemo11p`).

Unfortunately, the format of scalar and vector files is a bit cumbersome, and cannot be read directly
by Matlab, Rstudio, Python or whatever tool one may want to use to draw some statistical plot.
Once again, the `veins_scripts` come to our rescue. In particular,
there are some [evaluation scripts](https://github.com/veins/veins_scripts#eval)
that can convert scalar and vector file to csv format, and some others that can
even better extract a subset of stats from the Omnet result files and print a summary of them over further csv files.
Check out all of the [evaluation scripts](https://github.com/veins/veins_scripts#eval)
even if for the present exercise we limit ourselves to just use the
[opp_sca2csv.pl](https://github.com/veins/veins_scripts#opp_sca2csvpl)
script to extract the counter of generatedWSAs from each `*.sca` file we find.

```console
opp_sca2csv.pl -F generatedWSMs results/allowDuplicatesNoGui_1_0.sca 
node    generatedWSMs
demoVNCDScenario.node[0].appl   40
demoVNCDScenario.node[1].appl   40
demoVNCDScenario.node[2].appl   8
demoVNCDScenario.node[3].appl   1
demoVNCDScenario.node[4].appl   1
```

Let's use a bash script to apply the `opp_sca2csv.pl` to each `*.sca` file and save the result in a related `.csv` file.
Create a new directory called `analysis` and, inside it, create a bash script entitled `extract_data.sh` made like this:

**analysis/extract_data.sh**
```bash
#!/bin/bash
RESULT_FOLDER="../results"
for sca in $(ls $RESULT_FOLDER/*sca) 
do
    out=${sca/sca/csv}
    opp_sca2csv.pl -F generatedWSMs $sca > $out
done
```

Run the script and you should see a bunch of new `.csv` files available and ready to be easily processed and plotted with... Python!

```console
~/src/veins/examples/veins$ (cd analysis && bash extract_data.sh)
~/src/veins/examples/veins$ ls results/*.csv
results/allowDuplicatesNoGui_1_0.csv  results/allowDuplicatesNoGui_5_0.csv  results/DefaultNoGui_4_0.csv
results/allowDuplicatesNoGui_2_0.csv  results/DefaultNoGui_1_0.csv          results/DefaultNoGui_5_0.csv
results/allowDuplicatesNoGui_3_0.csv  results/DefaultNoGui_2_0.csv
results/allowDuplicatesNoGui_4_0.csv  results/DefaultNoGui_3_0.csv
```

**Analysis with Python**

Now you are free to process/plot the csv files the way you want.
On the Omnet website you can find an advanced tutorial about [Result Analysis with Python](https://docs.omnetpp.org/tutorials/pandas)

For the present case, just add this python script to your `analysis` folder and run it:

**analysis/compareNumMessages.py**
```python
import re
import pandas as pd
from glob import glob
import matplotlib.pyplot as plt
import seaborn as sns


regex = re.compile("demoVNCDScenario.node\[([\d+])\].appl")
def renameModule(s):
    nc = regex.search(s).groups()[0]
    return int(nc)


csvs = glob("../results/*NoGui_*.csv")

dfs = []
for csv in csvs:
    fname = csv.lstrip("../results/")
    df = pd.read_csv(csv, sep='\t')
    ttl = int(re.search("NoGui_([\d+])", fname).groups()[0])
    avoidDup = True if fname.startswith("Default") else False
    df['node'] = df['node'].apply(lambda x: renameModule(x))
    df['ttl'] = ttl
    df['dup'] = not avoidDup
    dfs.append(df)

df = pd.concat(dfs)

sns.set_style("whitegrid")
sns.catplot(x='node', hue='ttl', y='generatedWSMs',
            data=df, kind='bar', col='dup')

plt.savefig("comparisonNumMSGS.pdf", format='pdf')
```

```console
~/src/veins/examples/veins$ cd analysis
~/src/veins/examples/veins/analysis$ python3 compareNumMessages.py
~/src/veins/examples/veins/analysis$ evince comparisonNumMSGS.pdf &
```

<object data="{{ site.url }}{{ site.baseurl }}/assets/res/comparisonNumMSGS.pdf" width="800" height="450" type="application/pdf"></object>

The plot seems to confirm our expectation: 
- when we suppress duplicates (left plot), in general cars generate less messages;
- when duplicates are not suppressed (right plot), then a larger TTL implies that car keep re-propagating
messages more times, so bars become higher when growing from TTL=1 up to TTL=5

## 7. Conclusion

Congratulations! You have just finished to define and run a VEINS-supported V2V simulation study,
completing also a basic analysis of the Vehicular Communication traffic!

Tutorial written by [*Lorenzo Ghiro*]((mailto:lorenzo.ghiro@unibs.it))