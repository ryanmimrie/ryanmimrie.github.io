---
layout: default
title: WH-MAVI Relationship Designer
permalink: /whmavi/relationship_designer/
---

## Description
<div style="font-size: 0.95em;">This designer is a visual representation of the relationship function used in WH-MAVI to describe correlated phenotypes and effects that vary over time.<br><br></div>

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

<canvas id="relationshipChart" width="800" height="400"></canvas>

<div id="controls" style="display: flex; justify-content: space-between;">
  <div id="relationship-controls">
    <h3>Relationship</h3>
    <div class="control-group">
      <label for="shape">Shape:</label>
      <select id="shape" onchange="toggleInputs('1')">
        <option value="sigmoid">Sigmoid</option>
        <option value="tradeoff">Trade-off</option>
        <option value="linear">Linear</option>
        <option value="exponential">Exponential</option>
      </select>
    </div>
    <div class="control-group" id="start-x">
      <label for="start-x">Start X:</label>
      <input type="number" id="start-x-value" value="0" step="0.01">
    </div>
    <div class="control-group" id="start-y">
      <label for="start-y">Start Y:</label>
      <input type="number" id="start-y-value" value="0" step="0.01">
    </div>
    <div class="control-group"  id="end-x">
      <label for="end-x">End X:</label>
      <input type="number" id="end-x-value" value="1" step="0.01">
    </div>
    <div class="control-group" id="end-y">
      <label for="end-y">End Y:</label>
      <input type="number" id="end-y-value" value="1" step="0.01">
    </div>
    <div class="control-group"  id="base-y">
      <label for="base-y">Base Y:</label>
      <input type="number" id="base-y-value" value="0" step="0.01">
    </div>
    <div class="control-group" id="mid-y">
      <label for="mid-y">Midpoint Y:</label>
      <input type="number" id="mid-y-value" value="1" step="0.01">
    </div>
    <div class="control-group" id="curve">
      <label for="curve">Curve:</label>
      <input type="number" id="curve-value" value="0" step="0.01">
    </div>
    <div class="control-group" id="plateau-upper">
      <label for="plateau-upper-value">Upper:</label>
      <input type="number" id="plateau-upper-value" value="1" step="0.01">
    </div>
    <div class="control-group" id="plateau-lower">
      <label for="plateau-lower-value">Lower:</label>
      <input type="number" id="plateau-lower-value" value="0" step="0.01">
    </div>
</div>

  <div id="plot-controls">
    <h3>Plot Controls</h3>
    <div class="control-group">
      <label for="xmax">X Max:</label>
      <input type="number" id="xmax" value="1" step="0.1">
    </div>
    <div class="control-group">
      <label for="ymax">Y Max:</label>
      <input type="number" id="ymax" value="1" step="0.1">
    </div>
  </div>
</div>

<script>
  
function toggleInputs() {
  const shape = document.getElementById('shape').value;
  const start_y = document.getElementById('start-y');
  const start_x = document.getElementById('start-x');
  const end_y = document.getElementById('end-y');
  const end_x = document.getElementById('end-x');
  const base_y = document.getElementById('base-y');
  const midpoint_y = document.getElementById('mid-y');
  const curve = document.getElementById('curve');
  const upper_plateau = document.getElementById('plateau-upper');
  const lower_plateau = document.getElementById('plateau-lower');

  if (shape === 'sigmoid') {
    start_y.classList.remove('hidden');
    start_x.classList.remove('hidden');
    end_y.classList.remove('hidden');
    end_x.classList.remove('hidden');
    base_y.classList.add('hidden');
    midpoint_y.classList.add('hidden');
    curve.classList.add('hidden');
    upper_plateau.classList.add('hidden');
    lower_plateau.classList.add('hidden');
  } else if (shape === 'tradeoff') {
    start_y.classList.add('hidden');
    start_x.classList.remove('hidden');
    end_y.classList.add('hidden');
    end_x.classList.remove('hidden');
    base_y.classList.remove('hidden');
    midpoint_y.classList.remove('hidden');
    curve.classList.add('hidden');
    upper_plateau.classList.add('hidden');
    lower_plateau.classList.add('hidden');
  } else if (shape === 'linear') {
    start_y.classList.remove('hidden');
    start_x.classList.remove('hidden');
    end_y.classList.remove('hidden');
    end_x.classList.remove('hidden');
    base_y.classList.add('hidden');
    midpoint_y.classList.add('hidden');
    curve.classList.add('hidden');
    upper_plateau.classList.remove('hidden');
    lower_plateau.classList.remove('hidden');
  } else if (shape === 'exponential') {
    start_y.classList.remove('hidden');
    start_x.classList.remove('hidden');
    end_y.classList.remove('hidden');
    end_x.classList.remove('hidden');
    base_y.classList.add('hidden');
    midpoint_y.classList.add('hidden');
    curve.classList.remove('hidden');
    upper_plateau.classList.add('hidden');
    lower_plateau.classList.add('hidden');
  }
}

document.addEventListener('DOMContentLoaded', function() {
  toggleInputs();
  document.getElementById('shape').addEventListener('change', toggleInputs);
});

  
</script>

<script>
let chart;

function calculateRelationship(x, shape, start_y, end_y, start_x, end_x, base_y, mid_y, curve, plateau_upper, plateau_lower) {
  let yValues = [];

  if (shape === "sigmoid") {
    let steepness = 10 / (end_x - start_x);
    let inflection = (start_x + end_x) / 2;

    x.forEach(xi => {
      let y = start_y + (end_y - start_y) / (1 + Math.exp(-steepness * (xi - inflection)));
      yValues.push(y);
    });
  }

  if (shape === "tradeoff") {
    let mu = (start_x + end_x) / 2;
    let sigma = (end_x - start_x) / 8;

    x.forEach(xi => {
      let exponent = -Math.pow((xi - mu), 2) / (2 * Math.pow(sigma, 2));
      let y = base_y + (mid_y - base_y) * Math.exp(exponent);
      yValues.push(y);
    });
  }

  if (shape === "linear") {
    let slope = (end_y - start_y) / (end_x - start_x);
    let intercept = start_y - slope * start_x;

    x.forEach(xi => {
      let y = slope * xi + intercept;
      yValues.push(y);
    });
  }

  if (shape === "exponential") {
    let A = (end_y - start_y) / (Math.exp(curve * end_x) - Math.exp(curve * start_x));
    let B = start_y - A * Math.exp(curve * start_x);

    x.forEach(xi => {
      let y = A * Math.exp(curve * xi) + B;
      yValues.push(y);
    });
  }

  yValues = yValues.map(y => {
    if (y < plateau_lower) {
      return plateau_lower;
    } else if (y > plateau_upper) {
      return plateau_upper;
    } else {
      return y;
    }
  });

  return yValues;
}

function plotRelationship() {
  const shape = document.getElementById("shape").value;
  const start_y = parseFloat(document.getElementById("start-y-value").value);
  const end_y = parseFloat(document.getElementById("end-y-value").value);
  const start_x = parseFloat(document.getElementById("start-x-value").value);
  const end_x = parseFloat(document.getElementById("end-x-value").value);
  const base_y = parseFloat(document.getElementById("base-y-value").value);
  const mid_y = parseFloat(document.getElementById("mid-y-value").value);
  const curve = parseFloat(document.getElementById("curve-value").value);
  const plateau_upper = parseFloat(document.getElementById("plateau-upper-value").value);
  const plateau_lower = parseFloat(document.getElementById("plateau-lower-value").value);

  const xmax = parseFloat(document.getElementById("xmax").value);
  const ymax = parseFloat(document.getElementById("ymax").value);

  let x = [];
  for (let i = 0; i <= xmax; i += xmax / 1000) {
    x.push(parseFloat(i.toFixed(2)));
  }

  const y = calculateRelationship(x, shape, start_y, end_y, start_x, end_x, base_y, mid_y, curve, plateau_upper, plateau_lower);

  console.log("Curve:", curve);
  console.log("First 10 values of x:", x.slice(0, 10));
  console.log("First 10 values of y_values:", y.slice(0, 10));

  if (!chart) {
    const ctx = document.getElementById("relationshipChart").getContext("2d");
    chart = new Chart(ctx, {
      type: "line",
      data: {
        labels: x,
        datasets: [
          {
            data: y,
            borderColor: "#3498db",
            fill: false,
            pointRadius: 0,
          },
          {
            data: [], // New dataset for additional points
            borderColor: "red", // Different color for the points
            fill: false,
            pointRadius: 5, // Visible points
            showLine: false // Only show points, not lines
          }
        ],
      },
      options: {
        scales: {
          x: {
            min: 0,
            max: xmax,
            type: "linear",
            position: "bottom",
            title: {
              display: true,
              text: 'Variable 1'
            }
          },
          y: {
            min: 0,
            max: ymax,
            title: {
              display: true,
              text: 'Variable 2'
            }
          },
        },
        plugins: {
          legend: {
            display: false
          }
        }
      }
    });
  } else {
    chart.data.labels = x;
    chart.data.datasets[0].data = y;
    chart.options.scales.x.max = xmax;
    chart.options.scales.y.max = ymax;
  }

  let additionalPoints = [];
  if (shape === "tradeoff") {
    additionalPoints.push({ x: start_x, y: base_y });
    additionalPoints.push({ x: end_x, y: base_y });
    additionalPoints.push({ x: start_x + (end_x - start_x) / 2, y: mid_y });
  } else {
    additionalPoints.push({ x: start_x, y: start_y });
    additionalPoints.push({ x: end_x, y: end_y });
  }

  chart.data.datasets[1].data = additionalPoints;
  chart.update();
}

document.querySelectorAll("#controls input, #controls select").forEach((input) => {
  input.addEventListener("input", plotRelationship);
});

// Initial plot
plotRelationship();
</script>
