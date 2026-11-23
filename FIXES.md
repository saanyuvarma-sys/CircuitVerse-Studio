# CircuitVerse Studio — Bug Fixes & Improvements

This document lists every change made to `index.html`, why it was needed, and how it was verified.

---

## 1. CRITICAL — Run button (and the entire lab toolbar) was dead — LEDs never lit

**Symptom (the reported bug):** In the Electronics Lab you could place components and wire them, but pressing **Run** did nothing. The LED never turned on, no matter how the circuit was connected.

**Root cause:** One character. In `bindToolbar()` the mode-toggle segment was looked up with

```js
$('#emode-seg').addEventListener(...)      // '#emode-seg' passed to getElementById
```

`$` in that function is `id => document.getElementById(id)`, so the code actually called `getElementById('#emode-seg')`. IDs never contain `#`, so this returned `null` and the line threw:

```
TypeError: Cannot read properties of null (reading 'addEventListener')
    at E.bindToolbar (index.html:2218)
```

The error was swallowed by the `try/catch` around `bindToolbar()` in `mountPage()`, so nothing looked broken — but `bindToolbar` died **on its first line**, meaning **none** of the toolbar handlers were ever bound: Run, Pause, Stop, Reset, Rotate, Delete, Validate, PNG, the project selector, Save, zoom buttons and the panel tabs were all inert. That is why simulation never started and the light never came on.

**Fix (line 2218):**

```diff
-    $('#emode-seg').addEventListener('click', e => {
+    $('emode-seg').addEventListener('click', e => {
```

**Verified:** headless-browser end-to-end test — wire D13 → 220Ω resistor → LED(A), LED(K) → GND, set D13 OUTPUT HIGH, press Run: LED lights, log shows `LED "LED" turned ON.`, Mission 01 "First Light" (+50 XP) completes. Also verified with a 5V source circuit, a push-button circuit (hold to light, release to go dark, Mission 02 completes) and the "Blink D13" preset.

## 2. Panel tabs (PINS / LOG / ISSUES) rendered blank on the first click

**Root cause:** `E.panelTab(t)` called the render helpers **before** setting `body.dataset.tab = t`. Every render helper early-returns unless `body.dataset.tab` already equals its own tab, so the very first click on a tab produced an empty panel (it only worked from the second click on).

**Fix (in `E.panelTab`):** move `body.dataset.tab = t;` above the four render calls.

```diff
     const body = document.getElementById('epanel-body');
     if (!body) return;
+    body.dataset.tab = t;
     if (t === 'insp') E.renderInspector();
     if (t === 'pins') E.renderPins();
     if (t === 'log') E.renderLog();
     if (t === 'issues') E.renderIssues();
-    body.dataset.tab = t;
```

**Verified:** first click on PINS now renders all 12 pin rows + presets; first click on LOG renders the log; first click on ISSUES renders validation output.

## 3. Duplicate short-circuit error messages

**Root cause:** a net that is both driven and grounded was pushed into `st.shorts` twice (once by the generic `drivers ∩ grounds` sweep, once by the source-specific check), so Validate printed the same error twice.

**Fix (in `E.validate`):**

```diff
-    st.shorts.forEach(c => issues.push({sev: 'error', msg: 'Short circuit: ...'}));
+    Array.from(new Set(st.shorts)).forEach(c => issues.push({sev: 'error', msg: 'Short circuit: ...'}));
```

## 4. Crash guards for corrupt / imported circuit data

- `wireAt()` now skips wires whose endpoints reference deleted components (previously a dangling wire in imported data threw while hit-testing and blocked all canvas interaction).

```diff
 function wireAt(w) {
   for (const wd of E.wires) {
+    const ca = E.compById(wd.a.cid), cb = E.compById(wd.b.cid);
+    if (!ca || !cb) continue; // dangling wire — skip instead of crashing
     const a = E.pinPos(E.compById(wd.a.cid), ...);
```

- `drawScene()`, `fitView()` and `renderPNG()` now skip components with an unknown `key` instead of throwing on `DEFS[key].w`.
- `buildNets()` ignores unknown component keys instead of crashing the whole simulation loop.

## 5. Layout / alignment improvements

- **Lab toolbar:** removed the dead `<span style="flex:1">` spacer; the project selector + Save now form a right-aligned group (`.lab-proj`) that wraps as a unit. Tightened gaps and added `row-gap` so on narrow screens the toolbar breaks into clean rows instead of orphaning one button. The selector also can't overflow (`max-width:230px`).
- **Stage hint:** `max-width: calc(100% - 150px)` so the hint text can no longer run underneath the zoom HUD and get cut off.
- **Component Library header:** "drag → canvas" caption no longer wraps onto two lines (`white-space:nowrap`).
- **Achievements page:** missions were in a fixed 2-column layout (2 Beginner cards on the left, 4 Intermediate/Advanced on the right) which left a large empty hole. Now each tier (Beginner / Intermediate / Advanced) is its own section with a responsive card grid — balanced columns, no dead space.
- **Profile page:** the profile card is now centred (`margin:0 auto`) instead of hugging the left edge.
- **Breadcrumb:** `.crumb` gets ellipsis overflow so long crumbs like "LAB 01 · ELECTRONICS · DESIGN. CONNECT. BUILD." don't wrap to three lines on mobile.

---

## Verification summary

All checks run against the fixed file in a headless Chromium session (Playwright):

| Check | Result |
| --- | --- |
| All 13 script blocks pass `node --check` | pass |
| Run / Pause / Stop / Reset / Validate / Save / zoom handlers bound | pass |
| LED lights: Arduino D13 → resistor → LED → GND with D13 OUTPUT HIGH | pass |
| LED lights: 5V source → resistor → LED → Ground | pass |
| Push button (momentary) lights LED while held, mission awarded | pass |
| Blink D13 preset toggles LED | pass |
| Short circuit detected & reported once | pass |
| Panel tabs render content on first click | pass |
| Stop records session into Mission Control | pass |
| Project save / link / autosave flow | pass |
| All 10 routes render, zero console errors / warnings | pass |
| Mobile (390px) electronics lab layout | pass |
