---
layout: default
title: WH-MAVI Relationship Designer
permalink: /whmavi/relationship_designer/
---

## Description
This designer is a visual representation of the relationship function used in WH-MAVI to set the relationships between correlated phenotypes, and effects that vary over time.

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
  <div id="distribution1-controls">
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
    <div>
      <label for="fix1">Fix Ignored Density:</label>
      <input type="checkbox" id="fix1" checked style="transform: scale(1.35); margin-left: 5px;">
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
