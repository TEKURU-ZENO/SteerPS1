# The 4pm test

**Steer Hackathon, Problem Statement 1: the ad people want to tap**
Built by Dev Mehta (solo)

A 45-second mobile quiz that works as a Meta playable ad and as an entry point on the Steer website. It catches people at a moment they already know (4pm hunger), clears up the five doubts people have about weight loss and GLP-1 medicines, gives them a personal note, and hands them to a Steer doctor with their answers already attached.

| | |
|---|---|
| **Live link** | https://steer-4pm-test.netlify.app *(replace with your deployed URL)* |
| **Build** | `index.html`, a single self-contained file (~165 KB) |
| **Walkthrough** | Loom: *(add link)* · Screenshots of every screen: `walkthrough/` |
| **Sources** | `SOURCES.md`, also shown inside the app via the "Sources" link |

---

## The brief in three lines

| | |
|---|---|
| **Trigger** | "4:00 pm. Lunch was two hours ago. So why are you hungry again?" A moment every Indian desk worker knows, posed as a guess with three answers to tap. Curiosity does the work, not a sales line. |
| **Value** | In under a minute the user busts five myths (willpower, "only for diabetics", muscle loss, regain after stopping, Indian BMI), each with its source. Then they get a personal note with three tips matched to their worry and routine. They leave smarter, not sold to. |
| **Next step** | "See if the programme fits you", then a no-obligation call with a Steer doctor who already knows their worry, routine and which myths they believed. The first call starts with their health, not the basics. |

---

## From "today" to "ideal"

| | Today | With The 4pm test |
|---|---|---|
| The ad | A standard Meta ad with a lead form | A quiz people choose to play |
| What the user knows when they tap | Little, plus myths and fears | Their top doubts are cleared, with sources |
| The lead | Cold: a name and number | Warm: worry, routine, myths believed, score |
| The team's first call | Spent explaining basics | Spent on their health and fit |
| Reuse | Lives only on Meta | The same file runs on Meta (`?src=meta`) and the website (`?src=web`) |

---

## The flow, screen by screen

```
Hook  ->  5 myth or true cards  ->  biggest worry  ->  usual day
      ->  personal "doctor's note"  ->  doctor handoff  ->  warm-lead summary
```

1. **Hook (maroon, 4pm).** A big "4:00 pm", a drawn cutting chai glass and an afternoon window-light background. Three guesses: "I just have no willpower", "Blame the chai biscuits", "Something in my body". Their pick gets a tick, the others collapse, and a short sourced answer on ghrelin appears with the CTA "Bust the 5 myths".
2. **Myth or true? (cream).** Five swipe cards. Swipe left for myth and right for true, or tap the buttons. Each card flips: myths land on maroon, truths on paper, each with a two-line answer and its source. Five dots fill rust when you're right and blush when you're wrong.
3. **Biggest worry (blush).** "Now, make it about you." Safety / losing muscle / gaining it back / not sure it's for me.
4. **Usual day (blush).** One screen of chips: weekday (desk / on my feet / running the home), dinner time, and what they've tried before.
5. **Doctor's note (cream).** A ruled prescription-pad card with a stamped score (e.g. 3/5), a title matched to their worry, and three tips picked from their answers. Every tip that states a fact is cited.
6. **Handoff.** "Before you start anything, hear it from a doctor." Steer's three promises sit above the form: tests first, a real doctor decides, no obligation. The form asks for name, phone and explicit consent.
7. **Warm-lead summary.** "So you don't start from zero, your doctor will already know this": worry, weekday, dinner, what they've tried, and myths to clear up.

**Why play comes first:** asking personal questions before giving anything feels like a form. Once someone has busted a myth or two, answering two quick taps feels earned.

---

## How the personal note is chosen

The tips are chosen by plain rules, not an AI, so nothing unsourced can appear:

- Their **worry** picks the first tip (safety: tests first; muscle: protect muscle; regain: plan the exit; "is it for me": look beyond BMI).
- Any **myth they got wrong** adds the matching tip.
- Their **routine** adds context (a desk job adds "break up the desk day", late dinners add "plan around real life").
- It falls back to tests, muscle and exit, capped at three tips.

---

## The warm lead

On submit, the quiz sends one JSON row:

```json
{
  "ts": "2026-10-18T16:30:00Z", "src": "meta", "campaign": "",
  "seconds": 52, "hook_guess": "biscuits",
  "day": "desk", "dinner": "late", "tried": "both",
  "top_worry": "regain", "myths_believed": ["diabetes", "bmi"],
  "score": 3, "tips_shown": ["exit", "bmi", "notyou"],
  "name": "Asha Rao", "phone": "9876543210", "consent": true
}
```

**Google Sheets setup (10 minutes):**
1. Create a Sheet, then open Extensions > Apps Script and paste:
```js
   function doPost(e){
     const d = JSON.parse(e.postData.contents);
     SpreadsheetApp.getActiveSheet().appendRow([d.ts, d.name, d.phone, d.top_worry, d.day, d.dinner,
       d.tried, (d.myths_believed||[]).join(","), d.score, d.src, d.seconds]);
     return ContentService.createTextOutput("ok");
   }
```
2. Deploy as a web app with access set to "Anyone".
3. Paste the URL into `SHEET_ENDPOINT` at the top of the script in `index.html`.

Without an endpoint, leads are logged to the browser console. Users who skip the form are still logged without name or phone, so drop-off can be measured.

---

## Meta and website modes

| URL | Behaviour |
|---|---|
| `?src=meta` | Inside Meta's playable wrapper, the CTA calls `FbPlayableAd.onCTAClick()` |
| `?src=web` (default) | Full flow including the in-page form |
| `?src=share` | Set by the "Send this to someone" button |
| `?utm_campaign=...` | Stored with the lead for attribution |

---

## Target device and browser

- **Built for:** mobile, portrait, 360 to 440 px wide. Tested at 390 x 844 (iPhone 12 to 15 size).
- **Browsers:** Chrome on Android, Safari on iOS, and the Instagram and Facebook in-app browsers. It also works on desktop Chrome, Edge, Firefox and Safari, centred at phone width.
- **Inputs:** tap, swipe (pointer events), and keyboard (left and right arrows on cards, visible focus rings).
- **Accessibility:** respects `prefers-reduced-motion`, small text meets contrast on every background, tap targets are at least 44 px, and the progress dots carry aria labels.
- **Load:** one HTML file with one embedded, compressed photo. Google Fonts load on top, with system-font fallbacks
