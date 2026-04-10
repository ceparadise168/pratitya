# pratitya

A generative visual artwork inspired by Tibetan thangka painting traditions and Buddhist philosophy.

**[View Live](https://erictu.github.io/pratitya/)**

## What is this?

A single HTML file that generates an ever-changing, never-repeating visual meditation. Concentric geometric patterns emerge, exist, and dissolve in breathing cycles — embodying the Buddhist concepts of dependent origination (*pratītyasamutpāda*), impermanence (*anicca*), and emptiness (*śūnyatā*).

The artwork features:

- **Concentric three-layer structure** — a still geometric core surrounded by an active band of generative patterns, fading into void at the edges
- **Mineral color palette** — deep blue, vermilion, emerald, gold, purple — drawn from traditional thangka pigments
- **Individual beings** (眾生) — trackable pattern entities with unique shapes that are born, live, and die independently
- **Breathing life cycles** — each generation of patterns fades in from the center outward and dissolves from the edges inward
- **Traces of past lives** — ghostly remnants of previous generations linger beneath the current one
- **True emptiness** — rare moments of complete darkness where everything vanishes
- **Buddha-nature** — a nearly invisible golden point of light that persists through all states

No sound. No interaction. The viewer's only action is to watch — and to notice what happens inside themselves when beautiful things disappear.

## Philosophy

The title *pratitya* comes from *pratītyasamutpāda* (Sanskrit: प्रतीत्यसमुत्पाद), the Buddhist doctrine of dependent origination — the understanding that all phenomena arise in dependence upon conditions, and cease when those conditions change.

This artwork does not describe that teaching. It creates the conditions for the viewer to experience it directly.

## Technical

- Single `index.html`, zero dependencies
- Canvas 2D API, three layered canvases (trace / core grid / active patterns)
- Polar coordinate system with distance-driven behavior gradients
- 2D continuous field (energy × order) with distance-proportional interpolation speed
- ~1000 lines of vanilla JavaScript

## Run locally

```
open index.html
```

Or serve with any static file server.

## Exhibition notes

This piece is designed for projection or large-screen display in contemplative environments.

- **Environment:** Very low ambient light
- **Projection:** Minimum 2m × 2m, square aspect ratio preferred
- **Viewing distance:** 2-4 meters
- **Seating:** Provide cushions or benches for extended viewing
- **Context:** Place after a quiet transition space, not adjacent to high-stimulation works

## License

MIT

## Acknowledgments

Inspired by the thangka painting traditions of Tibetan Buddhism — their compositional principles, iconometric proportions, and mineral color relationships. This work does not reproduce thangka imagery; it translates the structural wisdom of that tradition into algorithmic form.
