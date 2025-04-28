---
layout: default
title: HW-OINK
permalink: /oink/hw/
---

## Hospital Ward (HW)-OINK (v0.1)
<div style="font-size: 0.95em;">This webtool provides model-based estimates of transmission dynamics for respiratory virus outbreaks in hospital wards.<br><br></div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

### Ward Information
<form id="room-form" onsubmit="return false;">
    <label>
        Number of Rooms:
        <input type="number" id="num-rooms" min="1" value="1" required>
    </label>
</form>
<div class="rooms-section" id="rooms-section"></div>

<style>
    .rooms-section table { border-collapse: collapse; margin-top: 20px; }
    .rooms-section th, .rooms-section td { border: 1px solid #ccc; padding: 8px 12px; }
    .rooms-section th { background: #f0f0f0; }
    .rooms-section input[type="number"] { width: 60px; }
</style>

<script>
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
                    <input type="number" min="0" step="1" value="0" name="beds-room-${i}" id="beds-room-${i}" required>
                </td>
            </tr>`;
        }
        html += `</table>`;
        roomsSection.innerHTML = html;
    }

    // Set up event listeners and initial rendering
    document.addEventListener('DOMContentLoaded', function() {
        generateRoomsUI();
        document.getElementById('num-rooms').addEventListener('input', generateRoomsUI);
    });
</script>

### Outbreak Information
<form id="setup-form" onsubmit="return false;">
    <label>
        Date of First Detection:
        <input type="date" id="start-date" required>
    </label>
    <br><br>
    <label>
        Number of Days:
        <input type="number" id="num-days" min="1" value="1" required>
    </label>
</form>
<div class="calendar-section" id="calendar-section"></div>

<style>
    table { border-collapse: collapse; margin-top: 20px; }
    th, td { border: 1px solid #ccc; padding: 8px 12px; }
    th { background: #f0f0f0; }
    input[type="number"] { width: 60px; }
    .calendar-section { margin-top: 20px; }
</style>

<script>
    // Helper to format dates
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

    function generateCalendar() {
        const startDateStr = document.getElementById('start-date').value;
        const numDays = parseInt(document.getElementById('num-days').value, 10);
        const calendarSection = document.getElementById('calendar-section');

        if (!startDateStr || isNaN(numDays) || numDays < 1) {
            calendarSection.innerHTML = "<p>Please enter a valid start date and number of days.</p>";
            return;
        }

        const startDate = new Date(startDateStr);

        // Create table
        let html = `<table>
            <tr>
                <th>Date</th>
                <th>Number of Cases</th>
            </tr>`;

        for (let i = 0; i < numDays; i++) {
            const currDate = new Date(startDate);
            currDate.setDate(startDate.getDate() + i);
            html += `<tr>
                <td>${formatDate(currDate)}</td>
                <td>
                    <input type="number" min="0" step="1" value="0" name="cases-day-${i}" id="cases-day-${i}" required>
                </td>
            </tr>`;
        }
        html += '</table>';
        calendarSection.innerHTML = html;
    }

    // Set today's date as default in yyyy-mm-dd format
    document.addEventListener('DOMContentLoaded', function() {
        const today = new Date();
        const yyyy = today.getFullYear();
        const mm = String(today.getMonth() + 1).padStart(2, '0');
        const dd = String(today.getDate()).padStart(2, '0');
        document.getElementById('start-date').value = `${yyyy}-${mm}-${dd}`;
        generateCalendar();

        document.getElementById('start-date').addEventListener('input', generateCalendar);
        document.getElementById('num-days').addEventListener('input', generateCalendar);
    });
</script>
