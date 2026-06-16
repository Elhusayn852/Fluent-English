# Fluent English — Interactive Study Website

## Project Overview

An interactive single-page web application that transforms 18 markdown-based English lesson files into a dynamic, self-paced study experience. Learners navigate vocabulary flashcards, grammar explanations with interactive exercises, phrasal verb references, review quizzes with answer keys, and cultural notes — all rendered from structured JavaScript data without any backend, database, build tools, or external frameworks.

The application reads content from markdown files in `/MD`, parses them into JS objects, and presents the material through tabbed lesson views designed for active recall and engagement rather than passive reading.

---

## Learning Goals & Intended Study Experience

| Goal | How it's addressed |
|---|---|
| Vocabulary retention | Flip-card system — learner sees the term, guesses the definition, then reveals. Difficulty tagging (easy/medium/hard) persists across sessions. |
| Grammar comprehension | Structured explanations followed by fill-in-the-blank practice with instant feedback. Accordion layout lets learners control their pace. |
| Phrasal verb familiarity | Searchable card grid — learner can browse or filter. Included in Review Mode for spaced recall. |
| Exercise/quiz mastery | Interactive text inputs for all practice and review exercises. Check button validates against answer key; Reveal shows correct answer. |
| Cultural awareness | Dedicated "Culture" tab per lesson — reading content with markdown formatting. |
| Active recall | Review Mode pulls random items from all lessons (vocabulary + phrasal verbs), hides the answer, and lets the learner quiz themselves. |
| Progress ownership | Lesson completion checkmarks, vocabulary difficulty tagging, and progress counters saved to `localStorage`. |

---

## Folder Structure

```
Fluent English/
├── index.html          # Single HTML shell — sidebar + main content area
├── styles.css           # All styling (no inline styles beyond minimal dynamic)
├── script.js            # Lesson data + rendering logic + interactivity
├── PROJECT.md           # This document
├── MD/
│   ├── Lesson-01.md
│   ├── Lesson-02.md
│   ├── Lesson-04.md
│   ├── Lesson-05.md
│   ├── Lesson-07.md
│   ├── Lesson-08.md
│   ├── Lesson-09.md
│   ├── Lesson-10.md
│   ├── Lesson-11.md
│   ├── Lesson-12.md
│   ├── Lesson-13.md
│   ├── Lesson-14.md
│   ├── Lesson-15.md
│   ├── Lesson-16.md
│   ├── Lesson-17.md
│   ├── Lesson-18.md
│   ├── Lesson-19.md
│   └── Lesson-20.md
└── PDFs (ignored)/     # Not used by the application
```

Note: `Lesson-03.md` and `Lesson-06.md` exist in `/MD` but are duplicates of `Lesson-02` and `Lesson-05` respectively. They are excluded from the website data.

---

## Technical Constraints

- **Three files only:** `index.html`, `styles.css`, `script.js` — no other files are loaded at runtime.
- **No frameworks:** Vanilla JS only. No React, Vue, Svelte, jQuery, etc.
- **No build tools:** No Webpack, Vite, Parcel, Babel, etc. The site is opened directly from the file system.
- **No backend:** No server, no database, no API calls. All data lives in `script.js`.
- **No external dependencies:** Everything is self-contained in the three files.
- **Browser storage only:** `localStorage` for persistence (lesson completion, vocabulary difficulty).
- **No module system:** The script uses an IIFE (Immediately Invoked Function Expression) to encapsulate scope and exposes a `window.FE` API object for inline `onclick` handlers.
- **File protocol support:** Must work when opened via `file://` (no CORS, no service workers).

---

## Data Architecture

### Lesson Object Structure

Every lesson in the `LESSONS` array follows this schema:

```js
{
  id: Number,             // 1, 2, 4, 5, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20
  title: String,          // Lesson title from H1
  intro: String,          // "ARE YOU READY FOR THE LESSON?" content (plain text)

  reading: String,        // "ENGLISH AT WORK" content — dialogue or reading passage
                          // Contains markdown formatting: **Speaker:**, *emphasis*

  vocabulary: [           // "BUILD YOUR VOCABULARY" items
    {
      term: String,       // The vocabulary term (bold text in source)
      definition: String, // Definition text
      example: String     // Example sentence (may be empty)
    }
  ],

  grammar: [              // "ENGLISH UNDER THE HOOD" topics
    {
      topic: String,                   // Topic title (e.g. "The Present Perfect Tense vs. the Simple Past Tense")
      explanation: String,             // Explanation text with markdown
      practice: {                      // Associated practice exercise (may be null)
        title: String,                 // Exercise title (e.g. "PRACTICE EXERCISE 1")
        instruction: String,           // Instruction text (may be empty)
        items: [                       // Array of practice items
          { sentence: String }         // The sentence with blank
        ]
      }
    }
  ],

  phrasalVerbs: [         // "PHRASAL VERBS" section
    {
      verb: String,       // The phrasal verb (e.g. "pick up")
      meanings: [         // Array of meaning objects
        { text: String }  // Meaning text with sub-meanings inline
      }
    }
  ],

  realEnglish: String,    // "REAL ENGLISH" content (markdown text)

  reviewExercises: [      // "BRING IT ALL TOGETHER" exercises
    {
      title: String,      // Exercise title (e.g. "REVIEW EXERCISE 1: Vocabulary")
      items: [            // Array of review items
        { question: String }
      ]
    }
  ],

  listening: [String],    // "LISTEN UP!" questions (may be empty)

  cultureNote: String,    // "WHY DO THEY DO THAT?" content (markdown text)

  answerKey: {            // "ANSWER KEY" section
    "Practice Exercise 1": String,   // Keys are section titles, values are raw text
    "Review Exercise 1": String      // Line-broken text for display
  }
}
```

### Answer Key Matching Strategy

Answer keys are stored as flat string values keyed by section title. When a user checks an answer:

1. **Grammar practice:** The exercise title (e.g. `"PRACTICE EXERCISE 1: Complete each..."`) is lowercased and cleaned. The answer key keys are similarly cleaned. The function finds the answer key whose title shares at least 2 significant words with the practice exercise title. A fallback uses sequential indexing (first practice → first "Practice" answer key).

2. **Review exercises:** The function extracts the exercise number from both the review exercise title and each answer key title (e.g. `"review exercise 1"` or `"review 1"`). It matches by number. A fallback uses sequential indexing.

3. **Answer parsing within a key:** The raw answer text is split into lines. Each line is checked against `^(\d+)[\.)]\s*(.+)` to extract the numbered answer. Multiple strategies are tried (comma-separated, space-separated, dot-separated) to handle variations in answer key formatting.

---

## UI Architecture

The interface must follow Google Material 3 (Material You) design principles and UX guidance precisely. Do not create a generic card website — every component, color, shape, elevation, and interaction must be deliberate Material 3.

### Visual System

#### Material 3 Color System

Use the full Material 3 color role mapping. All colors derive from a single primary seed color via the Material 3 tonal palette. Both light and dark themes must be supported.

| Light Theme Role | CSS Custom Property | Description |
|---|---|---|
| Primary | `--md-sys-color-primary` | Key brand color for active states, filled buttons, FABs |
| On Primary | `--md-sys-color-on-primary` | Content on primary surfaces (text/icons) |
| Primary Container | `--md-sys-color-primary-container` | Tonal surface for selected items, cards |
| On Primary Container | `--md-sys-color-on-primary-container` | Content on primary container |
| Secondary | `--md-sys-color-secondary` | Accent color for secondary controls, chips |
| On Secondary | `--md-sys-color-on-secondary` | Content on secondary surfaces |
| Secondary Container | `--md-sys-color-secondary-container` | Tonal surface for secondary selections |
| On Secondary Container | `--md-sys-color-on-secondary-container` | Content on secondary container |
| Tertiary | `--md-sys-color-tertiary` | Contrast accent for vocabulary difficulty, badges |
| On Tertiary | `--md-sys-color-on-tertiary` | Content on tertiary surfaces |
| Tertiary Container | `--md-sys-color-tertiary-container` | Tonal surface for tertiary selections |
| On Tertiary Container | `--md-sys-color-on-tertiary-container` | Content on tertiary container |
| Error | `--md-sys-color-error` | Incorrect answers, error states |
| On Error | `--md-sys-color-on-error` | Content on error surfaces |
| Error Container | `--md-sys-color-error-container` | Tonal surface for error states |
| On Error Container | `--md-sys-color-on-error-container` | Content on error container |
| Surface | `--md-sys-color-surface` | Card backgrounds, sheet surfaces |
| On Surface | `--md-sys-color-on-surface` | Primary text on surface |
| Surface Variant | `--md-sys-color-surface-variant` | Subtle surface distinction |
| On Surface Variant | `--md-sys-color-on-surface-variant` | Secondary text on surface |
| Background | `--md-sys-color-background` | Page background |
| On Background | `--md-sys-color-on-background` | Primary text on background |
| Outline | `--md-sys-color-outline` | Borders, dividers, unfilled input outlines |
| Outline Variant | `--md-sys-color-outline-variant` | Subtle borders |
| Shadow | `--md-sys-color-shadow` | Drop shadow color |
| Surface Tint | `--md-sys-color-surface-tint` | Elevation tint overlay color |

Dark theme roles follow the same mapping with inverted luminance. All color tokens must be defined as CSS custom properties on `:root` (light) and a `[data-theme="dark"]` selector.

#### Material 3 Typography Scale

Use the complete Material 3 type scale. Every text element maps to a specific type style. Use system fonts (`system-ui`, `Roboto`, `Noto Sans`) to avoid external dependencies.

| Type Style | Size | Weight | Line Height | Usage |
|---|---|---|---|---|
| Display Large | 57px | 400 | 64px | Hero welcome screen heading |
| Display Medium | 45px | 400 | 52px | Welcome subtitle |
| Display Small | 36px | 400 | 44px | Lesson title hero |
| Headline Large | 32px | 400 | 40px | Section headers (Reading, Vocabulary, etc.) |
| Headline Medium | 28px | 400 | 36px | Tab content section head |
| Headline Small | 24px | 400 | 32px | Grammar topic titles |
| Title Large | 22px | 400 | 28px | Sidebar brand heading |
| Title Medium | 16px | 500 | 24px | Sidebar lesson items, card titles |
| Title Small | 14px | 500 | 20px | Subheaders, chip labels |
| Body Large | 16px | 400 | 24px | Reading passages, explanations |
| Body Medium | 14px | 400 | 20px | Vocabulary definitions, exercise text |
| Body Small | 12px | 400 | 16px | Captions, timestamps, hints |
| Label Large | 14px | 500 | 20px | Button labels, tab labels |
| Label Medium | 12px | 500 | 16px | Badge text, small button labels |
| Label Small | 11px | 500 | 16px | Tiny meta text |

#### Material 3 Shape System

| Shape Token | Corner Radius | Usage |
|---|---|---|
| `--md-sys-shape-corner-none` | 0px | Full-bleed elements, dividers |
| `--md-sys-shape-corner-extra-small` | 4px | Input fields, small badges, chips |
| `--md-sys-shape-corner-extra-small-top` | 4px top only | Top app bar, sheets |
| `--md-sys-shape-corner-small` | 8px | Buttons, cards, accordion headers |
| `--md-sys-shape-corner-medium` | 12px | Dialogs, side panels, navigation drawer |
| `--md-sys-shape-corner-large` | 16px | Bottom sheets, FAB, search bar |
| `--md-sys-shape-corner-extra-large` | 28px | Full-screen dialogs, extended FAB |

#### Material 3 Elevation System

| Elevation Level | Shadow | Usage |
|---|---|---|
| Level 0 | None | Flat surfaces (background) |
| Level 1 | Small shadow | Cards, navigation drawer |
| Level 2 | Medium shadow | FAB, search bar, top app bar |
| Level 3 | Pronounced shadow | Elevated cards, dialogs |
| Level 4 | Large shadow | Modal bottom sheets, snackbar |
| Level 5 | Largest shadow | Review overlay, floating elements |

Elevations are applied via CSS custom properties `--md-sys-elevation-level{N}` using layered box-shadows and a `surface-tint` color overlay for the tonal elevation effect (Material 3 subtle tinted surface at higher elevations).

#### Material 3 Spacing System

Spacing follows an 8dp grid. All margins, padding, and gaps use values from the set: 4, 8, 12, 16, 20, 24, 28, 32, 40, 48, 56, 64 (pixels/rem equivalents). Gutter widths, content padding, and component spacing must align to this grid.

### Layout

```
┌──────────────────────────────────────────────────────────┐
│  Navigation Drawer (360px max) │  Main Content           │
│  ┌─────────────────────────┐   │  ┌──────────────────┐  │
│  │ Brand: "Fluent English" │   │  │ Top App Bar      │  │
│  │ with icon               │   │  │ [menu][title]    │  │
│  │                         │   │  ├──────────────────┤  │
│  │ Search bar (input chip) │   │  │ Lesson Title     │  │
│  │                         │   │  │ (Headline Small) │  │
│  │ Scrollable lesson list  │   │  │                  │  │
│  │  [✓] Lesson 1 (body)   │   │  │ Tabs (secondary  │  │
│  │  [✓] Lesson 2          │   │  │  container)      │  │
│  │  [ ] Lesson 4          │   │  ├──────────────────┤  │
│  │  [ ] Lesson 5          │   │  │                  │  │
│  │  ...                    │   │  │ Tab Content      │  │
│  ├─────────────────────────┤   │  │ (dynamic)        │  │
│  │ FAB or footer button    │   │  │                  │  │
│  └─────────────────────────┘   │  │                  │  │
│                                │  │                  │  │
│                                │  │ [FAB]            │  │
│                                │  └──────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

- **Navigation drawer (sidebar):** Modal or permanent navigation drawer following Material 3 guidance. Contains brand header with icon, an always-visible search bar (using the search bar component pattern), a scrollable list of lesson items with single-line list item layout, and a fixed bottom section with an elevated button (tonal or filled) and progress text.
- **Top app bar:** Appears at the top of the main content area. Contains a hamburger menu icon (for drawer toggle on narrow screens), the lesson title (or "Fluent English" on welcome), and contextual actions (theme toggle, mark complete).
- **Main content:** Scrollable area below the top app bar, contains lesson hero, tab row (scrollable, with active indicator), and dynamic tab content.
- **Mobile (< 840px):** Navigation drawer becomes modal overlay (behind backdrop). Top app bar menu button toggles it. Main content is full-width.

### Theming

- Light theme and dark theme must both be fully supported.
- Theme is toggled via a button in the top app bar (light/dark mode icon toggle).
- Theme state is persisted in `localStorage` under key `feTheme`.
- All component colors use the Material 3 color role tokens — no hardcoded hex values outside the seed color definition.
- Theme transitions should use `background-color` and `color` transitions of 200ms ease for smooth switching.

---

## User Flow

1. **Landing:** User sees the welcome screen with sidebar showing all 18 lessons. No lesson is open.

2. **Select a lesson:** User clicks a lesson in the sidebar. The site renders:
   - Lesson title and intro text
   - Tab bar with 7 tabs: Reading, Vocabulary, Grammar, Phrasal Verbs, Real English, Exercises, Culture
   - "Mark Complete" / "Unmark Complete" toggle button
   - Reading tab content by default

3. **Navigate tabs:** User clicks any tab — the tab button highlights blue, the content area updates with the appropriate section.

4. **Study vocabulary:** User sees a grid of flip-cards. Each card shows the term on the front. Clicking flips it to reveal the definition and example. Difficulty buttons (Easy/Medium/Hard) appear on the back. Difficulty state is saved to `localStorage` and restored on revisit.

5. **Study grammar:** User sees accordion sections per grammar topic. Each is open by default. Clicking the header toggles expand/collapse. Each topic has an explanation section and a practice exercise with text inputs.

6. **Do exercises:** User types answers into input fields. Pressing Enter or clicking "Check All" validates against the answer key. Correct answers are highlighted green, incorrect red. "Reveal Answers" fills in the correct answer.

7. **Browse phrasal verbs:** User sees a card grid. Typing in the search input filters cards in real-time.

8. **View answer key:** In the Exercises tab, a collapsible "View Answer Key" section at the bottom shows all answers for the lesson.

9. **Mark complete:** User clicks "Mark Complete" — the sidebar updates with a checkmark and the progress counter increments. State persists in `localStorage`.

10. **Review Mode:** User clicks "Start Review Session" — a modal overlay appears with random vocabulary and phrasal verb items from all lessons. The user sees a question, clicks "Show Answer" to reveal, and clicks "Next Question" to advance. Progress is tracked within the session.

11. **Search:** User types in the sidebar search input — the lesson list filters in real-time by title and intro text.

---

## Component Descriptions

Every component must follow Material 3 behavior expectations. Refer to the Material 3 component guidance for motion, states, and accessibility.

### 1. Navigation Drawer (`#sidebar`)

- Follows the Material 3 navigation drawer component pattern. On screens ≥840px it is a permanent drawer (side sheet). On screens <840px it is a modal drawer overlaying the content with a scrim backdrop.
- **Brand header:** Contains an icon (or text mark) and "Fluent English" title using Title Large type style.
- **Search bar:** A Material 3 search bar component inside the drawer header — an elevated rounded input with a leading search icon. Filtering lesson items uses the search bar's activated state pattern.
- **Lesson list (`.list-item`):** Each lesson is a single-line Material 3 list item with leading avatar (numbered badge) and supporting text (lesson title). Selected item uses primary container color. Completed items show a leading check icon in primary or tertiary color.
- **Footer:** Contains a Material 3 filled tonal button ("Start Review Session") and a caption with progress text (Label Medium).
- **States:** Loading (skeleton or progress indicator), populated, search-active (filtered list with visible count), no-results (empty state with supporting text).

### 2. Top App Bar (`#topAppBar`)

- Material 3 top app bar (small variant). Fixed at the top of the main content area.
- **Leading:** Hamburger menu icon button (toggles navigation drawer on mobile).
- **Title:** Current lesson title (Title Large) or "Fluent English" on the welcome screen.
- **Trailing actions:** Icon buttons for: theme toggle (light/dark), mark complete toggle (check icon with filled/unfilled state).
- The top app bar surface uses `surface` color at elevation level 2.

### 3. Welcome Screen (`#welcome`)

- Centered hero layout using Display Small or Display Medium typography.
- Subtle vertical rhythm with 24–48px spacing following the 8dp grid.
- An optional progress chip or badge showing overall course progress at the top.
- Replaced by lesson content when a lesson is selected. Use a fade transition (150ms ease).

### 4. Lesson Content (`#lessonContent`)

- **Hero title:** Headline Small or Display Small for the lesson title.
- **Intro text:** Body Large supporting text with comfortable line height (1.5).
- **Tab row:** Material 3 secondary tabs component. A horizontally scrollable row of tab items. The active tab has an underline indicator and uses `primary` color. Inactive tabs use `on-surface` at reduced opacity. Tabs should be full-width on larger screens and scrollable on narrow screens.
- **Mark Complete button:** A Material 3 icon button (checkmark) in the top app bar, or a filled tonal button in the lesson content area.

### 5. Tab Content (`#tabContent`)

Dynamically swapped based on active tab. Tab transitions use a subtle fade (100–150ms).

| Tab | Material 3 Component | Description |
|---|---|---|
| Reading | Card + typography | Reading passage displayed in an elevated card (level 1). Speakers use bold or primary color. Inline vocabulary highlights use tooltip-on-click or chip-style popups. |
| Vocabulary | Card grid + FAB | Grid of elevated flip-cards (level 1). A FAB or icon button for shuffle and reset. Cards use `surface` color with `outline` border. |
| Grammar | Accordion (expansion panel) | Expansion panels following Material 3 patterns. Each panel is an elevated card (level 1) with a header row (clickable, with expand/collapse chevron) and a body that slides open/closed with a 200ms ease animation. |
| Phrasal Verbs | Card grid + search | Filterable grid of elevated cards (level 1). Search uses a Material 3 search bar or an outlined text field with leading icon. |
| Real English | Card with tinted surface | Elevated card (level 1) with `tertiary-container` surface color for visual distinction. |
| Exercises | Card + text fields + buttons | Review exercises use Material 3 outlined text fields (with error state styling) and filled/tonal buttons for check/reveal. Answer key is a Material 3 expansion panel at the bottom. |
| Culture | Card with tinted surface | Elevated card (level 1) with `secondary-container` or `primary-container` surface color. |

### 6. Vocabulary Flip-Card (`.vocab-card`)

- Follows the Material 3 elevated card component pattern with added flip interaction.
- **Front:** Term displayed centered using Title Medium type. "Tap to reveal" supporting text (Body Small) at the bottom.
- **Back:** Definition (Body Medium), optional example sentence (Body Small italic), and difficulty chips (three Material 3 input chips: Easy/Medium/Hard — using `tertiary`, `secondary`, and `error` color mappings).
- **Interaction:** Tap to flip via CSS 3D transform (`rotateY(180deg)`) with 300–400ms ease animation. Difficulty chips have `stopPropagation` to prevent card flip. Focus states on cards for keyboard users (Enter/Space to flip).
- **State persistence:** Flipped state resets on tab switch. Difficulty persists per `lessonId-index` in `localStorage`.
- **Grid layout:** Responsive grid with card columns adapting from 4 (wide) to 2 (tablet) to 1 (mobile). Cards have a fixed aspect ratio of 3:2 or 4:3.

### 7. Grammar Accordion (Expansion Panel)

- Material 3 expansion panel pattern. Each panel is an elevated card (level 1) with `surface` color.
- **Header:** Topic title (Title Small) with an expand/collapse icon (chevron) on the right. Hover state uses `surface-variant` at low opacity. Ripple effect on tap/click.
- **Body:** Explanation text (Body Large), followed by optional practice exercise with Material 3 outlined text fields and a filled button ("Check All").
- **Animation:** Body expands/collapses with a 200–250ms ease-in-out height transition.
- **Default state:** All panels expanded (open).

### 8. Practice / Review Exercise Input

- Material 3 outlined text field (`md-outlined-text-field` pattern). Leading label animates up on focus.
- **Feedback states:** Input uses `error` color for incorrect, `primary` or `tertiary` for correct. Supporting text below the input shows feedback (Body Small).
- **Keyboard support:** Enter key checks answer. Tab navigates between inputs.
- **Bulk actions:** A Material 3 filled button ("Check All") and a tonal button ("Reveal Answers") at the top of each exercise section.

### 9. Dialogs and Overlays

- **Review dialog (`#reviewOverlay`):** Material 3 full-screen dialog pattern on mobile, or an alert dialog on larger screens. Has a top app bar with title ("Review Session"), close icon button, and a progress indicator (linear or segmented). Content area shows one question at a time with a "Show Answer" button (outlined or text button) and "Next" navigation (filled button).
- **Answer key panel:** Expansion panel or bottom sheet pattern at the bottom of the Exercises tab. Collapsed by default with a "View Answer Key" label.
- **Scrim:** Semi-transparent backdrop (at 32% opacity) for modal drawer and dialogs. Clicking scrim dismisses the overlay.

### 10. Chips

- Material 3 input chips for vocabulary difficulty (Easy / Medium / Hard).
- Filter chips for tab-like filtering within the Vocabulary tab (e.g., "All", "Easy", "Medium", "Hard").
- Assist chips for hint buttons, "Show Answer", etc.
- Chip colors use the Material 3 color roles: `secondary-container` for unselected, `primary-container` for selected.

### 11. Buttons

| Button Type | Material 3 Variant | Usage |
|---|---|---|
| Primary action | Filled button (`md-filled-button`) | "Start Review Session", "Check All", "Save", "Next Question" |
| Secondary action | Filled tonal button (`md-filled-tonal-button`) | "Reveal Answers", "Mark Complete", "Show Answer" |
| Subtle action | Text button (`md-text-button`) | "Cancel", "Close", "Reset filters" |
| Tertiary action | Outlined button (`md-outlined-button`) | Secondary actions in dialogs |
| Icon-only | Icon button (`md-icon-button`) | Theme toggle, menu, mark complete, close |

### 12. Floating Action Button (FAB)

- A Material 3 FAB (small or medium variant) appears in the Vocabulary tab for shuffle/reset actions.
- Positioned bottom-right within the tab content, not overlapping the navigation drawer.
- Uses `primary-container` / `on-primary-container` colors.

### 13. Progress Indicators

- **Linear progress indicator:** Used for lesson loading, review session progress, and overall course progress bar.
- **Circular progress indicator:** Used for brief loading states (e.g., switching lessons).
- Colors follow Material 3: `primary` track with `primary-container` background.

### 14. Tooltips

- Vocabulary terms in the reading passage use Material 3 tooltip pattern — small overlay on hover or focus showing the definition.
- Tooltips appear after a 200ms delay and disappear after 100ms of cursor leaving.
- Use `surface` level 3 elevation with `on-surface` text color.

### 15. Snackbar

- A Material 3 snackbar for transient feedback messages: "Lesson marked complete", "Progress saved", etc.
- Appears at the bottom of the viewport. Displays a message and an optional action button ("Undo" for completion toggling, "Dismiss").
- Auto-dismisses after 4 seconds. Respects user motion preferences.

---

## Interactive Features & Behaviors

| Feature | Implementation |
|---|---|
| Lesson switching | Click on sidebar item → `openLesson(id)` → renders entire lesson view |
| Tab switching | Click tab button → `switchTab(tabId)` → updates button classes + renders tab content |
| Card flip | `flipVocab(el)` → toggles `.flipped` class → CSS 3D transform |
| Card shuffle | Randomizes DOM order of `.vocab-card` elements in the grid |
| Difficulty marking | `markVocab(idx, diff)` → saves to `vocabState` object → persists to `localStorage` |
| Answer checking | `checkGP`/`checkRV` → looks up answer via `findAns()` → compares (case-insensitive, trimmed, slash-stripped) → sets input class + feedback text |
| Bulk check | `checkAllGP`/`checkAllRV` → iterates all inputs in a section → calls individual check |
| Bulk reveal | `revealAllGP`/`revealAllRV` → fills all inputs with answer → marks as revealed |
| Accordion toggle | `toggleGram(el)` → toggles `.open` class → CSS `display: block/none` |
| Phrasal verb search | `filterPV(val)` → filters `.pv-card` by `data-pvt` attribute (lowercased verb) |
| Lesson search | `searchLessons(q)` → filters `.lesson-item` elements by title and intro text |
| Complete toggle | `toggleComplete(id)` → toggles ID in array → persists → rerenders sidebar + updates button text |
| Review mode | `startReview()` → creates overlay → collects all vocab + phrasal verbs → shuffles → renders one at a time |
| Progress persistence | `saveProgress()` → `JSON.stringify` to `localStorage` keys `feCompleted` and `feVocab` |
| Escape key | `document.addEventListener('keydown', ...)` → calls `closeReview()` |

---

## Markdown Parsing Strategy

The source markdown lessons follow a consistent structure:

```
# Title

## ARE YOU READY FOR THE LESSON?
Intro text...

## SAY IT CLEARLY!
Pronunciation content (not stored — audio reference)

## ENGLISH AT WORK
### Dialogue/Reading Title (optional)
Content with **Speaker:** labels and *emphasis*

## BUILD YOUR VOCABULARY
**Term.** Definition. *Example sentence.*

## ENGLISH UNDER THE HOOD
### TOPIC 1: Title
Explanation text...
**PRACTICE EXERCISE 1:** Instruction
1. Sentence...
2. Sentence...
### TOPIC 2: Title
...

## PHRASAL VERBS
**Verb.** Meaning text.

## REAL ENGLISH
Content with *idioms* and descriptive text.

## BRING IT ALL TOGETHER
### REVIEW EXERCISE 1: Title
Items...
### REVIEW EXERCISE 2: Title
...

## LISTEN UP!
Questions...

## WHY DO THEY DO THAT?
Culture note content.

## Lesson N: Answer Key
**Practice Exercise 1**
1. answer
2. answer
...
```

### Parsing Approach (Node.js build script)

A Node.js script (`build-data.js`) performs the extraction:

1. **Section detection:** Lines starting with `## ` are matched against section header patterns. A non-capturing group `(?:\d+[A-Z]?\s)?` handles optional lesson-number prefixes (e.g. `## 1 BUILD YOUR VOCABULARY`, `## 1E PHRASAL VERBS WITH PICK`).

2. **Vocabulary:** Lines matching `^\*\*(.+?)\*\*\.?\s*(.*)` capture the bold term and the remaining text. Embedded `*...*` is extracted as the example sentence.

3. **Grammar:** Topic headers `^### TOPIC \d+:` create new topic objects. `PRACTICE EXERCISE` headers (both `###` and `**` bold variants) create practice objects. Lines beginning with `\d+[\.)]` become practice items.

4. **Phrasal verbs:** Lines matching `^\*\*(.+?)\*\*` start a new verb entry. Subsequent text lines are added as meanings.

5. **Review exercises:** `^### REVIEW` headers create new exercise objects. Numbered lines become items.

6. **Answer keys:** `**Section Title**` lines start a new answer section. Subsequent lines are accumulated as the answer value.

7. **Reading/Real English/Culture:** Raw text is accumulated with minimal processing.

8. **Deduplication:** Lessons 3 and 6 are excluded (they are duplicates of 2 and 5).

### Client-Side Markdown Rendering

In the browser, a simple `md()` function handles inline formatting:

1. `&`, `<`, `>`, `"` are HTML-escaped first.
2. `***text***` → `<strong><em>text</em></strong>`
3. `**text**` → `<strong>text</strong>`
4. `*text*` → `<em>text</em>`
5. `\n` → `<br>`

This covers the subset of markdown used in the lesson content (bold, italic, combined bold+italic, and line breaks).

---

## Lesson Structure Assumptions

1. Every lesson file starts with a single `# Title` on line 1.
2. Section headers are `##` level, subsection headers are `###`.
3. The sequence of sections is consistent across all lessons (with optional `SAY IT CLEARLY!` and `LISTEN UP!` sections that are parsed but not displayed).
4. Vocabulary terms follow the pattern `**Term.** Definition text. *Example sentence.*` — the period after `**Term.**` is sometimes missing.
5. Grammar topics are numbered `TOPIC 1`, `TOPIC 2`, `TOPIC 3`.
6. Practice exercises use either `### PRACTICE EXERCISE N:` or `**PRACTICE EXERCISE N:**` formatting.
7. Phrasal verb entries start with `**Verb.**` followed by meaning lines.
8. Review exercises use `### REVIEW EXERCISE N:` or `### REVIEW N:` formatting.
9. The answer key section is the final `##` section with `**Section Title**` subsections.
10. Item numbering within exercises uses `1.`, `2.`, etc.
11. Lessons 3 and 6 are duplicates of lessons 2 and 5 respectively and should be excluded.

---

## Rendering Strategy

### Static Data + Dynamic DOM

All lesson data is pre-parsed into a `LESSONS` array embedded in `script.js`. No runtime parsing of markdown files occurs in the browser.

### Rendering Flow

```
User clicks lesson → openLesson(id)
  → set currentLesson, currentTab = "reading"
  → hide #welcome, show #lessonContent
  → build tab bar HTML with active state
  → call renderTab("reading")

User clicks tab → switchTab(tabId)
  → update currentTab
  → update tab button classes
  → call renderTab(tabId)

renderTab(tabId)
  → build section-specific HTML string
  → set tc.innerHTML = html
  → [vocabulary only] restore difficulty state via setTimeout
```

### Key Design Decisions

- **HTML strings over DOM manipulation:** Tab content is built as a single HTML string and set via `innerHTML`. This is faster and simpler for the volume of content being rendered.
- **No virtual DOM:** With max ~100 elements per tab, direct DOM updates are performant enough.
- **Event delegation:** Sidebar clicks use a single `document` click listener with `closest(".lesson-item")` — no per-item event binding.
- **Inline event handlers:** Tab buttons, quiz inputs, and action buttons use `onclick` / `onkeydown` attributes referencing `window.FE.*` methods. This avoids the complexity of dynamic event binding for elements that are recreated on every render.

---

## Progress Tracking Strategy

### What's Saved

| Data | localStorage Key | Format | Updated |
|---|---|---|---|
| Completed lesson IDs | `feCompleted` | `[1, 2, 5, ...]` (JSON array) | When user clicks Mark/Unmark Complete |
| Vocabulary difficulty | `feVocab` | `{"1-0":"easy","1-1":"hard",...}` (JSON object) | When user clicks Easy/Medium/Hard on a card |

### When It's Loaded

- On page load, both keys are read from `localStorage` (with `|| "[]"` / `|| "{}"` fallbacks).
- On vocabulary tab render, difficulty states are restored from `vocabState` by matching `lessonId-index` keys.

### Granularity

- Lesson completion is all-or-nothing per lesson.
- Vocabulary difficulty is per-card per-lesson.
- Tab position, scroll position, and flipped state are not persisted (intentional — each session starts fresh).

---

## Future Scalability Considerations

1. **Additional lessons:** New markdown files added to `/MD` would need to be parsed by the build script and embedded into `script.js`. The rendering code would handle them automatically since it iterates the `LESSONS` array.

2. **Answer matching improvements:** The current heuristic-based matching could be replaced with exact title matching if the build script normalizes practice exercise and answer key titles to a shared identifier.

3. **Spaced repetition:** The difficulty tagging infrastructure (`vocabState`) is a foundation for a full SRS system. A future version could track review dates and prioritize hard items.

4. **Search across content:** The sidebar search could be extended to search within lesson content (vocabulary terms, grammar topics) and show contextual results.

5. **Data extraction as a build step:** The current approach of pre-embedding all data can be replaced with a proper build pipeline that runs the parser on every file change and regenerates `script.js`.

6. **Audio integration:** The `SAY IT CLEARLY!` and `LISTEN UP!` sections reference audio CD tracks. A future version could embed audio files and add playback controls to those sections.

7. **Theming expansion:** The Material 3 CSS custom properties architecture supports additional color schemes beyond light/dark — alternate seed colors or dynamic color extraction (Android-style Material You) could be added via the same token mapping.

8. **Offline PWA:** Adding a service worker and manifest could make the site fully offline-capable since all data is already embedded.

---

## Design Principles

All UX decisions follow Material 3 guidance. The interface must not invent a custom aesthetic.

### General UX Requirements

1. **Progressive disclosure:** Show only what is needed at each step. Grammar explanations start collapsed (user opens what they need). Answer keys are hidden behind an expansion panel. Review mode shows one question at a time. Do not overwhelm the learner with all content at once.

2. **Minimize cognitive load:** Use the Material 3 type scale consistently so learners subconsciously recognize element hierarchy. Group related content into elevated card surfaces. Use consistent iconography from the Material Symbols set. Avoid decorative flourishes — every visual element serves a purpose.

3. **Prioritize readability and studying flow:** Body text uses Body Large (16px) with adequate line height (1.5–1.6). Maximum line width for reading passages is 640px (40rem). Use the 8dp grid for consistent vertical rhythm. Headings are visually distinct through size/weight, not color alone.

4. **Clear visual hierarchy:** The Material 3 color roles create hierarchy without relying on font size alone. Primary surfaces indicate interactive/active elements. Surface colors indicate content containers. Outline colors indicate boundaries. Elevation indicates importance.

5. **Predictable interactions:** All clickable elements have Material 3 state layers (hover state, focus state, active/pressed state, and ripple effect). Buttons look like buttons (filled, tonal, outlined, or text — never plain text pretending to be a button). Cards are not clickable unless they are designed as Material 3 cards with interactive behavior.

6. **Avoid long uninterrupted text blocks:** Break reading passages into short paragraphs (3–5 lines max). Use cards and expansion panels to separate topics. Use tabs to divide lesson content into manageable sections. Use lists and tables where appropriate instead of prose.

7. **Reduce unnecessary scrolling:** Navigation drawer keeps lessons always accessible. Tabs allow quick switching between content sections. Vocabulary cards use a responsive grid. The top app bar provides context without taking up vertical space.

8. **Focus states and feedback:** Every interactive element has a visible focus ring using `outline` or `box-shadow` with primary color. Ripple effect on touch/click. The Material 3 state layer system (hover at 8% primary, focus at 10% primary, press at 12% primary) must be implemented via CSS pseudo-classes or JS.

9. **Accessibility and keyboard navigation:** All interactive elements are reachable via Tab. Enter/Space activates buttons and toggles. Escape closes dialogs and overlays. Arrow keys navigate tabs and list items. The navigation drawer can be opened/closed via keyboard. Color contrast meets WCAG AA (4.5:1 for text, 3:1 for large text and UI components).

10. **Consistent interaction patterns:** Same type of action uses the same component everywhere. All answer-checking uses the same feedback pattern. All expansion panels animate the same way. All cards use the same corner radius (small/12px). Never mix patterns for the same purpose.

### Study-Specific UX Requirements

11. **Reading mode — focus and distraction reduction:** Reading tab uses a clean elevated card with maximum 640px line width. Inline vocabulary highlights show definitions via tooltip (hover/click), not inline popups that break reading flow. Background is `surface` with ample padding. No distracting animations in the reading view.

12. **Vocabulary — study cards, not database entries:** Vocabulary items are always presented as flip-cards, never as a list or table. The card metaphor makes each term feel tangible and memorable. Difficulty chips (Easy/Medium/Hard) on the back allow self-assessment. Shuffle and reset controls are accessible but unobtrusive. Cards animate with a smooth 3D flip to reinforce the "study card" mental model.

13. **Exercises — immediate feedback:** Every text input checks the answer on Enter or on blur. Correct answers show primary color styling with a brief check animation. Incorrect answers show error color with the correct answer revealed. "Check All" and "Reveal Answers" buttons provide bulk actions. Feedback text is displayed as supporting text below the input (Body Small).

14. **Review mode — guided practice:** The review dialog presents one item at a time with a clear progress indicator (linear or step counter). The "Show Answer" button encourages self-testing before revealing. "Next Question" advances sequentially. Review items are drawn randomly from all completed lessons. The user feels guided, not tested.

15. **Progress indicators — encourage continuation:** The sidebar shows a clear completion count as a chip or label. The review session shows linear progress. Lesson completion is indicated by a check icon with a subtle animation. Progress is always positive and encouraging — never shaming or punitive.

### Material Motion

- All animations use Material 3 motion guidance: 150–200ms for small interactions (button presses, toggles), 200–300ms for medium transitions (tabs, expansion panels), 300–400ms for large transitions (card flips, dialog open/close).
- Easing follows Material 3: `cubic-bezier(0.2, 0, 0, 1)` for standard easing, `cubic-bezier(0.4, 0, 0.2, 1)` for deceleration, `cubic-bezier(0, 0, 0.2, 1)` for acceleration.
- Respect `prefers-reduced-motion` — disable non-essential animations when the user requests reduced motion.
- Animations are subtle and purposeful. No decorative animations, no parallax, no confetti, no floating elements.

---

## Accessibility Considerations

All components follow Material 3 accessibility guidance and WCAG 2.2 AA standards minimum.

1. **Color contrast:** All color role pairs must meet WCAG AA contrast ratios: 4.5:1 for text (Body, Title, Headline, Label styles), 3:1 for large text (Display styles) and UI components (outlines, dividers). All interactive states (hover, focus, active) maintain sufficient contrast. The dark theme must be designed with equivalent contrast ratios, not inverted colors.

2. **Touch targets:** All interactive elements have a minimum touch target of 48x48dp (Material 3 guidance). Icon buttons, chips, and small controls meet this even when the visual element is smaller (extended via padding or pseudo-element).

3. **Keyboard navigation:** Every interactive element (buttons, tabs, list items, expansion panel headers, card flip targets, input fields, chips) is reachable via sequential Tab navigation. Tab order follows visual reading order (left to right, top to bottom). Enter and Space activate controls. Arrow keys navigate tab rows and lists. Escape closes dialogs, overlays, and the navigation drawer (on mobile).

4. **Focus indicators:** Every focusable element has a visible focus ring using the Material 3 focus state layer (primary color at `10%` opacity overlay). The focus ring is at least 2px thick with a 2px offset from the element. Never use `outline: none` without providing an alternative focus style.

5. **Semantic HTML:** Use native semantic elements: `<nav>` for the lesson list, `<main>` for content, `<aside>` for the drawer, `<header>` for the top app bar, `<button>` for all buttons, `<details>`/`<summary>` for accordions where possible. ARIA roles and attributes supplement semantics where native HTML is insufficient (e.g., `role="tablist"`, `role="tab"`, `aria-selected`, `aria-expanded`, `role="dialog"`, `aria-modal`).

6. **Screen reader support:** The navigation drawer uses `aria-label` and `aria-hidden` on the scrim. Tab panels use `role="tabpanel"` with `aria-labelledby` pointing to the tab button. Expansion panels use `aria-expanded` on the header. Vocabulary cards announce flip state (use `aria-pressed` or `aria-label` change). Dialog content uses `role="dialog"` with `aria-modal="true"` and focus trapping. Progress text uses `aria-live="polite"` for dynamic updates.

7. **Motion and reduced motion:** All animations use CSS `@media (prefers-reduced-motion: no-preference)` to disable non-essential animation. Card flips, expansion panel slides, and dialog transitions are disabled when reduced motion is preferred. The card flip remains functional (content is always readable on both sides via button toggle as a fallback).

8. **Zoom and text resizing:** The interface must be fully functional at 200% browser zoom with no horizontal scrolling (for the main content area). All text sizes use relative units (`rem`, `em`) so browser font-size settings are respected.

9. **Screen reader announcements:** Use `aria-live="polite"` regions for: answer feedback ("Correct", "Incorrect, the answer is..."), progress updates ("Question 3 of 10"), and completion notifications ("Lesson marked complete"). Dynamic content changes in tab switches are announced appropriately.

---

## Performance Considerations

1. **File size:** `script.js` (~560 KB) is the largest file. This is unavoidable given 18 lessons of embedded content. It loads synchronously but is only parsed once.

2. **Rendering:** Tab content is built as HTML strings and set via `innerHTML` — this is faster than creating DOM elements individually for the volume involved.

3. **Event delegation:** Sidebar uses a single click listener rather than one per lesson item.

4. **No images:** The site has zero image assets — no sprites, no icon PNGs/SVGs. Icons use Material Symbols (variable font version via Google Fonts with `material-symbols-outlined` class) or Unicode characters. No fonts beyond system fonts and the optional Material Symbols variable font.

5. **CSS:** Single stylesheet with no imports, no web fonts. Responsive breakpoints follow Material 3 window size classes: compact (< 600px), medium (600–840px), expanded (> 840px). Transitions use `transform` and `opacity` (GPU-accelerated) where possible.

6. **localStorage:** Writes are batched — `saveProgress()` is called only after state changes, not on every interaction. Data size is minimal (an array of IDs and a flat object of key-value pairs).

7. **Memory:** The `LESSONS` array is created once and held in memory. No lazy-loading or content unloading is needed since only one tab is rendered at a time.

---

## Coding Conventions

### JavaScript

- **Style:** ES5-compatible syntax throughout (no `const`/`let`/`arrow functions` used in the rendering code) to ensure compatibility with older browsers if needed. The data portion (`LESSONS` array) uses `const` generated by the build script.
- **Naming:** Camel case for functions and variables (`renderSidebar`, `currentLesson`). Abbreviations are short and clear (`gi` = grammar index, `ii` = item index, `pv` = phrasal verb).
- **API object:** All interactive functions are exposed via `window.FE` (e.g. `FE.flipVocab`, `FE.switchTab`). This allows inline `onclick` handlers to reference them.
- **DOM queries:** `$` and `$$` aliases for `querySelector` and `querySelectorAll` — kept as simple wrappers, not a full library.
- **Error handling:** Minimal — answer lookup returns empty string on failure; all operations degrade gracefully.
- **IIFE:** The entire rendering/interactivity code is wrapped in `(function() { ... })();` to avoid polluting the global scope (except for the `window.FE` object).

### CSS

- **Custom properties:** All Material 3 design tokens use the `--md-sys-*` naming convention (e.g., `--md-sys-color-primary`, `--md-sys-typescale-body-large`, `--md-sys-shape-corner-small`, `--md-sys-elevation-level1`). Component-specific variables remain lowercase dashed (`.vocab-card`, `.grammar-accordion`).
- **Class naming:** Lowercase dashed format (`.vocab-card`, `.grammar-accordion`, `.quiz-input`). Material 3 component classes use the `md-` prefix convention (`.md-filled-button`, `.md-elevated-card`, `.md-outlined-text-field`).
- **No nesting:** Selectors are flat (no Sass/Less nesting) since the file is plain CSS.
- **Box-sizing:** Universal `border-box` reset at the top.
- **States:** Modifier classes use dot notation (`.vocab-card.flipped`, `.grammar-accordion.open`, `.quiz-input.correct`). State layer classes use Material 3 naming: `.state-hover`, `.state-focus`, `.state-active`, `.state-disabled`.
- **Theme:** Light theme variables in `:root`, dark theme in `[data-theme="dark"]` using `@media (prefers-color-scheme: dark)` as default with manual toggle override.

### HTML

- **Minimal:** The HTML file is a shell — only the app container, sidebar, main content area, and a welcome div. Everything else is rendered dynamically.
- **No inline styles:** All styling is in `styles.css`. The only exceptions are minimal dynamic styles set via `element.style.display` in JS.

---

## Implementation Roadmap

### Phase 1: Data Extraction (Complete)

1. Analyze all 20 markdown files to identify consistent structural patterns.
2. Write a Node.js build script that parses every `Lesson-NN.md` file:
   - Extract title, intro, reading passage, vocabulary, grammar topics with practice exercises, phrasal verbs, real English, review exercises, listening questions, culture note, and answer key.
   - Output a JSON file for validation.
3. Deduplicate (remove lessons 3 and 6).
4. Validate the extracted data — spot-check vocabulary counts, grammar topics, and answer key matching.
5. Clean up build artifacts.

### Phase 2: Static Files (Complete)

1. Write `index.html` — skeleton with sidebar, main content, welcome screen, and script/style links.
2. Write `styles.css` — layout (sidebar + main), typography, card components, form inputs, animations, responsive breakpoint.
3. Write the rendering/interactivity JS code (`script-render.js`) — all functions except the data.
4. Combine data + rendering code into the final `script.js`.

### Phase 3: Interactivity & Polish (Complete)

1. Implement lesson navigation (sidebar click → lesson render).
2. Implement tab switching with active button highlighting.
3. Implement vocabulary flip-cards with shuffle, difficulty marking, and state persistence.
4. Implement grammar accordion with expand/collapse.
5. Implement interactive exercise checking (single check, bulk check, bulk reveal).
6. Implement answer key matching algorithm.
7. Implement phrasal verb search.
8. Implement review session modal.
9. Implement lesson completion toggling with sidebar update.
10. Implement sidebar search filtering.
11. Ensure `localStorage` persistence on all stateful interactions.
12. Clean up build scripts and remove temporary files.

### Phase 4: Verification (Complete)

1. Validate JS syntax with Node.js `vm.Script`.
2. Verify all 18 lessons load and render correctly.
3. Spot-check answer key matching for multiple lessons.
4. Confirm no console errors on page load and interaction.
5. Test responsive layout at mobile widths.
6. Verify `localStorage` persistence across page reloads.

---

## File Reference

| File | Role | Size (approx) |
|---|---|---|
| `index.html` | Application shell | 1.2 KB |
| `styles.css` | All visual styling | 14 KB |
| `script.js` | Lesson data (530 KB) + rendering code (30 KB) | 560 KB |
| `PROJECT.md` | Documentation | This file |
| `MD/Lesson-*.md` | Source lesson content (18 unique files) | Variable |
