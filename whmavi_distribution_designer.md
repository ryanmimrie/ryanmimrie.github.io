---
layout: default
title: WH-MAVI Distribution Designer
permalink: /whmavi/distribution_designer/
---

## Description
The WH-MAVI Distribution Designer allows you to visualize different distributions with custom parameters using interactive controls. Choose a distribution, adjust parameters, and see the results on the graph.

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
  
  <div id="controls">
    <div class="control-group">
      <label for="distribution">Distribution:</label>
      <select id="distribution">
        <option value="johnson-su">Johnson-SU</option>
        <option value="beta">Beta</option>
      </select>
    </div>
    <div class="control-group">
      <label for="xmin">X Min:</label>
      <input type="number" id="xmin" value="0" step="0.1">
    </div>
    <div class="control-group">
      <label for="xmax">X Max:</label>
      <input type="number" id="xmax" value="10" step="0.1">
    </div>
    <div class="control-group">
      <label for="mean">Mean:</label>
      <input type="number" id="mean" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="sd">Standard Deviation:</label>
      <input type="number" id="sd" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="alpha">Alpha:</label>
      <input type="number" id="alpha" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="beta_param">Beta:</label>
      <input type="number" id="beta_param" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="scale">Scale:</label>
      <input type="number" id="scale" value="1" step="0.1">
    </div>
    <div class="control-group">
      <input type="checkbox" id="clamp"> Clamp
    </div>
    <button onclick="plotDistribution()">Generate Plot</button>
  </div>
  <canvas id="distributionChart" width="800" height="400"></canvas>

  <script>
  let chart; // Global chart instance

  // Function to plot or update the distribution chart
  function plotDistribution() {
    const distribution = document.getElementById("distribution").value;
    const xmin = parseFloat(document.getElementById("xmin").value);
    const xmax = parseFloat(document.getElementById("xmax").value);
    const mean = parseFloat(document.getElementById("mean").value);
    const sd = parseFloat(document.getElementById("sd").value);
    const alpha = parseFloat(document.getElementById("alpha").value);
    const beta_param = parseFloat(document.getElementById("beta_param").value);
    const scale = parseFloat(document.getElementById("scale").value);
    const clamp = document.getElementById("clamp").checked;

    const x_values = [];
    const step = (xmax - xmin) / 1000;

    for (let x = xmin; x <= xmax; x += step) {
      x_values.push(x);
    }

    let y_values = [];

    if (distribution === "johnson-su") {
      y_values = x_values.map((x) => {
        if (sd === 0) {
          return mean;
        }
        // Mock implementation of Johnson-SU distribution (simplified)
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
        // Mock implementation of Beta distribution (simplified)
        return Math.pow(x, alpha - 1) * Math.pow(1 - x, beta_param - 1);
      });
    }

    if (!chart) {
      // Initialize the chart the first time
      const ctx = document.getElementById("distributionChart").getContext("2d");
      chart = new Chart(ctx, {
        type: "line",
        data: {
          labels: x_values,
          datasets: [
            {
              label: "Sample Distribution",
              data: y_values,
              borderColor: "blue",
              fill: false,
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
      chart.data.labels = x_values;
      chart.data.datasets[0].data = y_values;
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


