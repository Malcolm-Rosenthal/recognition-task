# Recognition Task — GitHub Pages App

Timed encoding slideshow + old/new recognition task for the cross-race effect experiment.
Embedded in LimeSurvey via iframe with postMessage for response collection.

---

## Repository structure

```
recognition-task/
│
├── index.html                  ← the app
│
└── images/
    ├── faces/
    │   ├── encoding/           ← 10 CFD images shown during study phase
    │   └── foils/              ← 10 CFD images shown only at recognition
    └── objects/
        ├── encoding/           ← 10 BOSS images shown during study phase
        └── foils/              ← 10 BOSS images shown only at recognition
```

---

## Image setup

### Faces — Chicago Face Database (CFD)
Download from: https://www.chicagofaces.org/

Recommended selection strategy:
- Balance race (Black, White, Asian, Latino) and sex across encoding and foil sets
- Use neutral expression images only (suffix `-N`)
- Keep a metadata CSV mapping filename → race → sex for use in analysis

Update the `STIMULI.faces.encoding` and `STIMULI.faces.foils` arrays in `index.html`
with your chosen filenames.

### Objects — BOSS (Bank of Standardized Stimuli)
Download from: https://sites.google.com/site/bosspictures/

Recommended: select objects matched for visual complexity and familiarity ratings
(BOSS provides norming data). Avoid objects strongly associated with any demographic group.

Update the `STIMULI.objects.encoding` and `STIMULI.objects.foils` arrays accordingly.

---

## LimeSurvey integration

### 1. URL parameters

Block order is randomised by a coin flip inside the app — no treatment parameter needed.
The iframe URL only needs the participant token:

```
https://YOUR_USERNAME.github.io/recognition-task/?token={TOKEN}
```

- `token`: LimeSurvey participant token for linking responses

In LimeSurvey, embed via a Long Free Text or Display question with HTML:

```html
<iframe
  id="rec-task-frame"
  src="https://YOUR_USERNAME.github.io/recognition-task/?treatment={TREATMENT_Q}&token={TOKEN}"
  width="100%"
  height="700"
  frameborder="0"
  scrolling="no">
</iframe>

<script>
window.addEventListener('message', function(e) {
  if (!e.data || e.data.source !== 'recognition_task') return;

  // Store full JSON results in a hidden Long Free Text question
  document.querySelector('input[name="RESULTS_FIELD"]').value =
    JSON.stringify(e.data.results);

  // Advance survey to next page after short delay
  setTimeout(function() {
    document.querySelector('.navigator-next').click();
  }, 1500);
});
</script>
```

Replace `RESULTS_FIELD` with your hidden question's field name in LimeSurvey.

### 2. Hidden questions needed in LimeSurvey

| Question | Type | Purpose |
|---|---|---|
| `RESULTS_Q` | Long Free Text / Hidden | Receives full JSON payload via postMessage |

No treatment question needed — the app randomises block order internally and reports it in the payload.

### 3. Standalone testing

Open `index.html` directly in a browser (or via a local server) with:

```
index.html?treatment=faces_first
```

Results will be logged to the browser console on completion.

---

## Output data format

The postMessage payload has this structure:

```json
{
  "token": "abc123",
  "treatment": "faces_first",
  "startTime": "2025-09-01T14:22:01Z",
  "endTime":   "2025-09-01T14:27:44Z",
  "blocks": {
    "faces": {
      "shownOrder": ["CFD-AF-200-001-N.jpg", "..."],
      "responses": {
        "CFD-AF-200-001-N.jpg": { "response": "old", "correct": true,  "isOld": true },
        "CFD-WF-200-001-N.jpg": { "response": "new", "correct": true,  "isOld": false }
      }
    },
    "objects": { "..." }
  }
}
```

---

## Analysis notes

- **Hit rate** = correct "old" responses to old items (per block, per participant)
- **False alarm rate** = "old" responses to new foil items
- **d′ (d-prime)** = z(hit rate) − z(false alarm rate) — the standard SDT sensitivity measure
- Link participant demographics (from LimeSurvey) to face stimulus metadata (race/sex from CFD)
  to compute cross-race effect: own-race d′ vs. other-race d′

---

## Customisation

| Setting | Location in index.html |
|---|---|
| Encoding duration | `ENCODING_DURATION_MS` constant |
| Distractor problems | `DISTRACTOR_PROBLEMS` array |
| Image filenames | `STIMULI` object |
| Image base path | `IMAGE_BASE` constant |
