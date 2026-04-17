# TANAKA

> *An interactive visualization of color, time, culture, and the graphic design of four Japanese masters — Nagai, Tanaka, Fukuda, and Ishioka.*

**Live:** https://augustave.github.io/TANAKA/

---

## What this is

A single-file HTML experience (`index.html`) that turns a dataset of 178 colors extracted from a poster of four legendary post-war Japanese graphic designers into a living, interactive atlas. Each color is a particle you can hover. Each particle carries its hex value, its year, its designer, its cultural symbolism across West African, Zulu, Berber, Native American, and Roman traditions, and — in the final form — its place in a knowledge graph connecting Perkin's mauveine, the Synthetic Pigment Revolution, Impressionism, Berlin & Kay's linguistic universals, and Yoruba spirituality.

Three modes:

- **Orbital** — four designer clusters breathing in formation, their color particles drifting in spirals sized by the original height of each color band in the source poster
- **Timeline** — chronological bands, 1960 to 1993, each designer a horizontal stream
- **Network** — a force-directed knowledge graph of 31 concepts and 34 relationships across five domains: Production & Tech, Artistic Application, Linguistic Theory, Sociocultural Identity, Color Theory

No dependencies. No build step. No fetch calls (data inlined). Works from any static host.

---

## Who the four are

| | | |
|---|---|---|
| **永井一正** | Nagai Kazumasa | 54 colors · 1960–1992 |
| **田中一光** | Tanaka Ikko | 22 colors · 1961–1991 |
| **福田繁雄** | Fukuda Shigeo | 41 colors · 1962–1993 |
| **石岡瑛子** | Ishioka Eiko | 61 colors · 1963–1993 |

The palette data came from `palette.json` — a structured extraction of every color band visible in a poster by Take Nakano (中野豪雄). Each entry carries normalized coordinates, pixel positions, hex, RGB, band height in pixels, and the approximate year inferred from vertical position on the poster.

---

## The build — a honest log

### Stage 0 — "YOLO"

The user handed over the NAGAI folder and said *Yolo*. No brief. No specification. One file inside: `palette.json`, 178 entries, no context beyond column headers pointing to four designer columns.

The first move was to just build something with it. Static site, four vertical color strips, view toggles between Timeline / Grid / Hue Wheel / River. It rendered. The user opened it directly from the filesystem and saw nothing — `fetch('palette.json')` was blocked by the browser's file:// CORS policy.

Fixed by spinning up a Python HTTP server via a `.claude/launch.json`:

```json
{
  "version": "0.0.1",
  "configurations": [{
    "name": "nagai",
    "runtimeExecutable": "python3",
    "runtimeArgs": ["-m", "http.server", "8765", "--directory", "..."],
    "port": 8765
  }]
}
```

The page appeared. Colored strips on black. Functional. Accurate. Boring.

### Stage 1 — "boring"

The user said it was boring. They were right. A timeline of color bars is a data table with extra steps. The work deserved more.

**Complete rewrite.** Threw out the four views. Built a single canvas-based particle system:

- Every color becomes a Particle object with a home position, current position, velocity, phase, breath speed, drift amplitude
- 4 designers arranged in orbital clusters around the center, each cluster expanding into a year-based spiral
- Particles breathe (sine-wave oscillation on their home position)
- Mouse proximity pushes particles away with inverse-square force
- Nearby same-designer particles draw gossamer connection lines
- Radial glow gradient on hover
- Subtle vignette over the whole canvas

Added the second mode — **Timeline** — as a particle rearrangement rather than a separate view. Click the canvas and every particle springs to a new home position mapped chronologically across horizontal designer bands. Same particles, same physics, different targets.

Also: inlined the JSON directly into the HTML so the file works standalone without a server.

Result: a constellation of 178 breathing colors, hoverable, with a floating year display that ghosts into the background at the hex the cursor is nearest.

The user said "*There you go that is more interesting.*"

### Stage 2 — Cultural symbolism

The user dropped a CSV: *Symbolism, Psychology, and Cultural Meanings of Colors*. 17 rows covering Yoruba indigo, Ashanti black, Zulu red, Berber yellow, Roman imperial purple, Pan-African green, Tuareg blue, Uganda earth tones, Kuba royal blue and yellow.

Built a **SYMBOLISM** database inside the HTML — each entry had a hue range, saturation bounds, luminance bounds, a list of symbols, psychology notes, and associated cultures. Then a `classifyColor()` function that takes an RGB triple, converts to HSL, and finds the closest match.

Extended the tooltip to show:
- Color name (categorical: Red, Gray, Brown, etc.)
- Hex
- Designer and year
- **Symbolism** — e.g. *"Vitality, courage, life force · Passion, love, protection"*
- **Psychology** — e.g. *"Excitement, urgency — increases heart rate and alertness"*
- **Cultures** — e.g. *"Yoruba, Zulu"*

Now every hover doesn't just tell you *what* — it tells you *why*. Nagai's `#AF2325` reds echo Zulu umhlophe. Fukuda's whites carry Yoruba funfun purity. Ishioka's earth tones ground in Ugandan kikoy.

### Stage 3 — The knowledge graph

The user dropped a Mermaid diagram and structured JSON — a full knowledge graph of the history of color: Perkin, mauveine, coal tar, aniline dyes, the synthetic pigment revolution. Impressionism, en plein air, solar palette, optical mixing. Berlin & Kay, Basic Color Terms, the evolutionary hierarchy, linguistic relativity, Sapir-Whorf, the Emergence Hypothesis. Yoruba indigo, adire cloth, Native American red-as-earth-life.

Built a third mode — **Network** — as a force-directed graph rendered to the same canvas:

- 27 nodes initially, expanded to 31 in the second pass
- Each node belongs to one of five groups: Production & Tech, Artistic Application, Linguistic Theory, Sociocultural Identity, Color Theory
- Group halos: large radial gradients tinted per group, rendered behind the nodes, auto-sized from the convex hull of each cluster
- Physics: inverse-square repulsion between every node pair, spring attraction along every edge (ideal length 90px), center gravity, 0.82 damping
- Node hover: glow, opacity bump, enlarged radius, white ring, edge highlighting, edge labels appearing
- Group labels floating above each cluster
- Particle layer dims to 4% opacity when Network mode activates, ghosting the original palette behind the concept map

The second pass from the user added:
- **Alizarin Crimson** — the first natural pigment replicated synthetically
- **Natural Ultramarine** — the lapis lazuli blue whose price collapsed
- **Additive Color (RGB)** and **Subtractive Color (CMY)** — a new Color Theory cluster
- **Red Symbolism** — a bridge node between Native American culture and the Earth/Life Power concept
- Refined relationships — `accidentally synthesized`, `collapsed price of`, `offers alternative to`, `materially enabled`, `traditional connection`

### Debugging, the unglamorous parts

- **CORS / file://** — the original fetch failed when opening the HTML directly. Solved by inlining the JSON and by setting up a local Python server for development.
- **Stale cache in preview** — the preview kept showing the old page. Force-reload with `location.reload(true)` fixed it.
- **Mode bar clicks not registering** — the mode bar buttons were inside a `pointer-events:none` container. Pulled it out to be a top-level `position:fixed` element.
- **Canvas clicks eating button clicks** — clicking the Network button also fired the canvas click handler which would toggle mode back to Orbital. Added a `lastModeBarClick` timestamp guard with a 300ms debounce.
- **Graph nodes exploding off-screen** — original force simulation used repulsion of 8000 with attraction of 0.0008, producing runaway velocities. Tuned to repulsion 3000, attraction 0.005, damping 0.82, center pull 0.008, and added a `resetGraphPositions()` call on mode entry to seed nodes in a tight angular arrangement.

### Deploy

`git init`, committed three files (`index.html`, `palette.json`, `.claude/launch.json`), pushed to `github.com/augustave/TANAKA`, enabled Pages via the GitHub API pointing at the `main` branch root. Live at https://augustave.github.io/TANAKA/.

---

## The intent

The intent was never "visualize a CSV." The intent was to make an object that does justice to what these four designers actually did.

Nagai, Tanaka, Fukuda, and Ishioka were not color-pickers. They worked inside a post-war Japan that inherited centuries of indigo traditions and collided with the synthetic pigment revolution, Western modernism, Bauhaus formalism, and a global design language that was itself downstream of Perkin's accidental mauveine in 1856. Every one of those 178 color choices carries a history — material (what pigments were affordable), cultural (what a color *means* in which tradition), linguistic (whether the designer's language even had a basic term for that hue), and personal.

A table of hex codes says none of that.

The final form — orbital cluster → chronological timeline → knowledge network, with cultural symbolism available on every hover — is an attempt to let a viewer walk the whole chain in under a minute. Pick a color. See its designer. See its year. See its cultural meaning. See the 150 years of industrial chemistry, linguistic theory, and artistic practice that made that specific hex code available at that specific moment to that specific designer.

That's the goal. Not a chart.

---

## Progression, in one paragraph each

**First form** — four colored strips arranged by year. Boring. Accurate but inert.

**Second form** — living particle constellation. Breathing, drifting, mouse-responsive. Two modes (orbital / timeline) as particle state changes, not as separate views. Interesting.

**Third form** — every color now carries cultural and psychological meaning via a hue/saturation/luminance classifier mapped to a symbolism database. Hovering reveals the deep layer.

**Fourth form** — a knowledge graph mode connecting 31 concepts across color history: chemistry, art movements, linguistic theory, cultural semiotics, color theory. Force-directed, group-haloed, with Japanese design positioned as the receiving node that absorbs all of these influences.

**Current form** — all three modes coexisting in one canvas, switching between them without a page change, with the palette ghosting behind the concept map and the concept map snapping away to let the colors breathe.

---

## Technical specifics

- **Single file**: `index.html` (~145 KB with inlined palette)
- **Zero dependencies**: no React, no D3, no frameworks — raw Canvas 2D + vanilla JS
- **Data**: 178 palette entries + 31 graph nodes + 34 graph edges + 11 symbolism rules, all inlined
- **Fonts**: Google Fonts (Inter, 200/300/400/500) — the only external fetch
- **Rendering**: single canvas, 60fps requestAnimationFrame loop
- **Force sim**: O(n²) repulsion is fine at n=31 nodes; no quadtree needed
- **Hover hit-testing**: nearest-neighbor scan, threshold `radius * 2.5` for graph nodes, `40px` for particles

### File layout

```
NAGAI/
├── index.html              ← the whole experience
├── palette.json            ← source data (duplicated inside index.html)
├── README.md               ← this file
└── .claude/
    └── launch.json         ← dev server config for local preview
```

---

## Credits

- **Source poster**: Take Nakano (中野豪雄) — *Take Nakano 中野豪雄.jpeg*
- **Palette extraction**: the source `palette.json` with normalized coordinates and approximate years
- **Symbolism data**: *Symbolism, Psychology, and Cultural Meanings of Colors* (Table 1) — 17 rows across Yoruba, Ashanti, Zulu, Berber, Tuareg, Kuba, Pan-African, Uganda, Xhosa, Roman/Western traditions
- **Knowledge graph**: compiled from Mermaid diagrams covering Perkin's 1856 mauveine discovery, the synthetic pigment revolution, French Impressionism and its technological enablers, Berlin & Kay's 1969 Basic Color Terms, and cultural semiotics of color across continents
- **The four designers**: Nagai Kazumasa, Tanaka Ikko, Fukuda Shigeo, Ishioka Eiko — post-war Japanese graphic design, 1960–1993

---

## License

Do what you want with it. The data points to cultures and artists whose work predates any of this — treat their meanings with care.
