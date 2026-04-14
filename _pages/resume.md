---
title: "Resume"
layout: gridlay
sitemap: false
permalink: /resume/
---

## Resume

{% if site.links.cv and site.links.cv != "" %}
<p><a href="{{ site.url }}{{ site.baseurl }}/{{ site.links.cv }}" class="btn-pill btn-website"><i class="ai ai-cv"></i> Download CV (PDF)</a></p>
{% endif %}

<!-- ======================================================
     Research Interests
     ====================================================== -->
<div class="section-card" markdown="0">
<h3>Research Interests</h3>
{% include research-interests.html %}
</div>

<!-- ======================================================
     Education
     ====================================================== -->
<div class="section-card" markdown="0">
<h3>Education</h3>
<div class="cv-entry">
  <div class="cv-period">2017 – 2021</div>
  <div class="cv-body">
    <strong>PhD in Information and Communication Technology</strong><br>
    University of Trento, Department of Information Engineering and Computer Science<br>
    <em>Thesis: Centrality Routing and Blockchain Technologies in Distributed Networks</em><br>
    Advisors: Prof. Renato Lo Cigno, Prof. Leonardo Maccari
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2017</div>
  <div class="cv-body">
    <strong>MSc in Computer Science</strong> (110/110 cum laude)<br>
    University of Trento, Department of Information Engineering and Computer Science
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2014</div>
  <div class="cv-body">
    <strong>BSc in Computer Science</strong> (106/110)<br>
    University of Trento, Department of Information Engineering and Computer Science
  </div>
</div>
</div>

<!-- ======================================================
     Work Experience
     ====================================================== -->
<div class="section-card" markdown="0">
<h3>Work Experience</h3>
<div class="cv-entry">
  <div class="cv-period">Oct 2023 – present</div>
  <div class="cv-body">
    <strong>Assistant Professor (RTD-a)</strong><br>
    University of Brescia, Department of Information Engineering
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">Jul 2021 – Sep 2023</div>
  <div class="cv-body">
    <strong>Postdoctoral Researcher</strong><br>
    University of Trento and University of Brescia
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">Sep 2019 – Mar 2020</div>
  <div class="cv-body">
    <strong>Visiting Scholar</strong><br>
    Northeastern University, Boston (USA)<br>
    Hosted by Prof. Stefano Basagni, Institute for the Wireless Internet of Things (Dir. Prof. Tommaso Melodia). Research on distributed systems and blockchain integration in IoT environments.
  </div>
</div>
<!-- <div class="cv-entry">
  <div class="cv-period">2017</div>
  <div class="cv-body">
    <strong>High School Teacher</strong><br>
    I.T.E. "Cesare Battisti" and I.I.S.S. "Galileo Galilei", Bolzano (Italy)<br>
    Imperative and OOP, Computer Science fundamentals, Computer Networks, databases.
  </div>
</div> -->
</div>

<!-- ======================================================
     Grants
     ====================================================== -->
{% if site.data.grants %}
<div class="section-card" markdown="0">
<h3>Competitive Grant Applications</h3>
{% for grant in site.data.grants %}
<div class="cv-entry">
  <div class="cv-body">
    <strong>{{ grant.name }}</strong>{% if grant.shortname %} <span class="text-muted">({{ grant.shortname }})</span>{% endif %}<br>
    Role: {{ grant.role }} &nbsp;·&nbsp; Budget: {{ grant.budget }}{% if grant.notes %}<br><em>{{ grant.notes }}</em>{% endif %}
  </div>
</div>
{% endfor %}
</div>
{% endif %}

<!-- ======================================================
     Research Projects
     ====================================================== -->
<div class="section-card" markdown="0">
<h3>Research Projects</h3>
<div class="cv-entry">
  <div class="cv-period">2024 – 2025</div>
  <div class="cv-body">
    <strong>SCAR: a Privacy enHAnced SEcurity framework</strong> — <a href="https://ans.unibs.it/projects/scarphase" target="_blank" rel="noopener">ans.unibs.it</a><br>
    Research lead: distributed algorithms and AI-assisted mechanisms for misbehavior detection in vehicular platooning.
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2024 – 2025</div>
  <div class="cv-body">
    <strong>Innovative Security Paradigms for beyond 5G (ISP5G)</strong> — <a href="https://ans.unibs.it/projects/ISP5G" target="_blank" rel="noopener">ans.unibs.it</a><br>
    PHY/MAC-layer security and privacy mechanisms including algorithmic signal obfuscation for beyond-5G systems.
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2024 – 2025</div>
  <div class="cv-body">
    <strong>EMBRACE</strong> — <a href="https://ans.unibs.it/projects/embrace" target="_blank" rel="noopener">ans.unibs.it</a><br>
    Privacy-preserving wireless communication in challenging environments: hardware-aware design and system-level evaluation.
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2022 – 2024</div>
  <div class="cv-body">
    <strong>Sustainable Mobility Center — Spoke 7 (MOST)</strong> — <a href="https://www.centronazionalemost.it/eg/Spoke%207.html" target="_blank" rel="noopener">centronazionalemost.it</a><br>
    Research lead: heterogeneous CACC coordination algorithms, hybrid AI/model-based misbehavior detection, digital twin frameworks for large-scale road networks, wireless power transfer optimization.
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2021 – 2022</div>
  <div class="cv-body">
    <strong>Design and Implementation of an 802.11 Privacy Preserving Sub-Layer (DI-P2SL)</strong> — <a href="https://ans.unibs.it/projects/di-p2sl" target="_blank" rel="noopener">ans.unibs.it</a><br>
    FPGA-based implementation of anti-sensing signal obfuscation; real-time CSI acquisition and adversarial validation.
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2019 – 2020</div>
  <div class="cv-body">
    <strong>Experimental Analysis of CSI Based Anti-Sensing Techniques (CSI-MURDER)</strong> — <a href="https://ans.unibs.it/projects/csi-murder" target="_blank" rel="noopener">ans.unibs.it</a><br>
    CSI-based sensing vulnerabilities and robustness evaluation frameworks.
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2016 – 2018</div>
  <div class="cv-body">
    <strong>Network In-frastructure as Commons — NetCommons (H2020)</strong> — <a href="https://netcommons.eu" target="_blank" rel="noopener">netcommons.eu</a><br>
    Distributed algorithms for network centrality computation; blockchain-based mechanisms for network infrastructure.
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2016 – 2018</div>
  <div class="cv-body">
    <strong>Pop-Routing On WiSHFUL — POPROW (H2020)</strong> — <a href="https://ans.disi.unitn.it/poprow" target="_blank" rel="noopener">ans.disi.unitn.it</a><br>
    Experimental validation of distributed routing and resource allocation over programmable wireless platforms.
  </div>
</div>
</div>

<!-- ======================================================
     Teaching
     ====================================================== -->
<div class="section-card" markdown="0">
<h3>Teaching Experience</h3>
<p><strong>Total: 322 hours of classroom teaching</strong></p>
<div class="cv-entry">
  <div class="cv-period">2021 – 2026</div>
  <div class="cv-body">
    <strong>Vehicular Networks &amp; Cooperative Driving</strong> (MSc) — University of Brescia<br>
    30h/semester · English · Simulation and modeling of cooperative driving (SUMO, OMNeT++, Veins, Plexe); CACC algorithm design and evaluation.
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2021 – 2026</div>
  <div class="cv-body">
    <strong>Elements of Telecommunication Networks</strong> (BSc) — University of Brescia<br>
    20h/semester · Italian · IP subnetting, routing, MAC protocols, TCP congestion control, network emulation.
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2021/22</div>
  <div class="cv-body">
    <strong>Distributed Systems 2</strong> (MSc) — University of Trento<br>
    24h/semester · English · Broadcast protocols, gossip-based failure detection, DHT, blockchain fundamentals.
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2017 – 2019</div>
  <div class="cv-body">
    <strong>Algorithms and Data Structures — Laboratory</strong> (BSc) — University of Trento<br>
    24h/semester · Italian · C++ graph and algorithm exercises, algorithmic problem-solving.
  </div>
</div>
<div class="cv-entry">
  <div class="cv-period">2017</div>
  <div class="cv-body">
    <strong>High School Teacher</strong> — I.T.E. "Cesare Battisti" and I.I.S.S. "Galileo Galilei", Bolzano<br>
    OOP, Computer Science fundamentals, Computer Networks, databases.
  </div>
</div>
</div>

<!-- ======================================================
     Students
     ====================================================== -->
{% if site.data.students %}
<div class="section-card" markdown="0">
<h3>Student Supervision</h3>
<div style="overflow-x: auto; -webkit-overflow-scrolling: touch;">
<table style="width:100%; border-collapse: collapse; min-width: 500px;">
<thead>
<tr style="border-bottom: 2px solid var(--border-color);">
  <th style="text-align:left; padding: var(--space-2) var(--space-3); font-size:0.8125rem; color:var(--text-muted); font-weight:600; text-transform:uppercase; letter-spacing:0.06em;">Student</th>
  <th style="text-align:left; padding: var(--space-2) var(--space-3); font-size:0.8125rem; color:var(--text-muted); font-weight:600; text-transform:uppercase; letter-spacing:0.06em;">Degree</th>
  <th style="text-align:left; padding: var(--space-2) var(--space-3); font-size:0.8125rem; color:var(--text-muted); font-weight:600; text-transform:uppercase; letter-spacing:0.06em;">Year</th>
  <th style="text-align:left; padding: var(--space-2) var(--space-3); font-size:0.8125rem; color:var(--text-muted); font-weight:600; text-transform:uppercase; letter-spacing:0.06em;">Thesis</th>
</tr>
</thead>
<tbody>
{% for student in site.data.students %}
<tr style="border-bottom: 1px solid var(--border-color);">
  <td style="padding: var(--space-3); font-weight:500; white-space:nowrap;">{{ student.name }}</td>
  <td style="padding: var(--space-3); color:var(--text-secondary); white-space:nowrap;">{{ student.degree }}</td>
  <td style="padding: var(--space-3); color:var(--text-muted); white-space:nowrap;">{{ student.year }}</td>
  <td style="padding: var(--space-3); color:var(--text-secondary); font-size:0.875rem;">
    {% if student.file and student.file != "" %}
    <a href="{{ site.url }}{{ site.baseurl }}/thesis/{{ student.file }}" target="_blank" rel="noopener">{{ student.title }}</a>
    {% else %}
    {{ student.title }}
    {% endif %}
  </td>
</tr>
{% endfor %}
</tbody>
</table>
</div>
</div>
{% endif %}

<!-- ======================================================
     Academic Service
     ====================================================== -->
<div class="section-card" markdown="0">
<h3>Academic Service</h3>
<div class="cv-entry">
  <div class="cv-period">2026</div>
  <div class="cv-body"><strong>TPC Member</strong> — IEEE Vehicular Technology Conference (VTC) Fall 2026</div>
</div>
<div class="cv-entry">
  <div class="cv-period">2025, 2026</div>
  <div class="cv-body"><strong>TPC Member</strong> — IEEE Vehicular Technology Conference (VTC) Spring</div>
</div>
<div class="cv-entry">
  <div class="cv-period">2025</div>
  <div class="cv-body"><strong>TPC Member &amp; Web Chair</strong> — ACM Workshop on Wireless Network Testbeds, Experimental evaluation and Characterization (WiNTECH)</div>
</div>
<div class="cv-entry">
  <div class="cv-period">2022</div>
  <div class="cv-body"><strong>Web Chair</strong> — IEEE Wireless On-demand Network Systems and Services (WONS)</div>
</div>
</div>

<!-- ======================================================
     Language Skills
     ====================================================== -->
<div class="section-card" markdown="0">
<h3>Language Skills</h3>
<div style="overflow-x: auto; -webkit-overflow-scrolling: touch;">
<table style="width:60%; border-collapse:collapse; font-size:0.875rem; min-width: 380px;">
<thead>
<tr style="border-bottom: 2px solid var(--border-color);">
  <th style="text-align:left; padding: var(--space-2) var(--space-3); color:var(--text-muted); font-weight:600; text-transform:uppercase; letter-spacing:0.06em;">Language</th>
  <th style="text-align:center; padding: var(--space-2) var(--space-3); color:var(--text-muted); font-weight:600; text-transform:uppercase; letter-spacing:0.06em;">Listening</th>
  <th style="text-align:center; padding: var(--space-2) var(--space-3); color:var(--text-muted); font-weight:600; text-transform:uppercase; letter-spacing:0.06em;">Reading</th>
  <th style="text-align:center; padding: var(--space-2) var(--space-3); color:var(--text-muted); font-weight:600; text-transform:uppercase; letter-spacing:0.06em;">Spoken</th>
  <th style="text-align:center; padding: var(--space-2) var(--space-3); color:var(--text-muted); font-weight:600; text-transform:uppercase; letter-spacing:0.06em;">Writing</th>
</tr>
</thead>
<tbody>
<tr style="border-bottom: 1px solid var(--border-color);">
  <td style="padding: var(--space-3); font-weight:500;">Italian 🇮🇹</td>
  <td style="text-align:center; padding: var(--space-3);" colspan="4">Mother tongue</td>
</tr>
<tr style="border-bottom: 1px solid var(--border-color);">
  <td style="padding: var(--space-3); font-weight:500;">English 🇬🇧 🇺🇲</td>
  <td style="text-align:center; padding: var(--space-3);">C1</td>
  <td style="text-align:center; padding: var(--space-3);">C1</td>
  <td style="text-align:center; padding: var(--space-3);">C1</td>
  <td style="text-align:center; padding: var(--space-3);">C1</td>
</tr>
<tr>
  <td style="padding: var(--space-3); font-weight:500;">German 🇩🇪</td>
  <td style="text-align:center; padding: var(--space-3);">B1</td>
  <td style="text-align:center; padding: var(--space-3);">B2</td>
  <td style="text-align:center; padding: var(--space-3);">B1</td>
  <td style="text-align:center; padding: var(--space-3);">B2</td>
</tr>
</tbody>
</table>
</div>
<p style="font-size:0.8125rem; color:var(--text-muted); margin-top:var(--space-3);">English: Certificate in Advanced English (CAE – Level C1), University of Cambridge ESOL Examinations, 2015.</p>
</div>

