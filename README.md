# images

Clothing and accessory thumbnails for GTA V / FiveM, captured against a
greenscreen and cut out to transparent WebP.

**4736 images — 64 MB total, ~14 KB each.**

---

## URL pattern

The repository is served through [jsDelivr](https://www.jsdelivr.com/), a free
CDN. Nothing to install, nothing to pay, no bandwidth cap.

```
https://cdn.jsdelivr.net/gh/Metkan-ms/images@1.0.0/clothing/<gender>/<bucket>/<drawable>.webp
```

| part       | values                                            |
| ---------- | ------------------------------------------------- |
| `gender`   | `male` · `female`                                 |
| `bucket`   | a component id (`11`) or a prop id (`prop_0`)     |
| `drawable` | the drawable number, `0`-based                    |

Examples:

```
.../clothing/male/11/17.webp        male jacket, drawable 17
.../clothing/female/4/8.webp        female trousers, drawable 8
.../clothing/male/prop_0/3.webp     male hat, prop 0, drawable 3
```

### Pin the version

Always request a **tag** (`@1.0.0`), never a branch (`@main`).

A tag is immutable, so jsDelivr caches it forever and a request never has to
go back to GitHub. A branch is revalidated every 12 hours, which is slower and
means a push can change what your customers see without warning.

---

## `index.json`

Lists the drawables that actually exist, so the UI never requests a 404.

```json
{ "male": { "11": [0, 1, 2, ...], "prop_0": [0, 1, ...] }, "female": { ... } }
```

```js
const index = await (await fetch(`${CDN}/index.json`)).json()
const jackets = index.male['11']        // every male jacket drawable
```

Fetch it once at startup and keep it. It is 16 KB.

---

## Component and prop ids

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

Components `0`, `2` and `10` (head, hair, decals) are not captured — a
thumbnail of them shows a bare head rather than an item.

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
