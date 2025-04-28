---
layout: default
title: HW-OINK
permalink: /oink/hw/
---

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
     style="padding: 12px 24px; font-size: 16px; border: none; border-radius: 4px; cursor: pointer; display: flex; align-items: center; gap: 8px; background: #007bff; color: white; text-decoration: none;">
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

<script>
document.getElementById('upload-xlsx').addEventListener('change', function(e) {
  const file = e.target.files[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = function(evt) {
    const data = new Uint8Array(evt.target.result);
    const workbook = XLSX.read(data, {type: 'array'});

    // Assume first sheet
    const sheet = workbook.Sheets[workbook.SheetNames[0]];
    const json = XLSX.utils.sheet_to_json(sheet, {header:1});

    // 1. Parse number of rooms from ROOM table (A2:A9, blank for unused)
    let roomRows = [];
    for (let i = 1; i < 9; ++i) {
      const room = json[i] ? json[i][0] : "";
      const beds = json[i] ? json[i][1] : "";
      if (room && beds) roomRows.push({room, beds});
    }
    document.getElementById('num-rooms').value = roomRows.length;
    generateRoomsUI();
    roomRows.forEach((row, idx) => {
      const bedInput = document.getElementById('beds-room-' + idx);
      if (bedInput) bedInput.value = row.beds;
    });

    // 2. Patient stays from C2:C9 (match number of rooms)
    let stays = [];
    for (let i = 1; i <= roomRows.length; ++i) {
      if (json[i] && json[i][2]) stays.push(json[i][2]);
    }
    document.getElementById('stay-durations').value = stays.join(', ');

    // 3. Outbreak info: F2:Fx (dates), G2:...N2:... (cases)
    // Find where DATE row starts (header 'DATE')
    let dateRow = -1;
    for (let i = 0; i < json.length; ++i) {
      if (json[i] && json[i][5] && json[i][5].toString().toUpperCase().includes("DATE")) {
        dateRow = i;
        break;
      }
    }
    if (dateRow > 0) {
      let dates = [];
      for (let i = dateRow + 1; i < json.length; ++i) {
        const row = json[i];
        if (!row || !row[5]) break;
        dates.push(row[5]);
      }
      if (dates.length) {
        // set start-date to first date, num-days to count
        document.getElementById('start-date').value = new Date(dates[0]).toISOString().slice(0,10);
        document.getElementById('num-days').value = dates.length;
        generateCalendar();

        // Populate cases
        for (let d = 0; d < dates.length; ++d) {
          for (let r = 0; r < roomRows.length; ++r) {
            // G = index 6
            const val = (json[dateRow + 1 + d][6 + r]) || 0;
            const input = document.getElementById(`cases-day${d}-room${r}`);
            if (input) input.value = val;
          }
        }
      }
    }
  };
  reader.readAsArrayBuffer(file);
});
</script>
