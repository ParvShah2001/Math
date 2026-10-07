# Architecture & Technical Design

This document details the engineering decisions, state architecture, and UI/UX design patterns implemented in **MATH**.

---

## 1. Design Philosophy

**MATH** is engineered around three primary principles:

1. **Zero Cognitive Friction**: A strictly minimal black-and-white visual aesthetic that eliminates all non-essential visual elements (animations, gradients, ambient blurs) so the user focuses 100% on mental recall.
2. **Zero-Scroll Viewport Lock**: During practice drills, the entire interface is locked to the dynamic viewport height (`100dvh`). No scrolling, dragging, or accidental viewport shifts can occur while typing at high speed.
3. **Zero Runtime Dependencies**: The app is built with vanilla HTML5, CSS3, and ES6+ JavaScript. It requires no bundler, no transpiler, and no build step.

---

## 2. Application State & Screen Lifecycle

The application operates as a lightweight state machine navigating between three primary views and an overlay modal:

```
                  ┌──────────────────────┐
                  │     LOBBY SCREEN     │
                  │ (Topic Configuration)│
                  └──────────┬───────────┘
                             │ Start Session
                             ▼
                  ┌──────────────────────┐
           ┌─────►│   GAMEPLAY SCREEN    │◄────┐
           │      │   (Endless Recall)   │     │
           │      └──────────┬───────────┘     │
           │                 │ Exit / ESC      │
           │                 ▼                 │
           │      ┌──────────────────────┐     │
           │      │    DEBRIEF SCREEN    │     │
           │      │  (Metrics & Review)  │     │
           │      └──────────┬───────────┘     │
           │                 │                 │
           │     Retry       │    Menu         │
           └─────────────────┴─────────────────┘

     ┌──────────────────────────────────────────────────┐
     │             REFERENCE BOOK OVERLAY               │
     │   (Accessible from any view; scroll enabled)     │
     └──────────────────────────────────────────────────┘
```

### View Descriptions

| View | DOM Element | Behavior |
| :--- | :--- | :--- |
| **Lobby** | `#lobbyScreen` | Lets the user select target topic and configure the auto-advance toggle. |
| **Active Gameplay** | `#gameplayScreen` | Renders problem prompt, active input display, and 12-key responsive keypad. |
| **Debrief** | `#resultsScreen` | Calculates session statistics (accuracy, average pace, incorrect problem logs). |
| **Reference Book** | `#referenceModal` | Full-screen reference modal. **The only container where vertical scrolling is enabled.** |

---

## 3. Zero-Scroll Viewport Strategy

A common failure mode in mobile web-based speed trainers is accidental page bouncing or zoom when users tap keys quickly. MATH eliminates this using CSS constraints:

```css
html, body {
  height: 100%;
  height: 100dvh;
  width: 100%;
  overflow: hidden !important;
  position: fixed;
  inset: 0;
  touch-action: manipulation;
  user-select: none;
  -webkit-user-select: none;
}
```

* **`100dvh` (Dynamic Viewport Height)** accounts for dynamic browser navigation bars (Safari iOS, Chrome Android).
* **`position: fixed; inset: 0;`** anchors the root layout, eliminating iOS rubber-band bouncing.
* **`touch-action: manipulation;`** removes the default 300ms mobile browser tap delay and prevents double-tap zoom.
* **`user-select: none;`** prevents text selection highlights when repeatedly tapping keypad digits.

---

## 4. Game Engine Implementation

The core logic resides in the `GameEngine` class:

### Question Generation
* **Tables 1–10**: Generates factors $n \in [1, 10]$ and $m \in [1, 10]$.
* **Tables 11–20**: Generates factors $n \in [11, 20]$ and $m \in [1, 10]$.
* **All Tables 1–20**: Generates factors $n \in [1, 20]$ and $m \in [1, 10]$.
* **Squares**: Generates $n \in [1, 30]$, prompting for $n^2$.
* **Cubes**: Generates $n \in [1, 15]$, prompting for $n^3$.
* **Fractions**: Uniformly samples from `FRACTION_DATA`, an array of 34 high-frequency benchmark fractions with pre-indexed acceptable representations.
* **Mixed**: Randomly picks from all active categories. Immediate consecutive duplicate questions are rejected via prompt comparison.

### Tolerance & Input Validation
Fraction responses allow multiple valid notations:
* Both rounded decimals and standard integers (e.g. `33.33` or `33.3` for $1/3$, `50` for $1/2$).
* User inputs with or without the trailing `%` sign (e.g. `12.5` or `12.5%`).

### Instant Advance Engine
When `instantSubmit` is enabled:
```javascript
if (this.instantSubmit && this.currentQuestion) {
  const raw = this.currentInput.trim().replace('%', '');
  const isMatch = this.currentQuestion.acceptableAnswers.some(ans => ans.replace('%', '') === raw);
  if (isMatch) {
    setTimeout(() => { processSubmission(); }, 60);
  }
}
```
This saves one physical keypress (`Enter`) per question, speeding up drill cycles.

---

## 5. Input Subsystems

The application synchronizes two distinct input streams:

1. **Virtual Keypad**:
   - 12-key touch grid (`1`–`9`, `.`, `0`, `⌫`) plus full-width `ENTER ↵` bar.
   - Haptic vibration via `navigator.vibrate(8)` on devices supporting the Web Vibration API.
2. **Physical Keyboard Event Listener**:
   - Numeric input: `0`–`9` and `.`.
   - Actions: `Backspace` (delete), `Enter` or `Space` (submit/advance), `Escape` (exit to debrief), and `H` (reveal hint).

---

## 6. Performance Characteristics

* **Bundle Size**: Under 55 KB total.
* **External HTTP Requests**: Tailwind CSS CDN and favicon SVG data-URI.
* **Paint Times**: Zero re-renders through virtual DOM; direct DOM mutations on text nodes.
* **Memory Footprint**: Flat session array tracking response timestamps and mistakes; negligible memory consumption during long sessions.
