---
layout: default
title: WH-MAVI Distribution Designer
permalink: /whmavi/distribution_designer/
---

## Description
This designer is a visual representation of the distribution function used in WH-MAVI to create phenotypic heterogeneity across agents in the model.

## Distribution Plot Tool

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
  #controls {
    margin-bottom: 20px;
  }
  .control-group {
    margin-bottom: 10px;
  }
</style>

<div id="controls" style="display: flex; justify-content: space-between;">
  <!-- Distribution 1 Inputs -->
  <div id="distribution1-controls">
    <h3>Distribution 1</h3>
    <div class="control-group">
      <label for="distribution1">Distribution:</label>
      <select id="distribution1">
        <option value="johnson-su">Johnson-SU</option>
        <option value="beta">Beta</option>
      </select>
    </div>
    <div class="control-group">
      <label for="xmin1">X Min:</label>
      <input type="number" id="xmin1" value="0" step="0.1">
    </div>
    <div class="control-group">
      <label for="xmax1">X Max:</label>
      <input type="number" id="xmax1" value="10" step="0.1">
    </div>
    <div class="control-group">
      <label for="mean1">Mean:</label>
      <input type="number" id="mean1" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="sd1">Standard Deviation:</label>
      <input type="number" id="sd1" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="alpha1">Alpha:</label>
      <input type="number" id="alpha1" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="beta_param1">Beta:</label>
      <input type="number" id="beta_param1" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="scale1">Scale:</label>
      <input type="number" id="scale1" value="1" step="0.1">
    </div>
    <div class="control-group">
      <input type="checkbox" id="clamp1"> Clamp
    </div>
  </div>

  <!-- Distribution 2 Inputs -->
  <div id="distribution2-controls">
    <h3>Distribution 2</h3>
    <div class="control-group">
      <label for="distribution2">Distribution:</label>
      <select id="distribution2">
        <option value="johnson-su">Johnson-SU</option>
        <option value="beta">Beta</option>
      </select>
    </div>
    <div class="control-group">
      <label for="xmin2">X Min:</label>
      <input type="number" id="xmin2" value="0" step="0.1">
    </div>
    <div class="control-group">
      <label for="xmax2">X Max:</label>
      <input type="number" id="xmax2" value="10" step="0.1">
    </div>
    <div class="control-group">
      <label for="mean2">Mean:</label>
      <input type="number" id="mean2" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="sd2">Standard Deviation:</label>
      <input type="number" id="sd2" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="alpha2">Alpha:</label>
      <input type="number" id="alpha2" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="beta_param2">Beta:</label>
      <input type="number" id="beta_param2" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="scale2">Scale:</label>
      <input type="number" id="scale2" value="1" step="0.1">
    </div>
    <div class="control-group">
      <input type="checkbox" id="clamp2"> Clamp
    </div>
  </div>
</div>

<canvas id="distributionChart" width="800" height="400"></canvas>

<script>
  let chart; // Global chart instance

  // Function to calculate distribution values
  function calculateDistribution(distribution, xmin, xmax, mean, sd, alpha, beta_param, clamp) {
    const x_values = [];
    const step = (xmax - xmin) / 100;

    for (let x = xmin; x <= xmax; x += step) {
      x_values.push(x);
    }

    let y_values = [];

    if (distribution === "johnson-su") {
      y_values = x_values.map((x) => {
        if (sd === 0) {
          return mean;
        }
        return Math.exp(-0.5 * Math.pow((x - mean) / sd, 2));
      });

      if (clamp) {
        y_values = y_values.map((y, i) => {
          if (x_values[i] < 0 || x_values[i] > 1) {
            return 0;
          }
          return y;
        });
      }
    } else if (distribution === "beta") {
      y_values = x_values.map((x) => {
        return Math.pow(x, alpha - 1) * Math.pow(1 - x, beta_param - 1);
      });
    }

    return { x_values, y_values };
  }

  // Function to plot or update the distribution chart
  function plotDistribution() {
    // Fetch inputs for Distribution 1
    const distribution1 = document.getElementById("distribution1").value;
    const xmin1 = parseFloat(document.getElementById("xmin1").value);
    const xmax1 = parseFloat(document.getElementById("xmax1").value);
    const mean1 = parseFloat(document.getElementById("mean1").value);
    const sd1 = parseFloat(document.getElementById("sd1").value);
    const alpha1 = parseFloat(document.getElementById("alpha1").value);
    const beta_param1 = parseFloat(document.getElementById("beta_param1").value);
    const clamp1 = document.getElementById("clamp1").checked;

    // Fetch inputs for Distribution 2
    const distribution2 = document.getElementById("distribution2").value;
    const xmin2 = parseFloat(document.getElementById("xmin2").value);
    const xmax2 = parseFloat(document.getElementById("xmax2").value);
    const mean2 = parseFloat(document.getElementById("mean2").value);
    const sd2 = parseFloat(document.getElementById("sd2").value);
    const alpha2 = parseFloat(document.getElementById("alpha2").value);
    const beta_param2 = parseFloat(document.getElementById("beta_param2").value);
    const clamp2 = document.getElementById("clamp2").checked;

    // Calculate values for both distributions
    const { x_values: x_values1, y_values: y_values1 } = calculateDistribution(distribution1, xmin1, xmax1, mean1, sd1, alpha1, beta_param1, clamp1);
    const { x_values: x_values2, y_values: y_values2 } = calculateDistribution(distribution2, xmin2, xmax2, mean2, sd2, alpha2, beta_param2, clamp2);

    if (!chart) {
      // Initialize the chart the first time
      const ctx = document.getElementById("distributionChart").getContext("2d");
      chart = new Chart(ctx, {
        type: "line",
        data: {
          labels: x_values1, // First distribution is used for labels
          datasets: [
            {
              label: "Distribution 1",
              data: y_values1,
              borderColor: "blue",
              fill: false,
              pointRadius: 0,
            },
            {
              label: "Distribution 2",
              data: y_values2,
              borderColor: "red",
              fill: false,
              pointRadius: 0,
            },
          ],
        },
        options: {
          scales: {
            x: {
              type: "linear",
              position: "bottom",
            },
          },
        },
      });
    } else {
      // Update the existing chart's data and refresh it
      chart.data.labels = x_values1;
      chart.data.datasets[0].data = y_values1;
      chart.data.datasets[1].data = y_values2;
      chart.update();
    }
  }

  // Attach change listeners to all controls to update the chart on input change
  document.querySelectorAll("#controls input, #controls select").forEach((input) => {
    input.addEventListener("input", plotDistribution);
  });

  // Initial plot
  plotDistribution();
</script>
