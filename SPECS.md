# Note Reader — Specification

## Overview
A single-file web app for learning to read music notes on a grand staff. Presents a random note on treble or bass clef, the user identifies it by clicking C–B buttons or pressing keyboard keys, and gets correct/wrong feedback with streak tracking.

**Live:** https://bmtran.github.io/note-reader/
**Repo:** https://github.com/bmtran/note-reader

---

## Visual & Rendering Specification

### Scene Setup
- **Renderer:** VexFlow 4 (cdn.jsdelivr.net/npm/vexflow@4.2.5/build/cjs/vexflow-bravura.js)
- **Layout:** Single SVG with grand staff (treble + bass clef connected by brace)
  - SVG size: 320×230px
  - Treble stave: y=10
  - Bass stave: y=90
  - Brace connector (`StaveConnector.type.BRACE`) between staves
  - Bar line connector (`StaveConnector.type.SINGLE`) between staves
- **Background:** Animated gradient card (`linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%)`) with glassmorphism (`backdrop-filter: blur(12px)`)
- **Staff area:** Light gray (#f8f9fa) rounded panel that flashes green/red on answer

### Target Note
- Whole note (hollow ellipse): white fill, dark stroke (`#222`), no stem
- Rendered with VexFlow `StaveNote` + `Voice` + `Formatter` (centered on stave)
- Duration: `'w'` (whole)

### Gray Hint Notes (Show Path)
- Raw SVG `<ellipse>` elements appended after VexFlow renders
- White fill (`#ffffff`), gray stroke (`#bbbbbb`), 1.5px stroke-width
- X position: evenly distributed from `leftMargin=60` to `centerX`
- Y position: calculated per note based on diatonic path

### Labels (Show Labels)
- Raw SVG `<text>` elements appended after VexFlow renders
- Font: system-ui 13px bold, fill `#aaaaaa`, text-anchor middle
- Treble clef: labels at y=28 (above staff)
- Bass clef: labels at y=105 (below staff)

### Color Palette
```
--correct: #4ecca3
--wrong:   #e94560
--c: #ff6b6b   (C)
--d: #ff9f43   (D)
--e: #ffd93d   (E, text dark)
--f: #6bcb77   (F)
--g: #4d96ff   (G)
--a: #c77dff   (A)
--b: #ff6bcb   (B)
```

### UI Components
- **Clef toggle:** Pill buttons — Both / Treble / Bass (active = filled white)
- **Note buttons:** 7 rainbow buttons C–B, rounded, with hover/active scale animations
- **Scoreboard:** Score (X / Y), Streak (with ♥), Best (from localStorage)
- **Hint buttons:** Show Path → Show Labels (unlocks after Path) → Reset Hints
- **Keyboard:** C D E F G A B keys map to note buttons

---

## Interaction Specification

### Quiz Flow
1. App picks a random note from the active clef's pool (unique note names only)
2. User clicks a note button or presses a key
3. **Correct:** green flash, score++, streak++, next question after 900ms
4. **Wrong:** red flash + shake animation, streak resets, shows correct answer, next question after 1500ms

### Hint System (2-stage)
1. **Show Path:** Draws gray whole-note ellipses from middle C diatonically up/down to the note before the target (omits target itself)
2. **Show Labels:** After Path, adds `<text>` labels A–G above/below each gray note (not the target)
3. **Reset Hints:** Clears both hint layers

### Clef Toggle
- **Both:** Randomly picks treble or bass (50/50) on each new question
- **Treble:** Questions only from treble pool
- **Bass:** Questions only from bass pool

---

## Note Ranges

### Treble Clef
C4 (middle C) → G5
```
C4, D4, E4, F4, G4, A4, B4, C5, D5, E5, F5, G5
```

### Bass Clef
A2 → C4 (middle C)
```
A2, B2, C3, D3, E3, F3, G3, A3, B3, C4
```

### Diatonic Path (for hint system)
- **Ascending** (target is c/4 or above): walks c/4 → d/4 → ... → target
- **Descending** (target below c/4): walks c/4 → b/3 → a/3 → ... → target
- Gray notes shown: entire path **excluding** the target note

---

## State & Persistence

### Game State
```javascript
{
  clefMode: 'both' | 'treble' | 'bass',
  current: { clef, note: { name, key } },
  score: 0, total: 0,
  streak: 0,
  bestStreak: localStorage('notereader-best') | 0,
  answered: false,
  hintLevel: 0 | 1 | 2,
}
```

### localStorage
- Key: `notereader-best` — stores best streak integer

---

## Technical Notes

### VexFlow 4 API
- **DO NOT use `note.setX()`** — method does not exist in VexFlow 4
- Note X positioning: use `Voice` + `Formatter.format([voice], width)`
- Hint notes: draw as raw SVG `<ellipse>` after VexFlow render completes (avoids setX API limitation)
- Import: `const { Renderer, Stave, StaveNote, StaveConnector, Voice, Formatter } = Vex.Flow`

### Files
- `index.html` — single-file app (~18KB), no build step
- `SPECS.md` — this file
- Location: `~/git/note-reader/` / deployed to GitHub Pages

### Known Fixed Bugs
- ~~`setX` not a function~~ → Fixed by using Voice+Formatter for target note, raw SVG ellipses for hint notes
- ~~Show Labels not rendering~~ → Fixed by replacing broken VF.Annotation with raw SVG `<text>` injection
