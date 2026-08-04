# images

Thumbnails for GTA V / FiveM, captured against a greenscreen and cut out to
transparent WebP.

**5588 images — 79 MB total, ~14 KB each.**

Three trees:

| tree       | what                                                        | count |
| ---------- | ----------------------------------------------------------- | ----- |
| `clothing` | garments and worn accessories                               | 4736  |
| `barber`   | hair, overlays (face and skin), eye colours                 | 806   |
| `parents`  | the head-blend parents                                      | 46    |

---

## URL pattern

The repository is served through [jsDelivr](https://www.jsdelivr.com/), a free
CDN. Nothing to install, nothing to pay, no bandwidth cap.

```
https://cdn.jsdelivr.net/gh/Metkan-ms/images@1.2.0/<tree>/<gender>/<bucket>/<index>.webp
```

| part     | values                                                     |
| -------- | ---------------------------------------------------------- |
| `tree`   | `clothing` · `barber`                                      |
| `gender` | `male` · `female`                                          |
| `bucket` | see the tables below — a component, prop or overlay id     |
| `index`  | the drawable or style number, `0`-based                    |

Examples:

```
.../clothing/male/11/17.webp             male jacket, drawable 17
.../clothing/female/4/8.webp             female trousers, drawable 8
.../clothing/male/prop_0/3.webp          male hat, prop 0, drawable 3

.../barber/male/component_2/17.webp      male hairstyle 17
.../barber/female/overlay_2/4.webp       female eyebrows, style 4
.../barber/male/eyecolor_0/12.webp       male eye colour 12
```

One rule reads those two trees: `<gender>/<bucket>/<index>.webp`. Only the set
of buckets differs.

### `parents` — the exception

```
.../parents/Hannah.webp                  head-blend parent 21
.../parents/Benjamin.webp                head-blend parent 0
```

Flat, and named after the person rather than numbered. A parent is the same
face whichever ped is being built, so there is no gender to file it under, and
`SetPedHeadBlendData` takes an index into a list the engine fixes:

| index   | who                                                      |
| ------- | -------------------------------------------------------- |
| `0-20`  | the male heads, Benjamin to Anthony                      |
| `21-41` | the female heads, Hannah to Emma                         |
| `42-45` | Misty, Niko, John, Claude                                |

The order is not alphabetical and cannot be sorted: it IS the index the game
expects. See `parents.json` for the list.

### Pin the version

Always request a **tag** (`@1.2.0`), never a branch (`@main`).

A tag is immutable, so jsDelivr caches it forever and a request never has to
go back to GitHub. A branch is revalidated every 12 hours, which is slower and
means a push can change what your customers see without warning.

Adding images means a new tag. `@1.0.0` still exists and still holds the
clothing on its own — an old pin keeps working rather than silently changing.

---

## `index.json`

Lists what actually exists, so the UI never requests a 404.

```json
{
  "clothing": { "male": { "11": [0, 1, 2, ...] }, "female": { ... } },
  "barber":   { "male": { "component_2": [0, 1, ...] }, "female": { ... } }
}
```

```js
const index = await (await fetch(`${CDN}/index.json`)).json()
const jackets = index.clothing.male['11']       // every male jacket
const hair = index.barber.female['component_2'] // every female hairstyle
```

Fetch it once at startup and keep it. It is 19 KB.

> The tree name became the first level in `1.1.0`. In `1.0.0` the genders were
> at the top, with clothing only.

---

## `clothing` — component and prop ids

These are GTA V's own numbering — the same values you pass to
`SetPedComponentVariation` and `SetPedPropIndex`.

**Components** — `SetPedComponentVariation(ped, id, drawable, texture, 0)`

| id   | what                                    |
| ---- | --------------------------------------- |
| `1`  | masks                                   |
| `3`  | torso, arms, gloves                     |
| `4`  | trousers                                |
| `5`  | bags and parachutes                     |
| `6`  | shoes                                   |
| `7`  | neck accessories — chains, ties, scarves |
| `8`  | undershirts                             |
| `9`  | body armour and vests                   |
| `11` | tops and jackets                        |

**Props** — `SetPedPropIndex(ped, id, drawable, texture, true)`

| id       | what      |
| -------- | --------- |
| `prop_0` | hats      |
| `prop_1` | glasses   |
| `prop_2` | earrings  |
| `prop_6` | watches   |
| `prop_7` | bracelets |

Components `0` and `10` (head, decals) are not captured — a thumbnail of them
shows a bare head rather than an item. Component `2` is hair, and it lives in
the `barber` tree rather than here.

---

## `barber` — hair, overlays and eye colours

Three families, told apart by the bucket's prefix.

**`component_2`** — hairstyles. `SetPedComponentVariation(ped, 2, style, …)`

| bucket        | male | female |
| ------------- | ---- | ------ |
| `component_2` | 82   | 86     |

**`overlay_<id>`** — head overlays. `SetPedHeadOverlay(ped, id, style, opacity)`

The ids are GTA's own. Every one of the thirteen that draws something is here.

| bucket       | what                | male | female |
| ------------ | ------------------- | ---- | ------ |
| `overlay_0`  | blemishes, acne     | 24   | 24     |
| `overlay_1`  | facial hair, beards | 29   | —      |
| `overlay_2`  | eyebrows            | 34   | 34     |
| `overlay_3`  | ageing, wrinkles    | 15   | 15     |
| `overlay_4`  | make-up             | 95   | 95     |
| `overlay_5`  | blush               | 33   | 33     |
| `overlay_6`  | complexion          | 12   | 12     |
| `overlay_7`  | sun damage          | 11   | 11     |
| `overlay_8`  | lipstick            | 10   | 10     |
| `overlay_9`  | freckles, moles     | 18   | 18     |
| `overlay_10` | chest hair          | 17   | —      |
| `overlay_11` | body blemishes      | 12   | 12     |

`overlay_10` and `overlay_11` are torso shots; the other eleven are head
shots.

### How well they read

Worth knowing before wiring one to a small tile. Judged on the images
themselves, not guessed at:

| bucket                        | at thumbnail size            |
| ----------------------------- | ---------------------------- |
| `0` blemishes, `3` ageing     | obvious                      |
| `6` complexion, `9` freckles  | clear                        |
| `7` sun damage                | visible, styles look alike   |
| `11` body blemishes           | weak — a whole torso, and the marks are small in it |

**`eyecolor_0`** — eye colours. `SetPedEyeColor(ped, index)`

| bucket       | male | female |
| ------------ | ---- | ------ |
| `eyecolor_0` | 32   | 32     |

The `_0` is there for consistency of shape, not because GTA numbers eye
colours by category. There is only ever one.

### A note on the male-only buckets

`overlay_1` and `overlay_10` have no female counterpart, and that is not an
oversight — GTA offers neither beards nor chest hair on the female model.
Requesting `barber/female/overlay_1/3.webp` returns 404, which is correct.

---

## Notes on the images

- **Transparent background.** Put them on whatever colour your UI uses.
- **Cropped to the item**, so sizes differ. A watch is small, a coat is tall.
  Fit them in a fixed box with `object-fit: contain`, do not stretch.
- **Max 320 px wide.** Enough for a shop tile; the lossless PNG masters are
  kept outside this repository if a larger size is ever needed.
- **One texture per drawable.** The first variant is captured; recolours of
  the same garment are not separate images.

### Four known bad captures

`male/4/44`, `male/4/11`, `female/4/46` and `female/prop_7/18` came out as
small squares. Those drawables render as nothing on the model, so there was
no subject to photograph. They are left in place so the numbering stays
contiguous.

---

## Licence

The images are renders of Rockstar Games assets and are published here for use
with GTA V / FiveM servers only. Rockstar retains all rights to the underlying
artwork.
