# Repo Setup Design — Proyek UTS Grafkom (Super Bomberman R)

## Goal

One GitHub repo where Fabio (Magnet Bomber), Kevin (White Bomber) and Nick (Golem Bomber)
each build their own character, and all three can be combined into one final scene
without merge problems.

This spec covers **only the shared setup** (folders, shared engine, starter templates,
Git workflow). Each character's actual modeling and animation gets its own spec later.

## Why merges usually fail (what this design prevents)

- Everyone has their own `main()`, canvas, shader and render loop → can't be combined.
- Everyone has their own modified copy of `LIBS` → conflicting versions.
- Same global names (`GL`, `animate`, `CUBE_VERTEX`) → overwrite each other.
- Different sizes / facing directions → characters don't fit together.

## Decisions

| Topic | Decision |
|---|---|
| Final output | One combined scene (`index.html`) with all 3 characters |
| How to run | Everyone uses **Live Server** (VS Code). Code uses ES modules (`import`/`export`), like the class's `main_live.js` |
| Git | Each person works on their own branch and merges into `main` via Pull Request |
| Shared code | Engine only (matrix math, shader, render loop, `Objek` class). **Each person writes their own quadric and curve generators** in their own folder |
| Style | Stay close to the class code so every part can be explained to the asisten |

## Folder structure

```
bomberman-grafkom/
├── index.html            final scene (all 3 characters)
├── main.js               final scene: imports the 3 characters and places them
├── README.md             rules + Git guide for the group
├── shared/               common engine — do not edit without a group-agreed PR
│   ├── libs.js           class LIBS + multiply + scale
│   ├── objek.js          class Objek (one shape, can have children)
│   └── engine.js         canvas, WebGL context, shader, camera, floor, animate loop
├── fabio/
│   ├── magnetBomber.js   buatMagnetBomber(GL) + animasiMagnetBomber(time)
│   └── preview.html      shows only Magnet Bomber
├── kevin/
│   ├── whiteBomber.js    buatWhiteBomber(GL) + animasiWhiteBomber(time)
│   └── preview.html
├── nick/
│   ├── golemBomber.js    buatGolemBomber(GL) + animasiGolemBomber(time)
│   └── preview.html
└── docs/specs/           design documents
```

## Shared code

### `shared/libs.js`
The `LIBS` object from class, unchanged, exported as a module. Added:
- `multiply(a, b)` — 4×4 matrix multiply (same column-major layout as class matrices),
  needed so a child (hand) follows its parent (arm).
- `scale(m, x, y, z)` — scale a matrix.

### `shared/objek.js` — class `Objek`
One shape on screen. Wraps the buffer/draw lines everyone already wrote in class.

- `new Objek(GL, vertex, faces, color)`
  - `vertex`: flat array `[x, y, z, x, y, z, ...]` (same as class)
  - `faces`: flat index array (same as class, `drawElements`)
  - `color`: `[r, g, b]` — one colour per shape
- `MOVEMATRIX` — position/rotation/scale **relative to its parent** (like `main_auto.js`).
- `childs` + `addChild(objek)` — attached shapes follow this shape.
- `render(parentMatrix)` — final matrix = parent × own `MOVEMATRIX`; draws itself, then its childs.
- **Normals are computed automatically** from the faces inside `Objek`, so students only
  supply positions + faces but still get lighting.
- Faces use `Uint16Array` like class, so one `Objek` can have at most 65,535 vertices
  (far more than any body part needs).

### `shared/engine.js`
Written once, used by every preview page and the final scene. Same steps as the class
template:
- get canvas + WebGL context
- one shader: `position` + `normal` attributes, `Pmatrix/Vmatrix/Mmatrix`, `uColor`
  uniform, simple ambient + diffuse light (from the dragon example)
- camera: mouse drag rotation like `pertemuan6` (THETA/PHI with amortization)
- a flat grey floor at y = 0
- the `animate(time)` loop with depth test

Exported function:
```js
mulaiScene(function (GL) {
    // build objects here
    return {
        objek: [ /* root Objek list */ ],
        animasi: function (time) { /* update MOVEMATRIX values */ }
    };
});
```

## Character rules (go in README)

1. Export exactly two functions: `buat<Name>(GL)` returns the root `Objek`, and
   `animasi<Name>(time)` updates the animation.
2. Feet on the ground at **y = 0**, about **2 units tall** (Golem may be about 3), facing
   the camera (**+Z**).
3. Only edit your own folder. No own canvas, shader, or `requestAnimationFrame`.
   No global variables (module-level variables inside your file are fine).
4. Your own quadric/curve generator functions live in your own folder.
5. Your character may walk around, but stay near your spot.

## How the final scene places characters

`main.js` creates one empty "tempat" `Objek` per character at x = -3, 0, 3 and adds each
character's root as its child. A character's own movement (walking) is therefore relative
to its spot, and nobody has to coordinate positions.

## Starter templates

Each person's folder starts with a working placeholder: a simple ball body with two
legs built from a sample shape function, plus a small swinging animation. It shows the
whole pattern (make vertex/faces → `Objek` → `addChild` → animate `MOVEMATRIX`) and is
meant to be replaced. The preview page works immediately.

## Git workflow (go in README)

- `main` always works. Nobody pushes directly to `main`.
- Branches: `fabio`, `kevin`, `nick`. Work and commit there, open a PR into `main`.
- Since everyone only touches their own folder, PRs do not conflict.
- Changes to `shared/` go in a separate PR that the whole group agrees to.

## Testing

- Open `fabio/preview.html`, `kevin/preview.html`, `nick/preview.html` with Live Server:
  each shows its placeholder character on the floor, animated, rotatable by mouse.
- Open `index.html`: all three placeholders side by side.
- Browser console has no errors.

## Out of scope

- Each character's real model and animation (separate specs).
- Arena decoration, textures, sounds.
