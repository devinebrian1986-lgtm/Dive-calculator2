<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>NOAA Dive Calculator (Surface → Bottom, Group Letter)</title>
  <style>
    body { font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; max-width: 860px; margin: 24px auto; padding: 0 16px; }
    .card { border: 1px solid #ddd; border-radius: 12px; padding: 16px; margin: 12px 0; }
    label { display:block; margin: 10px 0 6px; font-weight: 600; }
    input, select { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 10px; font-size: 16px; }
    button { padding: 10px 14px; border: 0; border-radius: 10px; font-size: 16px; cursor: pointer; }
    button { background: #111; color: #fff; }
    .row { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
    .out { font-size: 18px; line-height: 1.5; }
    .muted { color: #555; font-size: 13px; line-height: 1.4; }
    code { background: #f6f6f6; padding: 2px 6px; border-radius: 6px; }
  </style>
</head>
<body>

  <h1>NOAA Dive Calculator</h1>
  <p class="muted">
    Computes time from <b>left surface → left bottom</b> (descent + time at depth until you start ascent),
    and the NOAA <b>Repetitive Group Letter (RGD)</b> after the dive using NOAA Chart 1.
  </p>

  <div class="card">
    <div class="row">
      <div>
        <label for="depth">Max depth (fsw)</label>
        <input id="depth" type="number" min="1" step="1" value="60" />
      </div>
      <div>
        <label for="descentRate">Descent rate (ft/min)</label>
        <input id="descentRate" type="number" min="1" step="1" value="60" />
      </div>
    </div>

    <label for="timeAtDepth">Time at depth until leaving bottom (min)</label>
    <input id="timeAtDepth" type="number" min="0" step="1" value="25" />

    <label for="rounding">Rounding</label>
    <select id="rounding">
      <option value="ceil" selected>Round UP (conservative)</option>
      <option value="none">No rounding</option>
    </select>

    <div style="margin-top:12px;">
      <button id="calcBtn">Calculate</button>
    </div>
  </div>

  <div class="card">
    <div class="out" id="output">Enter values and click Calculate.</div>
    <p class="muted">
      NOAA Chart 1 depths supported here: <code>40, 45, 50, 55, 60, 70, 80, 90, 100, 110, 120, 130 fsw</code>.
      If your depth is between rows, the calculator rounds up to the next deeper row (conservative).
    </p>
  </div>

<script>
/**
 * NOAA "Chart 1 – Dive times with end-of-dive group letter" (subset as shown on NOAA Multiple Air Dives sheet).
 * Depth (fsw) -> list of max dive times (minutes) for group letters A..(last provided at that depth).
 *
 * Notes:
 * - This is the no-decompression section (white cells). If your time exceeds the last value for that depth,
 *   it's beyond the no-stop limit for this chart.
 */
const NOAA_CHART1 = {
  40:  [12,20,27,36,44,53,63,73,84,95,108,121,135,151,163], // A..O
  45:  [11,17,24,31,39,46,55,63,72,82,92,102,114,125],     // A..N
  50:  [9,15,21,28,34,41,48,56,63,71,80,89,92],            // A..M
  55:  [8,14,19,25,31,37,43,50,56,63,71,74],               // A..L
  60:  [7,12,17,22,28,33,39,45,51,57,60],                  // A..K
  70:  [6,10,14,19,23,28,32,37,42,47,48],                  // A..K (48 is max no-stop shown)
  80:  [5,9,12,16,20,24,28,32,36,39],                      // A..J (39 max)
  90:  [4,7,11,14,17,21,24,28,30],                         // A..I (30 max)
  100: [4,6,9,12,15,18,21,25],                             // A..H (25 max)
  110: [3,6,8,11,14,16,19,20],                             // A..H (20 max)
  120: [3,5,7,10,12,15],                                   // A..F (15 max)
  130: [2,4,6,9,10]                                        // A..E (10 max)
};

const GROUPS = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".split(""); // We'll use A..Z as needed

function roundUpIf(value, roundingMode) {
  return (roundingMode === "ceil") ? Math.ceil(value) : value;
}

function nextDeeperSupportedDepth(depthFsw) {
  const depths = Object.keys(NOAA_CHART1).map(Number).sort((a,b)=>a-b);
  for (const d of depths) if (depthFsw <= d) return d;
  return null; // deeper than supported
}

function findGroupLetter(depthRow, diveMinutes) {
  const limits = NOAA_CHART1[depthRow];
  for (let i = 0; i < limits.length; i++) {
    if (diveMinutes <= limits[i]) return GROUPS[i]; // A for i=0, B for i=1, ...
  }
  return null; // beyond no-stop for this row
}

function fmtMinutes(min) {
  const m = Math.round(min * 100) / 100;
  return (Number.isInteger(m)) ? `${m} min` : `${m.toFixed(2)} min`;
}

document.getElementById("calcBtn").addEventListener("click", () => {
  const depth = Number(document.getElementById("depth").value);
  const descentRate = Number(document.getElementById("descentRate").value);
  const timeAtDepth = Number(document.getElementById("timeAtDepth").value);
  const rounding = document.getElementById("rounding").value;

  if (!(depth > 0) || !(descentRate > 0) || !(timeAtDepth >= 0)) {
    document.getElementById("output").innerText = "Please enter valid positive numbers.";
    return;
  }

  // Time from left surface -> left bottom = descent time + time at depth until you leave bottom
  const descentTime = depth / descentRate;
  const totalSurfaceToBottom = descentTime + timeAtDepth;

  // NOAA Chart 1 lookup uses depth row (rounded up to next deeper supported row, conservative)
  const depthRow = nextDeeperSupportedDepth(depth);
  if (depthRow === null) {
    document.getElementById("output").innerHTML =
      `Depth ${depth} fsw is deeper than this calculator’s NOAA Chart 1 range (max 130 fsw).`;
    return;
  }

  const lookupMinutes = roundUpIf(totalSurfaceToBottom, rounding);
  const group = findGroupLetter(depthRow, lookupMinutes);

  let html = "";
  html += `<b>Inputs</b><br/>`;
  html += `Max depth: ${depth} fsw (Chart row used: <b>${depthRow} fsw</b>)<br/>`;
  html += `Descent rate: ${descentRate} ft/min → descent time: <b>${fmtMinutes(descentTime)}</b><br/>`;
  html += `Time at depth until leaving bottom: <b>${fmtMinutes(timeAtDepth)}</b><br/><br/>`;

  html += `<b>Results</b><br/>`;
  html += `Total time (left surface → left bottom): <b>${fmtMinutes(totalSurfaceToBottom)}</b><br/>`;
  html += `Time used for NOAA Chart 1 lookup: <b>${lookupMinutes} min</b> (${rounding === "ceil" ? "rounded up" : "not rounded"})<br/>`;

  if (group) {
    html += `End-of-dive repetitive group (NOAA RGD): <b>${group}</b><br/>`;
  } else {
    html += `<b>Beyond no-stop limit</b> for ${depthRow} fsw on NOAA Chart 1 at ${lookupMinutes} min (would require decompression planning).<br/>`;
  }

  document.getElementById("output").innerHTML = html;
});
</script>

</body>
</html>
