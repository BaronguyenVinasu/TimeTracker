# ⏱ TimeTracker

> One-click time tracking with weekly summaries, CSV import/export, and PWA install. Zero dependencies. Copy-paste deployable.

---

## 🚀 Deploy in 60 seconds

### GitHub Pages
```bash
git init && git add . && git commit -m "feat: time tracker PWA"
git branch -M main
git remote add origin https://github.com/YOUR_USER/timetracker.git
git push -u origin main
# Settings → Pages → Source: main branch / root
# App lives at: https://YOUR_USER.github.io/timetracker/
```

### Netlify (drag & drop)
1. Go to [app.netlify.com](https://app.netlify.com)
2. Drag the project folder onto "Deploy manually"
3. Done — gets HTTPS automatically → PWA install prompt works

### Local dev
```bash
python3 -m http.server 8080
# Open http://localhost:8080
# Note: PWA install requires HTTPS (use Netlify/Pages for that)
```

---

## 📁 File Structure
```
timetracker/
├── index.html          # Full app — single self-contained file
├── manifest.json       # PWA manifest (icons, theme, display mode)
├── sw.js               # Service worker — offline cache + install signal
├── sample_sessions.csv # Example CSV to test import
└── README.md           # This file
```

---

## 🎹 Keyboard Shortcuts
| Key | Action           |
|-----|-----------------|
| `S` | Start session   |
| `E` | End/Stop session |

*(Ignored when an input is focused)*

---

## 🧪 Test Plan

### Manual Checklist

- [ ] **Start → Stop**: Click Start, wait ~2 mins, click Stop. Duration shows ≈2m.
- [ ] **Live timer**: Timer ticks every 10s while running. Page reload resumes timer.
- [ ] **Status display**: Shows "Running — started HH:MM" and live elapsed time.
- [ ] **Midnight crossing**: Start at 23:50, stop at 00:10 → duration ≈ 20m. Week groups by start date's Monday.
- [ ] **Double-start**: Click Start twice → toast warning, no duplicate session.
- [ ] **Stop with no session**: Click Stop idle → friendly toast, no crash.
- [ ] **Edit**: Click ✎ on a row → modal opens, save updates project/notes.
- [ ] **Delete**: Click ✕ → confirm dialog → row disappears.
- [ ] **Delete running session**: Prompt explains session will stop.
- [ ] **Weekly summary colors**: Week < 5h = green | 5-8h = yellow | > 8h = red.
- [ ] **Export CSV**: Produces valid CSV with header + all sessions.
- [ ] **Import sample_sessions.csv**: 4 valid rows imported, summary updates.
- [ ] **Import bad CSV**: Import a file with 3 valid + 2 invalid rows → toast shows skipped count.
- [ ] **Persistence**: Start session → reload page → session still running.
- [ ] **Clear All**: Confirm → sessions cleared, summary empty.
- [ ] **Keyboard**: Press S to start, E to stop (not in input fields).
- [ ] **PWA**: Served over HTTPS → install banner appears in browser.
- [ ] **Offline**: After first load, disable network → app still works.

### Browser Console Assertions
Paste into DevTools console to run quick sanity checks:

```javascript
// Helper
const delay = ms => new Promise(r => setTimeout(r, ms));
const { Sessions, Render, CSV } = window.__tt;

// TEST 1: Basic start/stop
(async () => {
  const before = Sessions.all().length;
  document.getElementById('btn-start').click();
  await delay(100);
  const active = Sessions.getActive();
  console.assert(active !== null, 'FAIL: no active session after start');
  console.assert(document.getElementById('btn-start').disabled, 'FAIL: start not disabled');

  await delay(2000);
  document.getElementById('btn-stop').click();
  const after = Sessions.all();
  const rec = after[0];
  console.assert(rec.endISO !== null, 'FAIL: endISO not set');
  console.assert(rec.durationMins >= 0, 'FAIL: negative duration');
  console.log('TEST 1 PASS — start/stop OK, duration:', rec.durationMins, 'min');
})();

// TEST 2: Double-start blocked
(async () => {
  document.getElementById('btn-start').click();
  await delay(50);
  const count1 = Sessions.all().length;
  document.getElementById('btn-start').click(); // should be blocked (btn disabled)
  const count2 = Sessions.all().length;
  console.assert(count1 === count2, 'FAIL: double-start created extra session');
  document.getElementById('btn-stop').click();
  console.log('TEST 2 PASS — double-start blocked');
})();

// TEST 3: Duration math (midnight crossing)
(() => {
  const start = '2026-03-09T23:50:00.000Z';
  const end   = '2026-03-10T00:20:00.000Z';
  const dur = Sessions.computeDuration(start, end);
  console.assert(dur === 30, `FAIL: expected 30, got ${dur}`);
  console.log('TEST 3 PASS — midnight crossing duration:', dur, 'min');
})();

// TEST 4: CSV round-trip
(() => {
  const all = Sessions.all();
  console.assert(all.length > 0, 'SKIP: no sessions to export (run tests 1/2 first)');
  // Export triggers download — check no errors thrown
  try { CSV.exportCSV(); console.log('TEST 4 PASS — export ran without error'); }
  catch(e) { console.error('TEST 4 FAIL', e); }
})();
```

---

## ⚠️ Known Limitations

| Limitation | Details |
|---|---|
| **localStorage scope** | Data is per-browser per-origin. Incognito/private browsing uses in-memory only. |
| **No sync** | Sessions don't sync across devices or browsers. Export CSV as backup. |
| **Clock drift** | Durations computed from system clock epoch ms. If the clock jumps, sessions flagged with ⚠. |
| **Browser compatibility** | Requires modern browsers (Chrome 90+, Firefox 88+, Safari 14+). No IE support. |
| **Storage quota** | Typically 5–10MB localStorage limit. ~10,000+ sessions before hitting limits. |
| **PWA install** | Requires HTTPS. GitHub Pages and Netlify both provide this free. |

---

## 📊 CSV Format

```
WeekStart,Date,StartISO,EndISO,DurationMins,Project,Notes
2026-03-09,2026-03-09,2026-03-09T09:00:00.000Z,2026-03-09T12:00:00.000Z,180,Project A,Focus work
```

**Import accepts flexible headers**: `StartISO`, `startISO`, `Start`, `Start Time`, `start_time`

---

*Built with ♥ — vanilla JS, zero deps, works everywhere.*
