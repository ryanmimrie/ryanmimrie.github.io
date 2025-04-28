---
layout: default
title: HW-OINK
permalink: /oink/hw/
---

## HW-OINK (v0.1)
<div style="font-size: 0.95em;">This webtool provides model-based estimates of transmission dynamics for respiratory virus outbreaks in hospital wards.<br><br></div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<!-- Virus dropdown -->
<label for="virus">Select virus:</label>
<select id="virus" name="virus" required>
  <option value="">--Choose one--</option>
  <option value="Influenza A virus">Influenza A virus</option>
  <option value="SARS-CoV-2">SARS-CoV-2</option>
</select>
<br><br>

<!-- Number of rooms and beds -->
<label for="rooms">Number of rooms:</label>
<input id="rooms" name="rooms" type="number" min="1" required>
<br>
<label for="beds_per_room">Beds per room:</label>
<input id="beds_per_room" name="beds_per_room" type="number" min="1" required>
<br><br>

<!-- Days since detection and cases per day -->
<label for="days_since_detection">Days since outbreak detected:</label>
<input id="days_since_detection" name="days_since_detection" type="number" min="1" required>
<br>
<label for="cases_per_day">
  Number of cases each day (comma-separated):
</label>
<input id="cases_per_day" name="cases_per_day" type="text" pattern="^(\d+,)*\d+$" placeholder="e.g. 2,3,5,1,0" required>
<br><br>

<!-- Durations of stay -->
<label for="durations_of_stay">
  Previous 30 patient durations of stay (days, comma-separated):
</label>
<input id="durations_of_stay" name="durations_of_stay" type="text" pattern="^(\d+,)*\d+$" placeholder="e.g. 7,5,10,4,..." required>
<br><br>

<button type="submit">Submit</button>
