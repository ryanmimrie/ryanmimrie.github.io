---
layout: default
title: WH-MAVI Distribution Designer
permalink: /whmavi/distribution_designer/
---

## Description
<div style="font-size: 0.95em;">This designer is a visual representation of the distribution function used in WH-MAVI to create phenotypic variation In this example, the output of two instances of the distribution function are shown combined to describe the phenotypic variation across a population that contains two distinct sub-populations.<br><br></div>

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
      </select>
    </div>
    <div class="control-group" id="mean1-group">
      <label for="mean1">Mean:</label>
      <input type="number" id="mean1" value="0.75" step="0.01">
    </div>
    <div class="control-group" id="spread1-group">
      <label for="spread1">Spread:</label>
      <input type="number" id="spread1" value="0.15" step="0.01">
    </div>
    <div class="control-group" id="skew1-group">
      <label for="skew1">Skew:</label>
      <input type="number" id="skew1" value="0" step="0.01">
    </div>
    <div class="control-group" id="rarity1-group">
      <label for="rarity1">Outlier Rarity:</label>
      <input type="number" id="rarity1" value="1" step="0.1">
    </div>
    <div class="control-group" id="clamp1-group">
      <label for="clamp1">Clamp:</label>
      <select id="clamp1">
        <option value="none">None</option>
        <option value="ignore">Ignore</option>
        <option value="squish">Squish</option>
      </select>
    </div>
    <div class="control-group" id="upper1-group">
      <label for="upper1">Upper:</label>
      <input type="number" id="upper1" value="1" step="0.01">
    </div>
    <div class="control-group" id="lower1-group">
      <label for="lower1">Lower:</label>
      <input type="number" id="lower1" value="0" step="0.01">
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
      </select>
    </div>
    <div class="control-group" id="mean2-group">
      <label for="mean2">Mean:</label>
      <input type="number" id="mean2" value="0.25" step="0.01">
    </div>
    <div class="control-group" id="spread2-group">
      <label for="spread2">Spread:</label>
      <input type="number" id="spread2" value="0.15" step="0.01">
    </div>
    <div class="control-group" id="skew2-group">
      <label for="skew2">Skew:</label>
      <input type="number" id="skew2" value="0" step="0.01">
    </div>
    <div class="control-group" id="rarity2-group">
      <label for="rarity2">Outlier Rarity:</label>
      <input type="number" id="rarity2" value="1" step="0.1">
    </div>
    <div class="control-group" id="clamp2-group">
      <label for="clamp2">Clamp:</label>
      <select id="clamp2">
        <option value="none">None</option>
        <option value="ignore">Ignore</option>
        <option value="squish">Squish</option>
      </select>
    </div>
    <div class="control-group" id="upper2-group">
      <label for="upper2">Upper:</label>
      <input type="number" id="upper2" value="1" step="0.01">
    </div>
    <div class="control-group" id="lower2-group">
      <label for="lower2">Lower:</label>
      <input type="number" id="lower2" value="0" step="0.01">
    </div>
  </div>
</div>

<div id="controls" style="display: flex; justify-content: space-between;">
  <div id="population-controls">
    <h3>Population</h3>
    <div class="control-group">
        <label for="phenoratio">Phenotype Ratio:</label>
        <input type="number" id="phenoratio" value="0.75" step="0.01">
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
      <input type="number" id="ymax" value="3" step="0.1">
    </div>
  </div>
</div>

### Details
<div style="font-size: 0.95em;">The WH-MAVI sample distribution function takes in six arguments to produce a distribution:<br><br>
<ul>
  <li><b>Distribution</b>: Either "Normal" or "Johnson-SU". The latter is a four-parameter extension of the normal distribution which allows for increased control of the distribution's shape.</li>
  <li><b>Mean</b>: The centre of the distribution. For the normal distribution, this value represents the mean. For the Johnson-SU distribution, this value represents the Lambda (λ) parameter.</li>
  <li><b>Spread</b>: The extent of variation around the mean. For the normal distribution, this value represents the standard deviation. For the Johnson-SU distribution, this value represents the Xi (ξ) parameter.</li>
  <li><b>Skew</b>: The extent to which the Johnson-SU distribution is positively or negatively skewed. This value represents the Gamma (γ) parameter.</li>
  <li><b>Outlier Rarity</b>: The rarity of outlier values in the Johnson-SU distribution. This value represents the Delta (δ) parameter. </li>
  <li><b>Clamp</b>: This parameter controls the behaviour of values that fall outside the range defined by <b>Upper</b> and <b>Lower</b>.</li>
  <ul>
    <li>"None": The values are unchanged.</li>
    <li>"Ignore": The values are dropped from the distribution and their density re-distributed to maintain the correct phenotype ratios.</li>
    <li>"Squish": The values are added to the boundaries of the distribution.</li>
  </ul>
  <li><b>Upper</b>: The upper X-axis limit of the distribution.</li>
  <li><b>Lower</b>: The lower X-axis limit of the distribution.</li>
</ul>
</div>

<script>
function toggleInputs(groupNumber) {
  
  const distributionType = document.getElementById('distribution' + groupNumber).value;
  const meanGroup = document.getElementById('mean' + groupNumber + '-group');
  const spreadGroup = document.getElementById('spread' + groupNumber + '-group');
  const skewGroup = document.getElementById('skew' + groupNumber + '-group');
  const rarityGroup = document.getElementById('rarity' + groupNumber + '-group');
  const clampGroup = document.getElementById('clamp' + groupNumber + '-group');
  const upperGroup = document.getElementById('upper' + groupNumber + '-group');
  const lowerGroup = document.getElementById('lower' + groupNumber + '-group');

  if (distributionType === 'normal') {
    meanGroup.classList.remove('hidden');
    spreadGroup.classList.remove('hidden');
    skewGroup.classList.add('hidden');
    rarityGroup.classList.add('hidden');
    clampGroup.classList.remove('hidden');
    upperGroup.classList.remove('hidden');
    lowerGroup.classList.remove('hidden');
  } else if (distributionType === 'johnson-su') {
    meanGroup.classList.remove('hidden');
    spreadGroup.classList.remove('hidden');
    skewGroup.classList.remove('hidden');
    rarityGroup.classList.remove('hidden');
    clampGroup.classList.remove('hidden');
    upperGroup.classList.remove('hidden');
    lowerGroup.classList.remove('hidden');
  }
}

document.addEventListener('DOMContentLoaded', function() {
  toggleInputs('1');
  toggleInputs('2');
});
</script>

<script>
let chart;
  
function calculateDistribution(distribution, xmin, xmax, mean, spread, skew, rarity, clamp, upper, lower) {
  
  const x_values = [];
  const step = (xmax - xmin) / 200;

  for (let x = xmin; x <= xmax; x += step) {
    x_values.push(x);
  }

  let y_values = [];

  if (distribution === "normal") {
    y_values = x_values.map((x) => {
      if (spread === 0) {
        const closestX = x_values.reduce((a, b) => Math.abs(b - mean) < Math.abs(a - mean) ? b : a);
        return x === closestX ? 40 : 0;
      }
      const factor = 1 / (spread * Math.sqrt(2 * Math.PI));
      const exponent = -0.5 * Math.pow((x - mean) / spread, 2);
      return factor * Math.exp(exponent);
    });
  } else if (distribution === "johnson-su") {
    y_values = x_values.map((x) => {
      if (spread === 0) {
        const closestX = x_values.reduce((a, b) => Math.abs(b - mean) < Math.abs(a - mean) ? b : a);
        return x === closestX ? 40 : 0;
      }
      const sqrtTwoPi = Math.sqrt(2 * Math.PI);
      const factor = rarity / (spread * sqrtTwoPi);
      const z = Math.asinh((x - mean) / spread);
      const exponent = -0.5 * Math.pow(skew + rarity * z, 2);
      const denominator = Math.sqrt(1 + Math.pow((x - mean) / spread, 2));
      return factor * Math.exp(exponent) / denominator;
    });
  } 

  if (clamp == "ignore") {
    let sumA = y_values.reduce((acc, y, i) => {
      if (x_values[i] < lower || x_values[i] > upper) {
        return acc + y;
      }
      return acc;
    }, 0);
  
    let sumB = y_values.reduce((acc, y, i) => {
      if (x_values[i] > lower && x_values[i] < upper) {
        return acc + y;
      }
      return acc;
    }, 0);
  
    let correctionFactor = (sumA / sumB) + 1;
      
    y_values = y_values.map((y, i) => {
      if (x_values[i] < lower || x_values[i] > upper) {
        return 0;
      }
      return y * correctionFactor;
    });
  } else if (clamp == "squish") {
    const sumYBelowLower = x_values.reduce((acc, x, i) => x < lower ? acc + y_values[i] : acc, 0);
    const sumYAboveUpper = x_values.reduce((acc, x, i) => x > upper ? acc + y_values[i] : acc, 0);

    let indexClosestToLower = 0;
    let indexClosestToUpper = 0;
    let minDistToLower = Infinity;
    let minDistToUpper = Infinity;

    x_values.forEach((x, i) => {
      if (Math.abs(x - lower) < minDistToLower) {
        minDistToLower = Math.abs(x - lower);
        indexClosestToLower = i;
      }
      if (Math.abs(x - upper) < minDistToUpper) {
        minDistToUpper = Math.abs(x - upper);
        indexClosestToUpper = i;
      }
    });

    y_values = y_values.map((y, i) => {
      if (x_values[i] < lower || x_values[i] > upper) {
        return 0;
      } else if (i === indexClosestToLower) {
        return sumYBelowLower;
      } else if (i === indexClosestToUpper) {
        return sumYAboveUpper;
      }
      return y;
    });
  }

  return { x_values, y_values };
}


function plotDistribution() {
  const distribution1 = document.getElementById("distribution1").value;
  const mean1 = parseFloat(document.getElementById("mean1").value);
  const spread1 = parseFloat(document.getElementById("spread1").value);
  const skew1 = parseFloat(document.getElementById("skew1").value);
  const rarity1 = parseFloat(document.getElementById("rarity1").value);
  const clamp1 = document.getElementById("clamp1").value;
  const upper1 = parseFloat(document.getElementById("upper1").value);
  const lower1 = parseFloat(document.getElementById("lower1").value);

  const distribution2 = document.getElementById("distribution2").value;
  const mean2 = parseFloat(document.getElementById("mean2").value);
  const spread2 = parseFloat(document.getElementById("spread2").value);
  const skew2 = parseFloat(document.getElementById("skew2").value);
  const rarity2 = parseFloat(document.getElementById("rarity2").value);
  const clamp2 = document.getElementById("clamp2").value;
  const upper2 = parseFloat(document.getElementById("upper2").value);
  const lower2 = parseFloat(document.getElementById("lower2").value);
  
  const phenoratio = parseFloat(document.getElementById("phenoratio").value);
  
  
  const xmin = parseFloat(document.getElementById("xmin").value);
  const xmax = parseFloat(document.getElementById("xmax").value);
  const ymax = parseFloat(document.getElementById("ymax").value);
  
  const { x_values: x_values1, y_values: y_values1 } = calculateDistribution(distribution1, xmin, xmax, mean1, spread1, skew1, rarity1, clamp1, upper1, lower1);
  const { x_values: x_values2, y_values: y_values2 } = calculateDistribution(distribution2, xmin, xmax, mean2, spread2, skew2, rarity2, clamp2, upper2, lower2);

  const y_values1_mult = y_values1.map(value => value * phenoratio);
  const y_values2_mult = y_values2.map(value => value * (1 - phenoratio));

  const y_values_sum = y_values1_mult.map((y, i) => y + y_values2_mult[i]);

  if (!chart) {
    const ctx = document.getElementById("distributionChart").getContext("2d");
    chart = new Chart(ctx, {
      type: "line",
      data: {
        labels: x_values1,
        datasets: [
          {
            label: "Phenotype 1",
            data: y_values1_mult,
            borderColor: "#3498db",
            fill: false,
            pointRadius: 0,
          },
          {
            label: "Phenotype 2",
            data: y_values2_mult,
            borderColor: "#e74c3c",
            fill: false,
            pointRadius: 0,
          },
          {
            label: "Population Probability Density",
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
            title: {
            display: true,
            text: 'Phenotype Strength'
            }
          },
          y: {
            max: ymax,
            title: {
            display: true,
            text: 'Probability Density'
            }
          },
        },
      }
    });
  } else {
    chart.data.labels = x_values1;
    chart.data.datasets[0].data = y_values1_mult;
    chart.data.datasets[1].data = y_values2_mult;
    chart.data.datasets[2].data = y_values_sum;
    chart.options.scales.y.max = ymax;
    chart.update();
  }
}


  document.querySelectorAll("#controls input, #controls select").forEach((input) => {
    input.addEventListener("input", plotDistribution);
  });

  plotDistribution();
</script>
