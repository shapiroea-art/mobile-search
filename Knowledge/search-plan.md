# Search Section — Structure & Design Plan

## 1. Overview

Search is a dedicated tab in the app's bottom navigation. It enables operators to find video footage by **predefined object categories** (primary) or **free-text AI descriptions** (secondary). There is no empty state — the screen always has active filter values and displays results immediately upon entry.

**Default state on first load:**
- Object: People
- Cameras: All cameras
- Time: Last 3 days
- Mode: Filter search (primary)

---

## 2. Screen Structure (top to bottom)

### 2.1 Top Bar
- Hamburger menu (left)
- Page title: "Search"

### 2.2 Search Mode Toggle
A segmented control directly below the top bar that switches between the two mutually exclusive search modes:

| Filter Search (default) | Text Search |
|---|---|
| User picks a predefined object category | User types a free-text AI prompt |
| System searches for that object type | System searches for the described scene |

**Why a toggle, not tabs:** The two modes share the same filter bar area and results grid. A compact pill-style segmented control (e.g., `Filters | AI Text`) keeps them tightly coupled while making the active mode obvious. This avoids hiding the secondary mode behind a menu.

### 2.3 Filter / Input Area

This area changes based on the active mode:

#### Mode A — Filter Search (default)
- **Object selector** — a prominent, tappable row showing the currently selected predefined object (e.g., "People" with its icon). Tapping opens the Object Picker (see 3.1).
- **Camera filter** — pill/chip showing count or name (e.g., "All Cameras" or "5 Cameras"). Tapping opens camera picker.
- **Time filter** — pill/chip showing range (e.g., "Last 3 Days"). Tapping opens time picker.
- **Object-specific settings row** (conditional) — appears only for objects that have extra options. For example:
  - LPR → text field for plate number (full or partial match)
  - Faces → option to upload/capture a reference photo
  - Vehicles → color or type selector (if supported)

#### Mode B — Text Search
- **Text input field** — replaces the object selector. Placeholder examples rotate (e.g., "person in red jacket near gate 3"). Has a clear button and optional voice input icon.
- **Camera filter** — same pill as Mode A, persists across modes.
- **Time filter** — same pill as Mode A, persists across modes.

**Key interaction:** Switching between modes preserves the Camera and Time filter values. Only the "what to search for" changes (object category vs. free text).

### 2.4 Results Header
- Result count (e.g., "142 results")
- Sort control (by time, by relevance — relevance makes sense mainly for text search)

### 2.5 Results Grid
- 2-column grid of video thumbnails (the main content area, scrollable)
- Each cell shows: keyframe thumbnail, camera name overlay, timestamp, detection highlight/bounding box
- Tapping a cell opens the video player/detail view
- Infinite scroll / lazy loading as user scrolls down

### 2.6 Bottom Navigation
Standard app tab bar. Search tab is highlighted as active.

---

## 3. Key Components & Interactions

### 3.1 Object Picker (Filter Search mode)

Opens as a bottom sheet when the user taps the object selector.

**5 predefined objects:**

| Object | Icon | Has Extra Settings |
|---|---|---|
| People | 🧑 | No (default selection) |
| Faces | 😶 | Yes — upload/capture reference photo |
| Vehicles | 🚗 | Optional — color/type filter |
| Pets | 🐾 | No |
| LPR (License Plate) | 🔢 | Yes — plate number input |

**Behavior:**
- Grid or list of 5 items. Simple, no scrolling needed.
- Selecting an object immediately updates the search and closes the picker (or shows the extra settings inline before closing).
- If the object has extra settings (LPR, Faces), the picker expands or transitions to show those options before search executes.

### 3.2 Camera Picker

Opens as a full-screen modal or bottom sheet:
- "All Cameras" toggle at top
- Grouped by location/site
- Multi-select with checkboxes
- Search/filter within the camera list (for enterprise clients with 100+ cameras)
- "Apply" button at bottom
- Shows count of selected cameras

### 3.3 Time Picker

Opens as a bottom sheet with:
- Preset quick-select options: Last 1 hour, Last 3 hours, Last 24 hours, **Last 3 days** (default), Last 7 days, Last 30 days
- Custom date/time range picker below presets
- "Apply" button

### 3.4 LPR Settings (Object-specific)

When LPR is selected as the object:
- A text field appears inline below the object selector: "Enter plate number"
- Supports partial matching (e.g., "ABC" matches "ABC-1234")
- Optional: match type selector (Contains / Starts with / Exact)
- Search triggers on "Search" button tap or Enter

### 3.5 Face Search Settings (Object-specific)

When Faces is selected:
- Option to upload a reference photo from gallery
- Option to capture a photo with camera
- Without a reference photo, system returns all detected faces
- With a reference photo, system returns similarity-ranked matches

### 3.6 Search Mode Toggle Details

**Recommended pattern:** Pill-style segmented control with two options.

```
┌─────────────────────────────────┐
│  [ Filters ]  [ AI Text ]      │
└─────────────────────────────────┘
```

- "Filters" is selected by default (filled/active state)
- "AI Text" is the secondary option (outlined/inactive state)
- Switching mode is instant — no confirmation needed
- Camera and Time filter values carry over between modes
- When switching from Text back to Filters, the last-used object is restored

---

## 4. Results Grid — Detail

### 4.1 Grid Cell Anatomy

```
┌──────────────────────┐
│                      │
│   [Video Thumbnail]  │
│   with detection box │
│                      │
│  ┌──────────────┐    │
│  │ Camera Name  │    │
│  └──────────────┘    │
├──────────────────────┤
│ Nov 19 · 3:26 PM     │
│ 4s duration           │
└──────────────────────┘
```

- **Thumbnail**: Keyframe image with the detected object highlighted (bounding box or crop overlay)
- **Camera label**: Semi-transparent overlay at bottom of thumbnail
- **Timestamp**: Below thumbnail, compact format
- **Duration**: How long the object was visible in this clip

### 4.2 Grid Layout Rationale

- **2-column grid** balances information density with thumb-tap targets on mobile
- Aspect ratio: ~16:10 or 4:3 for thumbnails (matching common camera aspect ratios)
- 8–12px gap between cells
- Cards have subtle border or shadow for separation

### 4.3 Result Sorting

| Mode | Default Sort | Options |
|---|---|---|
| Filter Search | Most recent first | Time (newest/oldest) |
| Text Search | By relevance | Relevance, Time (newest/oldest) |

### 4.4 Loading & Pagination

- Skeleton placeholders while results load (2-column grid of gray rectangles)
- Infinite scroll with a loading spinner at bottom
- Pull-to-refresh to re-run the current search

---

## 5. State Transitions

### 5.1 Default → Change Object
1. User taps object selector
2. Bottom sheet opens with 5 object options
3. User taps "Vehicles"
4. (If object has settings → show inline settings)
5. Bottom sheet closes
6. Results grid refreshes with vehicle detections

### 5.2 Filter → Text Search
1. User taps "AI Text" in segmented control
2. Object selector transforms into text input field
3. Camera and Time filters stay visible and retain values
4. Keyboard opens automatically
5. User types prompt and taps Search / Enter
6. Results grid refreshes with AI text search results

### 5.3 Text → Filter Search
1. User taps "Filters" in segmented control
2. Text input transforms back into object selector
3. Last-used object (e.g., People) is restored
4. Results grid refreshes with filter-based results

### 5.4 Change Time / Cameras
1. User taps the Time or Camera pill
2. Picker opens (bottom sheet or full screen)
3. User makes selection and taps Apply
4. Pill updates to show new value
5. Results refresh automatically

---

## 6. Design Considerations

### 6.1 No Empty State
The screen always shows results because all three filters always have values. On first entry, the user sees People detections from all cameras in the last 3 days. This makes the search tab feel immediately useful — no blank screens, no "start searching" prompts.

### 6.2 Filter vs. Text — Keep Them Clearly Separated
The toggle approach (rather than a unified bar that tries to do both) is correct for your use case because:
- Filter search (predefined objects) is deterministic and fast — the system knows exactly what to look for
- Text search is AI/probabilistic — results depend on prompt quality
- Mixing them would create confusion about what the system is actually searching for
- Operators in time-sensitive situations need predictable behavior from filter search

The segmented control makes the active mode unmistakable at a glance.

### 6.3 Object Selector Prominence
The predefined object is the most important filter. Make it visually dominant:
- Larger than the Camera/Time pills
- Show the object icon + name
- Consider a dedicated row (not crammed into a pill strip)
- Active object should feel like a "current search identity"

### 6.4 Camera/Time as Persistent Context
Camera and Time are contextual constraints, not the primary search axis. They should:
- Be visible but secondary (smaller pills below the object/text input)
- Persist across mode switches
- Show their current value at all times (never "Select cameras…")
- Default intelligently (all cameras, last 3 days)

### 6.5 Object-Specific Settings — Progressive Disclosure
Most objects (People, Pets) need no extra settings. Don't force all users through a settings step. Only reveal additional options when the selected object supports them:
- LPR → plate number field slides in
- Faces → reference photo option slides in
- Other objects → no extra step, search executes immediately

### 6.6 Light Theme
Use the established light theme palette from the existing prototypes:
- Background: `#FFFFFF`
- Secondary bg: `#F5F6F8`
- Text: `#101828`
- Accent: `#4A55F2`
- Borders: `#E4E7EC`
- Font: Inter

### 6.7 Results Grid — Density vs. Clarity
- 2-column grid is the sweet spot for mobile: shows enough context per thumbnail while keeping results scannable
- Avoid single-column cards (too much scrolling for high-result-count searches)
- Avoid 3-column grid (thumbnails become too small to see detection details)
- Consider letting users switch between grid and list view (list shows more metadata per result)

### 6.8 Performance Signals
For enterprise clients with hundreds of cameras and days of footage:
- Show a result count early ("~240 results" even before all load)
- Show a progress indicator if the search is still running across cameras
- Allow browsing partial results while search continues

### 6.9 Interaction with Video Detail
When user taps a result thumbnail:
- Open a video player view with the clip starting at the detection moment
- Show the detection bounding box on the video
- Allow scrubbing forward/backward
- Provide options: Save to case, Archive, Share, View full camera timeline

---

## 7. Comparison with Verkada

| Feature | Verkada | Lumana (this plan) |
|---|---|---|
| Search input | Unified text field (object + AI) | Separated toggle (clearer intent) |
| Object categories | People, Faces, Vehicles, LPR | People, Faces, Vehicles, Pets, LPR |
| Text search | AI-powered natural language | AI-powered natural language |
| Image-based search | Upload/capture photo | Via Face search object settings |
| Default state | Needs user action to start | Auto-populated with defaults (no empty state) |
| Camera selection | Per-camera or all | Multi-select with grouping |
| Results format | Timeline + cards | 2-column thumbnail grid |
| Mobile-specific | Recently added (Oct 2025) | Native mobile-first design |

**Differentiators for Lumana:**
- No empty state → faster time-to-value
- Clear mode separation → less confusion for operators under pressure
- Object-specific settings (LPR plate input, face reference) → more targeted searches
- Pets as a category → broader market appeal

---

## 8. Edge Cases & Open Questions

### Edge Cases to Handle
1. **Zero results for current filters** — Show an informational state ("No vehicles detected in the last 3 days across all cameras") with suggestions to broaden time or add cameras. This is not the same as an empty state — the user actively set filters that returned nothing.
2. **Very high result count** — Consider a "Load more" approach or time-bucketed sections (results grouped by hour/day) for easier scanning.
3. **Camera goes offline** — Camera pill should indicate if selected cameras include offline ones.
4. **Long-running text search** — AI text search may take longer than filter search. Show a "Searching..." state with cancel option.

### Open Questions
1. **Should the search auto-execute on filter change?** (Recommended: yes — changing object, time, or cameras immediately re-runs the search. No manual "Search" button needed for filter mode.)
2. **Should text search require a button tap?** (Recommended: yes — "Search" button or Enter key to execute, since partial prompts shouldn't trigger expensive AI queries.)
3. **Result detail view: inline expand or new screen?** (Recommend: new screen for video playback, since video needs full-screen focus.)
4. **Grid vs. list toggle for results?** (Consider for v2 — grid is sufficient for v1.)
5. **Search history — where does it live?** (Consider adding a "Recent" section accessible from the text input field, showing past text prompts and filter combinations.)
