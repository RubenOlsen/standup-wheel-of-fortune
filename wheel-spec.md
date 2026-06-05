# Standup Wheel of Fortune, Complete Specification

## Overview

A single self-contained HTML page that displays a colorful spinning wheel populated from a text file (or saved team list), picks a random name with realistic deceleration, and announces the winner with confetti and sound effects. Designed for standup meetings to randomly select who goes first.

## Architecture

- **Single file:** `index.html`, all HTML, CSS, and JS embedded
- **Zero dependencies:** No external libraries, frameworks, or assets
- **APIs used:** Canvas API (wheel rendering), Web Audio API (sound effects), FileReader API (file upload), localStorage (team persistence)
- **State:** All state managed in plain JS variables; team lists persisted to localStorage

## Visual Design

### Theme
- Dark background (`#1a1a2e`)
- Carnival/game show aesthetic with bold colors and glow effects
- System sans-serif font stack (`-apple-system, BlinkMacSystemFont, 'Segoe UI'`)

### Title
- Text: "Standup Wheel of Fortune"
- Gold shimmer effect: animated horizontal linear gradient (`#f9d423`, `#ff6f00`, `#ffe176`) scrolling via `background-clip: text`
- Gold drop shadow glow
- Font: 4rem, weight 900, 2px letter spacing
- Animation: continuous 3s linear shimmer loop

### Color Palette (Wheel Slices)
96 high-saturation colors arranged so adjacent entries always contrast (red, teal, yellow, purple, green, orange, blue, pink cycle):
```
#FF2D55, #00BCD4, #FFD600, #7C4DFF, #00E676, #FF6D00,
#2979FF, #F50057, #00BFA5, #FF9100, #304FFE, #C6FF00,
#D500F9, #FFAB40, #00B8D4, #FF1744, #64DD17, #AA00FF,
#FFD740, #0091EA, #FF3D00, #1DE9B6, #E040FB, #AEEA00,
#FF5252, #00ACC1, #FFC400, #651FFF, #69F0AE, #FF6E40,
#448AFF, #FF4081, #26A69A, #FFAB00, #536DFE, #B2FF59,
#E040FB, #FFB300, #0097A7, #FF1744, #76FF03, #7B1FA2,
#FFD54F, #1565C0, #FF3D00, #00E5FF, #CE93D8, #A7FF00,
#F44336, #009688, #FDD835, #9C27B0, #4CAF50, #E65100,
#1E88E5, #EC407A, #00897B, #F9A825, #283593, #8BC34A,
#AB47BC, #FFB74D, #00838F, #E53935, #AED581, #6A1B9A,
#FFE082, #0277BD, #BF360C, #4DD0E1, #BA68C8, #9CCC65,
#EF5350, #00796B, #FFCA28, #5E35B1, #66BB6A, #D84315,
#42A5F5, #F06292, #00695C, #FFB300, #3949AB, #7CB342,
#8E24AA, #FFA726, #006064, #C62828, #81C784, #4A148C,
#FFE57F, #01579B, #DD2C00, #80DEEA, #9575CD, #CCFF90
```

### Color Assignment
- Each name is assigned a color from the palette at file-load time, stored as `{name, color}` objects
- Colors are assigned sequentially by index: `COLORS[i % COLORS.length]`
- Colors remain permanently bound to names, removing a name does **not** reassign colors to remaining names
- On reset, restored names keep their original colors

## Page States

### State 1: File Upload / Saved Teams
- **Saved teams section** (shown if localStorage has saved teams):
  - Dropdown select listing all saved team files with name count
  - "Load" button: loads selected team onto the wheel
  - "Edit" button: opens modal to edit names and rename the list
  - "New" button: opens modal pre-filled for creating a new list from scratch
  - "Delete" button: removes selected team from localStorage
  - Horizontal divider ("or upload a new file") separates saved teams from upload area
- **Upload area:**
  - Dashed-border drop zone (3px dashed `#4ECDC4`, 16px border radius)
  - Text: "Drop a .txt file here or click to upload" with hint "One name per line"
  - Hover/dragover: teal background tint, border changes to `#FF6B6B`
  - Hidden `<input type="file">` triggered by click on drop zone
  - Accepts `.txt` and `text/plain` files
  - Uploaded files are automatically saved to localStorage

### State 2: Wheel Active
- Upload area and saved teams section hidden
- Wheel container fades in with 0.6s ease-out opacity + scale (0.8 to 1.0) transition
- Wheel canvas displayed with roll button centered over it
- Below the wheel: names counter ("X names left"), then "Choose New File" button

## Edit Modal

- Full-screen overlay with dark backdrop (`rgba(0, 0, 0, 0.7)`)
- Modal content: dark card (`#16213e`) with teal border, 16px border radius
- **Name input:** text field for the team/file name
- **Textarea:** one name per line, editable
- **Save button:** persists changes to localStorage, reloads the saved files dropdown
- **Cancel button:** dismisses without saving
- Used for both editing existing lists and creating new ones

## Wheel Rendering (Canvas)

### Sizing
- Canvas dimensions: 75% of the smaller viewport dimension (`min(innerWidth, innerHeight) * 0.75`)
- Minimum size: 300px
- Responsive: recalculates on window resize (only when not spinning)
- Wheel container has 20px top margin

### Drawing Layers (bottom to top)
1. **Shadow:** Blurred black circle (`blur(10px)`, radius + 8px)
2. **Outer ring:** Dark ring (`#2c3e50`) with gold stroke (`#f1c40f`, 3px)
3. **Slices:** Equal-angle pie slices from center, each using its assigned color. White borders (2px) between slices
4. **Name text:** White text with black stroke outline (3px). Right-aligned, positioned along the middle angle of each slice. Font size scales proportionally: `min(radius * 0.12, radius * sliceAngle * 0.45)`, minimum 10px. Names truncated with ellipsis if they exceed available space
5. **Pointer flapper:** Drawn on top of the wheel (see Pointer section)

### Pointer / Flapper
- Position: top center of canvas, pivoting from point at y=6
- Shape: sleek trapezoid pointing **downward**, top width = `radius * 0.06` each side, tapering to `15%` of that width at the tip. Height = `radius * 0.22`
- Color: red (`#e74c3c`) with white 2px stroke
- Pivot circle: fully round circle at pivot point, radius = `ptrW`, dark red (`#c0392b`) with white stroke
- **Animation during spin:** Spring physics simulation
  - Spring constant: 0.3, damping: 0.75
  - On each slice boundary crossing, the flapper receives a "kick" (velocity impulse)
  - Kick strength is **inversely proportional** to wheel speed: `max(0.08, 0.35 - speed * 40)`, bigger bounces when wheel is slow
  - Flapper resets to 0 angle when spin completes

## Roll Button

- Position: absolute center of wheel canvas
- Shape: circle, size = 16% of wheel diameter (scales with wheel)
- Font size: 22% of button size
- Text: "ROLL" (uppercase, 900 weight, 2px letter spacing)
- Border: 4px solid white (`rgba(255, 255, 255, 0.9)`)
- Background: animated `conic-gradient` rotating through gold/pink/purple/cyan/green/gold over 3s
  - Uses `@property --btn-angle` for smooth CSS rotation
- Glow: multi-layered box-shadow pulsing every 1.5s (gold + pink + purple layers) plus a persistent dark drop shadow (`0 6px 20px rgba(0,0,0,0.6)`) on all sides
- Hover: intensified glow
- Active: scale down to 0.92
- Disabled state: grey background (`#555`), no animation, 0.5 opacity, no border glow
- Disabled when: spinning, or only 1 name remains after removal

## Names Counter

- Position: between the wheel and the "Choose New File" button
- Text: "X names left" (dynamically updated)
- Font: 2rem (50% of header size), weight 700, semi-transparent white (`rgba(255,255,255,0.7)`)
- Spacing: 25px margin below (before "Choose New File" button)
- Updates when: file loaded, name removed, wheel reset

## Spin Mechanics

### Trigger
- Click on Roll button
- Blocked if: `spinning === true`, `names.length === 0`, or `wheelReady === false`

### Animation
- Easing: cubic ease-out (`1 - (1 - progress)^3`)
- Normal spin: 8-14 full rotations + random extra angle, 10s duration
- Single-name spin: 2-4 full rotations, 3s duration
- Uses `requestAnimationFrame` loop

### Winner Calculation
- Pointer is at canvas top = `-PI/2` in canvas coordinate system
- Formula: normalize `(-PI/2 - rotation)` to `[0, 2*PI)`, divide by slice angle, floor to get index
- Winner is the name whose slice is directly under the pointer

### Post-Spin
- Flapper settles to center
- 300ms delay, then fanfare plays and winner overlay appears
- Wheel does **not** automatically spin again, user must manually click Roll

## Sound Effects (Web Audio API)

### Initialization
- `AudioContext` created on first Roll button click (requires user gesture)

### Tick Sound
- Plays on each slice boundary crossing during spin
- Sine oscillator, frequency 800-1200Hz (randomized)
- Volume: 0.15, exponential ramp to 0.001 over 50ms
- Natural effect: ticks slow down as wheel decelerates

### Fanfare
- Plays when winner is determined
- Four triangle wave notes in sequence: C5 (523Hz), E5 (659Hz), G5 (784Hz), C6 (1047Hz)
- Each note: 150ms apart, volume 0.2, exponential ramp to 0.001 over 400ms

## Winner Overlay

The winner announcement is displayed directly over the dimmed wheel, no separate modal box.

### Wheel Dimming
- When the overlay is shown, the wheel canvas and roll button are dimmed via CSS filter: `brightness(0.15) saturate(0.3)`
- Filter transition: 0.4s ease-out
- Only the canvas and roll button are filtered; the overlay text and buttons remain at full brightness

### Overlay Positioning
- Absolutely positioned inside `#wheel-container` covering the full wheel area (`inset: 0`)
- Flexbox column layout, centered both axes
- z-index: 50 (above roll button's z-index: 10)
- Fade-in: 0.4s opacity transition

### Content
- **"THE WINNER IS"** label: uppercase, 1.1rem, semi-transparent white (`rgba(255,255,255,0.8)`), 2px letter spacing, 20px margin below
- **Winner name:** 4.5rem, weight 900, gold gradient text (`#f9d423` to `#ff6f00` to `#f9d423`) via `background-clip: text`, with dark drop-shadow. 28px margin below
- **Remove question:** "Remove [name] from the wheel?", 1rem, semi-transparent white (`rgba(255,255,255,0.7)`), 24px margin below

### Buttons
Two action buttons side by side in a flex row (12px gap):
1. **"Yes, remove"**, orange-to-pink gradient (`#FF6D00` to `#FF2D55`), white text. Closes overlay, removes winner from wheel, updates names counter, redraws wheel
2. **"No, keep"**, semi-transparent white background (`rgba(255,255,255,0.15)`), white text. Closes overlay, keeps winner on wheel

One conditional button:
3. **"Reset Wheel"**, red gradient (`#FF6B6B` to `#e74c3c`), white text. Only shown when all names are used. Restores all removed names with their original colors, updates counter, and redraws

### Edge Cases
- **1 name remaining after removal:** Roll button disabled; that name is auto-announced as winner after 600ms delay with fanfare
- **All names used:** Overlay shows "All names used!" with only the Reset Wheel button; label and question are hidden

## Confetti

- Separate full-screen canvas (z-index: 200, pointer-events: none)
- **Continuous animation:** confetti runs as long as the winner overlay is displayed, stops when dismissed
- Initial burst: 450 rectangular particles, random colors from COLORS array
- Ongoing: 90 new particles spawned every 60 frames to maintain continuous rainfall
- Initial positions: spread across screen width, above viewport
- Physics: gravity (vy += 0.05 per frame), horizontal drift, rotation
- Dead particles (fallen off screen) are removed to prevent memory buildup
- Confetti is stopped via `cancelAnimationFrame` and canvas cleared when overlay closes

## "Choose New File" Button

- Position: below the names counter, under the wheel
- Styling: ghost button (no background, 2px semi-transparent white border, 20px border radius)
- Hover: border turns teal, text turns white
- Behavior: clears file input value, triggers file picker. On new file load: resets all state (names, removedNames, rotation), reassigns colors, updates counter, redraws wheel

## Team Persistence (localStorage)

### Storage Format
- Key: `standup-wheel-files` in localStorage
- Value: JSON object mapping file names to arrays of name strings
- Example: `{"team-alpha.txt": ["Alice", "Bob", "Charlie"]}`

### Saved Teams UI
- Dropdown selector shows all saved teams with name count
- File names displayed without extension in the dropdown
- **Load:** Populates the wheel with the selected team's names
- **Edit:** Opens modal with team name and names (one per line) for inline editing
- **New:** Opens blank modal for creating a new team from scratch
- **Delete:** Removes team from localStorage after confirmation, updates dropdown

### Auto-save Behavior
- When a `.txt` file is uploaded, its contents are automatically saved to localStorage using the filename as key
- Edits made via the Edit modal are persisted immediately on Save

## Responsive Behavior

- Wheel canvas and roll button scale dynamically with viewport
- Window resize handler recalculates sizes (skipped during spin)
- Mobile breakpoint (600px): smaller roll button (60x60px), smaller winner name text (2rem)

## Edge Cases Handled

| Case | Behavior |
|------|----------|
| Empty file | Alert message, upload area stays visible |
| 1 name in file | Brief 3s spin, that name wins |
| 1 name remaining after removals | Auto-announced, no spin |
| All names removed | "All names used!" message with reset option |
| Very long names | Truncated with ellipsis based on available slice space |
| Double-click Roll | Blocked by `spinning` flag |
| Resize during spin | Ignored until spin completes |
| New file while wheel active | Full state reset, wheel redraws |
| Name removed from wheel | Remaining names keep their original colors |
| No saved teams in localStorage | Saved teams section hidden, only upload shown |
| Duplicate file upload | Overwrites existing entry in localStorage |
