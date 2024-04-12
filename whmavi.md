---
layout: default
title: Within-Host MAVI
permalink: /whmavi/
---

## Quick Summary
Within-Host Model of Acute Virus Infection (WH-MAVI) is an agent-based mathematical model of within-host virus dynamics during acute infection. It has been designed to be highly customisible, user-friendly, and reproducible without requiring a detailed understanding of mathematics or coding to run.

## How to run WH-MAVI
Running WH-MAVI is a two stage process:
<ol>
  <li><b>Designer:</b> In this stage, the user provides information on the behaviours and parameters of the model they would like to run. The user may also provide experimental data for the model to be fit to, and specify which parameters they would like the model to estimate values for during data fitting. All of this information is stored in a .mavi text file, which can be saved for running later, shared with other researchers, or published alongside findings for reproducibility. </li>
  <br>
  <li><b>Solver:</b> In this stage, the model(s) specified in the .mavi text file are solved by the WH-MAVI engine. When there is not any data for fitting, this process is straightforward and the engine will simply run every theoretical scenario specified by the user. If the .mavi file includes data for fitting, the engine will run many iterations of the model with different values for the parameters the user has asked to be estimated.</li>
</ol>

## Designing a WH-MAVI run
.mavi text files can be created using the GUI-based design tool (either in a web browser or locally), or manually. 
### GUI-based Designer
<div>
  <a href="/whmavi/" class="project-button">Web Designer</a>
  <a href="/whmavi/" class="project-button">Local Designer</a>
</div>

### Manually creating .mavi files
The .mavi text file follows a simple structure that can be seen below:

{% highlight yaml %}
# Comment lines begin with a hashtag

# Parameters with fixed values can be specified as:
parameterA: fixed_value

# Parameters with values that change over time can be specified as:
parameterB: [value_1, value_2]

# Parameters with experimental data can be specified as:
parameterC: {data_1, data_2}

# Parameters to be used in different model iterations can be specified as:
parameterD: (value_for_model_1, value_for_model_2)

# Parameters to be estimated by the model can be specified as:
parameterE: {}
{% endhighlight %}


