---
layout: default
title: HW-OINK
permalink: /oink/hw/
---

<style>
  #virus-select {
    background-color: white;
    border: 1px solid #90909C;
    border-radius: 6px;
    transition: border-color 0.2s;
  }
  #virus-select:hover, #virus-select:focus {
    border-color: #676774;
  }
</style>

# Hospital Ward (HW)-OINK (v0.1)
<div style="font-size: 0.95em;">
  This webtool provides model-based estimates of transmission dynamics for respiratory virus outbreaks in hospital wards. The method has been adapted from the original OINK (Outbreak Inference with Negligible Knowledge) model described in <a href="https://doi.org/10.1098/rsif.2024.0168" target="_blank" rel="noopener noreferrer" style="color: #159957;">Fozard et al., (2024)</a>.
  <br><br>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

## Data Entry
### Upload Existing Data
<div style="display: flex; gap: 16px; margin-bottom: 16px;">
  <a href="/assets/files/HWOINK_Template.xlsx" download
     style="padding: 12px 24px; font-size: 16px; border: none; border-radius: 4px; cursor: pointer; display: flex; align-items: center; gap: 8px; background: #2ecc71; color: white; text-decoration: none;">
    <svg width="18" height="18" viewBox="0 0 20 20" style="vertical-align: middle;">
      <path fill="currentColor" d="M10 14l4-4h-3V2h-2v8H6l4 4zm-8 4v-2h16v2H2z"/>
    </svg>
    Download Template
  </a>
<input type="file" id="upload-xlsx" accept=".xlsx" style="display:none;">
<button id="upload-btn" type="button" style="padding: 12px 24px; font-size: 16px; border: none; border-radius: 4px; cursor: pointer; display: flex; align-items: center; gap: 8px;">
  <svg width="18" height="18" viewBox="0 0 20 20" style="vertical-align: middle;">
    <path fill="currentColor" d="M10 6l-4 4h3v8h2v-8h3l-4-4zm-8-4v2h16V2H2z"/>
  </svg>
  Upload Data
</button>
</div>

<script>
document.getElementById('upload-btn').onclick = function() {
  document.getElementById('upload-xlsx').click();
};
</script>
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
        Patient stay durations (days, comma separated):<br>
        <textarea id="stay-durations" name="stay-durations" rows="3" style="margin-top: 6px; width:420px; resize:vertical;"
            placeholder="e.g. 4, 7, 10, 12, 6, ..."
        ></textarea>
    </label>
    <div style="font-size:90%; color:#888;">
        Recommended: 20-30 values
    </div>
</div>

### Outbreak Information
<label>
    Virus:
    <select id="virus-select">
        <option value="Influenza A virus">Influenza A virus</option>
        <option value="SARS-CoV-2">SARS-CoV-2</option>
    </select>
</label>

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
<div style="font-size:90%; color:#888;">
        Note: Record only the day of detection of each new case.
    </div>

<style>
    table { border-collapse: collapse; margin-top: 20px; }
    th, td { border: 1px solid #ccc; padding: 8px 12px; text-align: center; }
    th { background: #f0f0f0; font-weight: normal; } /* Remove bold */
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
    // Gather beds per room values (default to 1 if not present)
    let bedsPerRoom = [];
    for (let i = 0; i < numRooms; i++) {
        const el = document.getElementById(`beds-room-${i}`);
        bedsPerRoom.push(el ? el.value : 1);
    }

    // Table: First column for row labels, no beds-per-room in header
    let html = `<table>
        <tr>
            <th></th>`;
    for (let i = 0; i < numRooms; i++) {
        html += `<th>Room ${i + 1}</th>`;
    }
    html += `</tr>
        <tr>
            <td>Number of Beds</td>`;
    for (let i = 0; i < numRooms; i++) {
        html += `<td>
            <input type="number" min="1" max="24" step="1" value="${bedsPerRoom[i]}" name="beds-room-${i}" id="beds-room-${i}" required>
        </td>`;
    }
    html += `</tr></table>`;
    roomsSection.innerHTML = html;
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

    // Table header: two rows, with "Cases" spanning all room columns
    let html = `<table>
        <tr>
            <th style="background: transparent; border: none; padding: 8px 12px;">&nbsp;</th>
            <th colspan="${numRooms}">Cases</th>
        </tr>
        <tr>
            <th>Date</th>`;
    for (let r = 0; r < numRooms; r++) {
        const beds = bedsPerRoom[r] || 0;
        html += `<th>Room ${r + 1}<br><span style="font-weight: normal;">(${beds} bed${beds != 1 ? 's' : ''})</span></th>`;
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

<script>
function excelDateToISO(val) {
    if (typeof val === "number") {
        // Excel serial number date
        // Excel's epoch is Jan 1, 1900
        const jsDate = new Date(Date.UTC(1899, 11, 30) + val * 86400000);
        return jsDate.toISOString().slice(0, 10);
    } else if (typeof val === "string") {
        // String date (try to parse)
        // Accepts: "1/1/2025", "2025-01-01", etc.
        const jsDate = new Date(val);
        if (!isNaN(jsDate)) {
            return jsDate.toISOString().slice(0, 10);
        }
    }
    return "";
}
  
document.getElementById('upload-xlsx').addEventListener('change', function(e) {
  const file = e.target.files[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = function(evt) {
    const data = new Uint8Array(evt.target.result);
    const workbook = XLSX.read(data, {type: 'array'});
    const sheet = workbook.Sheets[workbook.SheetNames[0]];
    const json = XLSX.utils.sheet_to_json(sheet, {header:1});

    // 1. Number of rooms = non-empty values in B2:B9
    let numRooms = 0;
    let bedsPerRoom = [];
    for (let i = 1; i <= 8; i++) { // B2:B9 => json[1][1] to json[8][1]
      const beds = json[i]?.[1];
      if (beds !== undefined && beds !== "" && beds != null) {
        numRooms++;
        bedsPerRoom.push(beds);
      } else {
        break;
      }
    }
    document.getElementById('num-rooms').value = numRooms;
    generateRoomsUI();
    // Set beds per room
    for (let i = 0; i < numRooms; ++i) {
      const bedInput = document.getElementById('beds-room-' + i);
      if (bedInput) bedInput.value = bedsPerRoom[i];
    }

    // 2. Stay durations: D2 downwards, until a blank
    let stayDurations = [];
    for (let i = 1; i < json.length; ++i) { // D2 => json[1][3]
      const stay = json[i]?.[3];
      if (stay !== undefined && stay !== "" && stay != null) {
        stayDurations.push(stay);
      } else {
        break;
      }
    }
    document.getElementById('stay-durations').value = stayDurations.join(', ');

    // 3. Virus-select: F2 (json[1][5])
    const virusVal = json[1]?.[5];
    if (virusVal) {
      const select = document.getElementById('virus-select');
      for (let i = 0; i < select.options.length; i++) {
        if (select.options[i].value === virusVal) {
          select.selectedIndex = i;
          break;
        }
      }
    }

    // 4. Start date: H2 (json[1][7])
    const startDate = json[1]?.[7];
    if (startDate) {
      const isoDate = excelDateToISO(startDate);
      if (isoDate) document.getElementById('start-date').value = isoDate;
    }

    // 5. Number of days: count rows from H2 down until blank
    let numDays = 0;
    for (let i = 1; i < json.length; ++i) {
      const dayVal = json[i]?.[7]; // H
      if (dayVal !== undefined && dayVal !== "" && dayVal != null) {
        numDays++;
      } else {
        break;
      }
    }
    document.getElementById('num-days').value = numDays;
    generateCalendar();

    // 6. Cases by date for each room (I to P = columns 8 to 15, zero-based)
    for (let day = 0; day < numDays; ++day) {
      const row = json[1 + day];
      for (let r = 0; r < numRooms; ++r) {
        const casesVal = row?.[8 + r] ?? 0; // I is 8, J is 9, ..., P is 15
        const input = document.getElementById(`cases-day${day}-room${r}`);
        if (input) input.value = casesVal;
      }
    }
  };
  reader.readAsArrayBuffer(file);
});

