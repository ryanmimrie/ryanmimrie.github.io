---
layout: default
title: WH-MAVI Relationship Designer
permalink: /whmavi/relationship_designer/
---

## Description
<div style="font-size: 0.95em;">This designer is a visual representation of the relationship function used in WH-MAVI to describe correlated phenotypes and effects that vary over time.</div>

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
        <option value="linear (model)">Linear (model)</option>
        <option value="linear (descriptive)">Linear (descriptive)</option>
        <option value="acceleratingup">Accelerating (positive)</option>
        <option value="deceleratingup">Decelerating (positive)</option>
        <option value="acceleratingdown">Accelerating (negative)</option>
        <option value="deceleratingdown">Decelerating (negative)</option>
      </select>
    </div>
    <div class="control-group">
      <label for="start-y">Start Y:</label>
      <input type="number" id="start-y-value" value="0" step="0.01">
    </div>
    <div class="control-group">
      <label for="start-x">Start X:</label>
      <input type="start-x" id="start-x-value" value="0.5" step="0.01">
    </div>
    <div class="control-group">
      <label for="end-y">End Y:</label>
      <input type="number" id="end-y-value" value="1" step="0.01">
    </div>
    <div class="control-group">
      <label for="end-x">End X:</label>
      <input type="number" id="end-x-value" value="1" step="0.1">
    </div>
    <div>
      <label for="clamp-negative">Clamp Negative Values:</label>
      <input type="checkbox" id="clamp-negative-value" checked style="transform: scale(1.35); margin-left: 5px;">
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
  let chart;

  function calculateRelationship(x, shape, start_y, end_y, start_x, end_x) {
    let yValues = [];

    if (shape === "sigmoid") {
      let steepness = 10 / (end_x - start_x);
      let inflection = (start_x + end_x) / 2;
  
      x.forEach(xi => {
          let y = start_y + (end_y - start_y) / (1 + Math.exp(-steepness * (xi - inflection)));
          yValues.push(y);
      });
  }

    return yValues;
  }

  function plotRelationship() {
    const shape = document.getElementById("shape").value;
    const start_y = parseFloat(document.getElementById("start-y-value").value);
    const end_y = parseFloat(document.getElementById("end-y-value").value);
    const start_x = parseFloat(document.getElementById("start-x-value").value);
    const end_x = parseFloat(document.getElementById("end-x-value").value);

    const xmax = parseFloat(document.getElementById("xmax").value);
    const ymax = parseFloat(document.getElementById("ymax").value);

    let x = [];
    for (let i = 0; i <= 100; i += 0.01) {
        x.push(parseFloat(i.toFixed(2)));
    }

    console.log(`shape: ${shape}, start y: ${start_y}, end y: ${end_y}, start x: ${start_x}, end x: ${end_x}`);
    console.log(`xmax: ${xmax}, ymax: ${ymax}`);
    
    const y = calculateRelationship(x, shape, start_y, end_y, start_x, end_x);

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
      chart.update();
    }
  }

  document.querySelectorAll("#controls input, #controls select").forEach((input) => {
    input.addEventListener("input", plotRelationship);
  });

  // Initial plot
  plotRelationship();
</script>
