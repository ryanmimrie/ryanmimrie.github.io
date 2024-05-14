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
      <label for="distribution1">Shape:</label>
      <select id="distribution1" onchange="toggleInputs('1')">
        <option value="sigmoid">Sigmoidal</option>
        <option value="tradeoff">Trade-off</option>
        <option value="sudden">Sudden</option>
        <option value="linear (model)">Linear (model)</option>
        <option value="linear (descriptive)">Linear (descriptive)</option>
        <option value="acceleratingup">Accelerating (positive)</option>
        <option value="deceleratingup">Decelerating (positive)</option>
        <option value="acceleratingdown">Accelerating (negative)</option>
        <option value="deceleratingdown">Decelerating (negative)</option>
      </select>
    </div>
    <div class="control-group" id="start">
      <label for="start">Start:</label>
      <input type="number" id="start" value="0" step="0.01">
    </div>
    <div class="control-group" id="end">
      <label for="end">End:</label>
      <input type="number" id="end" value="1" step="0.01">
    </div>
    <div class="control-group" id="inflection">
      <label for="inflection">Inflection:</label>
      <input type="number" id="inflection" value="0.5" step="0.01">
    </div>
    <div class="control-group" id="steepness">
      <label for="steepness">Steepness:</label>
      <input type="number" id="steepness" value="1" step="0.1">
    </div>
  </div>

  <div id="distribution2-controls">
    <h3>Plot Controls</h3>
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

<script>
  let chart;

  function calculateRelationship(x, shape, start, end, inflection, steepness) {
    let yValues = [];

    if (shape === "sigmoidal") {
        x.forEach(xi => {
            let y = start + (end - start) / (1 + Math.exp(-steepness * (xi - inflection)));
            yValues.push(y);
        });
    }

    return yValues;
  }

  function plotRelationship() {
    const shape = document.getElementById("shape").value;
    const start = parseFloat(document.getElementById("mean1").value);
    const end = parseFloat(document.getElementById("spread1").value);
    const inflection = parseFloat(document.getElementById("skew1").value);
    const steepness = parseFloat(document.getElementById("rarity1").value);

    const xmax = parseFloat(document.getElementById("xmax").value);
    const ymax = parseFloat(document.getElementById("ymax").value);

    let x = [];
    for (let i = 0; i <= 100; i += 0.01) {
        x.push(parseFloat(i.toFixed(2)));
    }

    const y_values = calculateRelationship(x, shape, start, end, inflection, steepness);

    if (!chart) {
      const ctx = document.getElementById("relationshipChart").getContext("2d");
      chart = new Chart(ctx, {
        type: "line",
        data: {
          labels: x,
          datasets: [
            {
              data: y_values,
              borderColor: "#3498db",
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
              title: {
                display: true,
                text: 'Variable 1'
              }
            },
            y: {
              max: ymax,
              title: {
                display: true,
                text: 'Variable 2'
              }
            },
          },
        }
      });
    } else {
      chart.data.labels = x;
      chart.data.datasets[0].data = y_values;
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
