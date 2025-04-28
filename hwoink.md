---
layout: default
title: HW-OINK
permalink: /oink/hw/
---

# Hospital Ward (HW)-OINK (v0.1)
<div style="font-size: 0.95em;">
  This webtool provides model-based estimates of transmission dynamics for respiratory virus outbreaks in hospital wards. The method has been adapted from the original OINK (Outbreak Inference with Negligible Knowledge) model described in <a href="https://doi.org/10.1098/rsif.2024.0168" target="_blank" rel="noopener noreferrer">Fozard et al., (2024)</a>.
  <br><br>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

## Data Entry
### Upload Existing Data
<div style="display: flex; gap: 16px; margin-bottom: 16px;">
  <button style="padding: 12px 24px; font-size: 16px; border: none; border-radius: 4px; cursor: pointer; display: flex; align-items: center; gap: 8px;">
    <svg width="18" height="18" viewBox="0 0 20 20" style="vertical-align: middle;"><path fill="currentColor" d="M10 14l4-4h-3V2h-2v8H6l4 4zm-8 4v-2h16v2H2z"/></svg>
    Download Template
  </button>
  <button style="padding: 12px 24px; font-size: 16px; border: none; border-radius: 4px; cursor: pointer; display: flex; align-items: center; gap: 8px;">
    <svg width="18" height="18" viewBox="0 0 20 20" style="vertical-align: middle;"><path fill="currentColor" d="M10 6l-4 4h3v8h2v-8h3l-4-4zm-8-4v2h16V2H2z"/></svg>
    Upload Data
  </button>
</div>

### Ward Information
<form id="setup-form" onsubmit="return false;">
    <label>
        Number of Rooms:
        <input type="number" id="num-rooms" min="1" max="8" value="1" required>
    </label>
</form>
<div id="rooms-section" style="margin-top: 24px;"></div>

### Patient Information
<div id="stay-durations-section">
    <label for="stay-durations">
        Patient stay durations (comma separated):<br>
        <textarea id="stay-durations" name="stay-durations" rows="3" style="margin-top: 6px; width:340px; resize:vertical;"
            placeholder="e.g. 4, 7, 10, 12, 6, ..."
        ></textarea>
    </label>
    <div style="font-size:90%; color:#888;">
        Recommended: up to 30 values (in days, e.g. <span style="font-style:italic;">4, 7, 10, 12</span>)
    </div>
</div>

### Outbreak Information
<form id="calendar-form" onsubmit="return false;">
    <label>
        Start Date:
        <input type="date" id="start-date" required>
    </label>
    <br><br>
    <label>
        Number of Days:
        <input type="number" id="num-days" min="1" value="1" required>
    </label>
</form>
<div id="calendar-section" style="margin-top: 24px;"></div>

<style>
    table { border-collapse: collapse; margin-top: 20px; }
    th, td { border: 1px solid #ccc; padding: 8px 12px; text-align: center; }
    th { background: #f0f0f0; }
    input[type="number"] { width: 60px; }
</style>

<script>
    // --- ROOMS UI ---
    function generateRoomsUI() {
        const numRooms = parseInt(document.getElementById('num-rooms').value, 10);
        const roomsSection = document.getElementById('rooms-section');
        if (isNaN(numRooms) || numRooms < 1) {
            roomsSection.innerHTML = "<p>Please enter a valid number of rooms.</p>";
            return;
        }
        let html = `<table>
            <tr>
                <th>Room</th>
                <th>Number of Beds</th>
            </tr>`;
        for (let i = 0; i < numRooms; i++) {
            html += `<tr>
                <td>Room ${i + 1}</td>
                <td>
                    <input type="number" min="1" max="24 step="1" value="1" name="beds-room-${i}" id="beds-room-${i}" required>
                </td>
            </tr>`;
        }
        html += `</table>`;
        roomsSection.innerHTML = html;
        // Also regenerate cases table if it's already loaded
        generateCalendar();
    }

    // --- DATE FORMATTER ---
    function formatDate(date) {
        const days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
        const dayOfWeek = days[date.getDay()];
        const day = date.getDate();
        const daySuffix = (n) => {
            if (n > 3 && n < 21) return 'th';
            switch (n % 10) {
                case 1:  return "st";
                case 2:  return "nd";
                case 3:  return "rd";
                default: return "th";
            }
        };
        const month = date.toLocaleString('default', { month: 'long' });
        const year = date.getFullYear();
        return `${dayOfWeek} ${day}${daySuffix(day)} ${month} ${year}`;
    }

    // --- CASES CALENDAR ---
    function generateCalendar() {
        const startDateStr = document.getElementById('start-date').value;
        const numDays = parseInt(document.getElementById('num-days').value, 10);
        const numRooms = parseInt(document.getElementById('num-rooms').value, 10);
        const calendarSection = document.getElementById('calendar-section');

        if (!startDateStr || isNaN(numDays) || numDays < 1 || isNaN(numRooms) || numRooms < 1) {
            calendarSection.innerHTML = "<p>Please enter a valid start date, number of days, and number of rooms.</p>";
            return;
        }

        // Read number of beds per room
        let bedsPerRoom = [];
        for (let i = 0; i < numRooms; i++) {
            const el = document.getElementById(`beds-room-${i}`);
            bedsPerRoom.push(el ? el.value : 0);
        }

        const startDate = new Date(startDateStr);

        // Table header
        let html = `<table>
            <tr>
                <th>Date</th>`;
        for (let r = 0; r < numRooms; r++) {
            html += `<th>Room ${r + 1}<br><span style="font-weight:normal;font-size:90%;">(${bedsPerRoom[r]} beds)</span></th>`;
        }
        html += `</tr>`;

        // Table body
        for (let d = 0; d < numDays; d++) {
            const currDate = new Date(startDate);
            currDate.setDate(startDate.getDate() + d);
            html += `<tr>
                <td>${formatDate(currDate)}</td>`;
            for (let r = 0; r < numRooms; r++) {
                html += `<td>
                    <input type="number" min="0" max="${bedsPerRoom[r]}" step="1" value="0" name="cases-day${d}-room${r}" id="cases-day${d}-room${r}" required>
                </td>`;
            }
            html += `</tr>`;
        }
        html += `</table>`;
        calendarSection.innerHTML = html;
    }

    // --- ON LOAD ---
    document.addEventListener('DOMContentLoaded', function() {
        // Default today for start date
        const today = new Date();
        const yyyy = today.getFullYear();
        const mm = String(today.getMonth() + 1).padStart(2, '0');
        const dd = String(today.getDate()).padStart(2, '0');
        document.getElementById('start-date').value = `${yyyy}-${mm}-${dd}`;

        // Initial UI
        generateRoomsUI();
        generateCalendar();

        // Room/beds UI triggers
        document.getElementById('num-rooms').addEventListener('input', generateRoomsUI);
        // Also update beds -> calendar if beds change
        document.getElementById('rooms-section').addEventListener('input', generateCalendar);

        // Calendar controls
        document.getElementById('start-date').addEventListener('input', generateCalendar);
        document.getElementById('num-days').addEventListener('input', generateCalendar);
    });
</script>
