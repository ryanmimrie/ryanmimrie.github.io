---
layout: default
title: WH-MAVI Distribution Designer
permalink: /whmavi/distribution_designer/
---

## Description
This designer is a visual representation of the distribution function used in WH-MAVI to create agent phenotypic heterogeneity.

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>

  h3 {
      text-align: left;
  }
  
  #controls {
    margin-top: 20px;
  }
  
  .control-group {
    margin-bottom: 10px;
    display: flex;
    align-items: center;
  }
  
  .control-group label {
    flex: 1;
    margin-right: 10px;
  }
  
  .control-group select {
    flex: 2;
  }
  
  @media (max-width: 768px) {

    h3 {
        text-align: center;
    }
    
      .control-group {
          flex-direction: column;
      }
  
      .control-group label {
          margin-right: 0;
          margin-bottom: 5px;
      }
  
      .control-group input[type="number"],
      .control-group select {
        flex: none;
        width: 95%;
      }
  }  

  .hidden {
    display: none;
  }
</style>

<canvas id="distributionChart" width="800" height="400"></canvas>

<div id="controls" style="display: flex; justify-content: space-between;">
  <!-- Distribution 1 Inputs -->
  <div id="distribution1-controls">
    <h3>Phenotype 1</h3>
    <div class="control-group">
      <label for="distribution1">Distribution:</label>
      <select id="distribution1" onchange="toggleInputs('1')">
        <option value="normal">Normal</option>
        <option value="johnson-su">Johnson-SU</option>
        <option value="beta">Beta</option>
      </select>
    </div>
    <div class="control-group" id="mean1-group">
      <label for="mean1">Mean:</label>
      <input type="number" id="mean1" value="0.75" step="0.01">
    </div>
    <div class="control-group" id="sd1-group">
      <label for="sd1">SD:</label>
      <input type="number" id="sd1" value="0.1" step="0.01">
    </div>
    <div class="control-group" id="alpha1-group">
      <label for="alpha1">Alpha:</label>
      <input type="number" id="alpha1" value="0" step="0.01">
    </div>
    <div class="control-group" id="beta1-group">
      <label for="beta1">Beta:</label>
      <input type="number" id="beta1" value="1" step="0.1">
    </div>
    <div class="control-group hidden" id="scale1-group">
      <label for="scale1">Scale:</label>
      <input type="number" id="scale1" value="1" step="0.1">
    </div>
    <div class="control-group" id="clamp1-group">
      <label for="clamp1">Clamp:</label>
      <select id="clamp1">
        <option value="none">None</option>
        <option value="ignore">Ignore</option>
        <option value="squish">Squish</option>
      </select>
    </div>
  </div>

  <!-- Distribution 2 Inputs -->
  <div id="distribution2-controls">
    <h3>Phenotype 2</h3>
    <div class="control-group">
      <label for="distribution2">Distribution:</label>
      <select id="distribution2" onchange="toggleInputs('2')">
        <option value="normal">Normal</option>
        <option value="johnson-su">Johnson-SU</option>
        <option value="beta">Beta</option>
      </select>
    </div>
    <div class="control-group" id="mean2-group">
      <label for="mean2">Mean:</label>
      <input type="number" id="mean2" value="0.25" step="0.01">
    </div>
    <div class="control-group" id="sd2-group">
      <label for="sd2">SD:</label>
      <input type="number" id="sd2" value="0.1" step="0.01">
    </div>
    <div class="control-group" id="alpha2-group">
      <label for="alpha2">Alpha:</label>
      <input type="number" id="alpha2" value="0" step="0.01">
    </div>
    <div class="control-group" id="beta2-group">
      <label for="beta2">Beta:</label>
      <input type="number" id="beta2" value="1" step="0.1">
    </div>
    <div class="control-group hidden" id="scale2-group">
      <label for="scale2">Scale:</label>
      <input type="number" id="scale2" value="1" step="0.1">
    </div>
    <div class="control-group" id="clamp2-group">
      <label for="clamp2">Clamp:</label>
      <select id="clamp2">
        <option value="none">None</option>
        <option value="ignore">Ignore</option>
        <option value="squish">Squish</option>
      </select>
    </div>
  </div>
</div>

<div id="controls" style="display: flex; justify-content: space-between;">
  <div id="population-controls">
    <h3>Ratio</h3>
    <div class="control-group">
          <label for="phenoratio">Phenotype Ratio:</label>
          <input type="number" id="phenoratio" value="0.5" step="0.01">
        </div>
  </div>
  <div id="plot-controls">
    <h3>Plot Controls</h3>
    <div class="control-group">
          <label for="xmin">X Min:</label>
          <input type="number" id="xmin" value="-0.2" step="0.1">
        </div>
    <div class="control-group">
      <label for="xmax">X Max:</label>
      <input type="number" id="xmax" value="1.2" step="0.1">
    </div>
    <div class="control-group">
      <label for="ymax">Y Max:</label>
      <input type="number" id="ymax" value="1.2" step="0.1">
    </div>
  </div>
</div>

<script>
function toggleInputs(groupNumber) {
  
  const distributionType = document.getElementById('distribution' + groupNumber).value;
  const meanGroup = document.getElementById('mean' + groupNumber + '-group');
  const sdGroup = document.getElementById('sd' + groupNumber + '-group');
  const clampGroup = document.getElementById('clamp' + groupNumber + '-group');
  const alphaGroup = document.getElementById('alpha' + groupNumber + '-group');
  const betaGroup = document.getElementById('beta' + groupNumber + '-group');
  const scaleGroup = document.getElementById('scale' + groupNumber + '-group');

  if (distributionType === 'normal') {
    meanGroup.classList.remove('hidden');
    sdGroup.classList.remove('hidden');
    clampGroup.classList.remove('hidden');
    alphaGroup.classList.add('hidden');
    betaGroup.classList.add('hidden');
    scaleGroup.classList.add('hidden');
  } else if (distributionType === 'johnson-su') {
    meanGroup.classList.remove('hidden');
    sdGroup.classList.remove('hidden');
    clampGroup.classList.remove('hidden');
    alphaGroup.classList.remove('hidden');
    betaGroup.classList.remove('hidden');
    scaleGroup.classList.add('hidden');
  } else if (distributionType === 'beta') {
    meanGroup.classList.add('hidden');
    sdGroup.classList.add('hidden');
    clampGroup.classList.add('hidden');
    alphaGroup.classList.remove('hidden');
    betaGroup.classList.remove('hidden');
    scaleGroup.classList.remove('hidden');
  }
}

document.addEventListener('DOMContentLoaded', function() {
  toggleInputs('1');
  toggleInputs('2');
});
</script>

<script>
let chart;
  
function calculateDistribution(distribution, xmin, xmax, mean, sd, alpha, beta, scale, clamp) {
  const x_values = [];
  const step = (xmax - xmin) / 200;

  for (let x = xmin; x <= xmax; x += step) {
    x_values.push(x);
  }

  let y_values = [];

  if (distribution === "normal") {
    y_values = x_values.map((x) => {
        if (sd === 0) {
            const closestX = x_values.reduce((a, b) => Math.abs(b - mean) < Math.abs(a - mean) ? b : a);
            return x === closestX ? 1 : 0;
        }
      const factor = 1 / (sd * Math.sqrt(2 * Math.PI));
      const exponent = -0.5 * Math.pow((x - mean) / sd, 2);
      return factor * Math.exp(exponent);
    });
  } else if (distribution === "johnson-su") {
    y_values = x_values.map((x) => {
        if (sd === 0) {
            const closestX = x_values.reduce((a, b) => Math.abs(b - mean) < Math.abs(a - mean) ? b : a);
            return x === closestX ? 1 : 0;
        }
      const sqrtTwoPi = Math.sqrt(2 * Math.PI);
      const factor = beta / (sd * sqrtTwoPi);
      const z = Math.asinh((x - mean) / sd);
      const exponent = -0.5 * Math.pow(alpha + beta * z, 2);
      const denominator = Math.sqrt(1 + Math.pow((x - mean) / sd, 2));
      return factor * Math.exp(exponent) / denominator;
    });
  } else if (distribution === "beta") {
      y_values = x_values.map((x) => {
      return Math.pow(x, alpha - 1) * Math.pow(1 - x, beta - 1);
    });
  }
  if (clamp == "ignore") {
      y_values = y_values.map((y, i) => {
        if (x_values[i] < 0 || x_values[i] > 1) {
          return 0;
        }
        return y;
      });
    } else if (clamp == "squish") {
      const sumYBelowZero = x_values.reduce((acc, x, i) => x < 0 ? acc + y_values[i] : acc, 0);
      const sumYAboveOne = x_values.reduce((acc, x, i) => x > 1 ? acc + y_values[i] : acc, 0);
    
      let indexClosestToZero = 0;
      let indexClosestToOne = 0;
      let minDistToZero = Infinity;
      let minDistToOne = Infinity;
    
      x_values.forEach((x, i) => {
        if (Math.abs(x) < minDistToZero) {
          minDistToZero = Math.abs(x);
          indexClosestToZero = i;
        }
        if (Math.abs(x - 1) < minDistToOne) {
          minDistToOne = Math.abs(x - 1);
          indexClosestToOne = i;
        }
      });
    
      y_values = y_values.map((y, i) => {
        if (x_values[i] < 0 || x_values[i] > 1) {
          return 0;
        } else if (i === indexClosestToZero) {
          return sumYBelowZero;
        } else if (i === indexClosestToOne) {
          return sumYAboveOne;
        }
        return y;
      });
    }

  return { x_values, y_values };
}

function plotDistribution() {
  const distribution1 = document.getElementById("distribution1").value;
  const mean1 = parseFloat(document.getElementById("mean1").value);
  const sd1 = parseFloat(document.getElementById("sd1").value);
  const alpha1 = parseFloat(document.getElementById("alpha1").value);
  const beta1 = parseFloat(document.getElementById("beta1").value);
  const scale1 = parseFloat(document.getElementById("scale1").value);
  const clamp1 = document.getElementById("clamp1").value;

  const distribution2 = document.getElementById("distribution2").value;
  const mean2 = parseFloat(document.getElementById("mean2").value);
  const sd2 = parseFloat(document.getElementById("sd2").value);
  const alpha2 = parseFloat(document.getElementById("alpha2").value);
  const beta2 = parseFloat(document.getElementById("beta2").value);
  const scale2 = parseFloat(document.getElementById("scale2").value);
  const clamp2 = document.getElementById("clamp2").value;

  const xmin = parseFloat(document.getElementById("xmin").value);
  const xmax = parseFloat(document.getElementById("xmax").value);
  const ymax = parseFloat(document.getElementById("ymax").value);

  const { x_values: x_values1, y_values: y_values1 } = calculateDistribution(distribution1, xmin, xmax, mean1, sd1, alpha1, beta1, scale1, clamp1);
  const { x_values: x_values2, y_values: y_values2 } = calculateDistribution(distribution2, xmin, xmax, mean2, sd2, alpha2, beta2, scale2, clamp2);

  const y_values_sum = y_values1.map((y, i) => y + y_values2[i]);

  if (!chart) {
    const ctx = document.getElementById("distributionChart").getContext("2d");
    chart = new Chart(ctx, {
      type: "line",
      data: {
        labels: x_values1,
        datasets: [
          {
            label: "Phenotypic Group 1",
            data: y_values1,
            borderColor: "#3498db",
            fill: false,
            pointRadius: 0,
          },
          {
            label: "Phenotypic Group 2",
            data: y_values2,
            borderColor: "#e74c3c",
            fill: false,
            pointRadius: 0,
          },
          {
            label: "Agent Population Distribution",
            data: y_values_sum,
            backgroundColor: "#34495e",
            fill: true,
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
          y: {
            max: ymax,
          },
        },
      }
    });
  } else {
    // Update the existing chart's data and refresh it
    chart.data.labels = x_values1;
    chart.data.datasets[0].data = y_values1;
    chart.data.datasets[1].data = y_values2;
    chart.data.datasets[2].data = y_values_sum;
    chart.options.scales.y.max = ymax;
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
