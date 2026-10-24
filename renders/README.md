# Renders

Tool: [Blender](https://www.blender.org) 5.2.

## Layout

- `<scene>/`: one folder per scene, holding `<scene>.blend`. Import geometry from the STEP exports of the mechanical parts.
- `<scene>/out/`: rendered images, not committed. Images used in documentation are copied to `docs/`.

Blender backups (`*.blend1`) are not committed.

## Web

- `web/`: standalone HTML previews (three.js), for product-side views that do not need Blender. `web/concept.html` is the source of the concept-art cover in [`docs/concept-art/`](../docs/concept-art/README.md) and the page at [opendap.cc](https://opendap.cc/). By default it is a responsive page; open it with `?cover` (`concept.html?cover`) for the fixed 16:9 frame and capture it at 1920 × 1080.
