---
layout: default
title: Within-Host MAVI
permalink: /whmavi/
---

# Within-Host MAVI
## Quick Summary
Within-Host Model of Acute Virus Infection (WH-MAVI) is an agent-based mathematical model of within-host virus dynamics during acute infection.
It has been designed to be highly customisible, user-friendly, and reproducible without requiring any detailed understanding of mathematics or coding.
## How to run WH-MAVI
Running WH-MAVI is a two stage process:
<ol>
  <li>Designer: In this stage, the user provides information on the behaviours and parameters of the model they would like to run. These are stored in an output text file, which can be saved for running later, shared with other researchers, or published alongside findings for reproducibility. </li>
  <li>Solver: In this stage, the model(s) specified in the output text file are solved by the WH-MAVI engine. If the specified models contain data, the solver will run many iterations of the model to find optimal parameter values to describe the data.</li>
</ol>
