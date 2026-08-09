# images

Thumbnails for GTA V / FiveM, captured against a greenscreen and cut out to
transparent WebP.

**7756 images — 94 MB total, ~12 KB each.**

Five trees:

| tree       | what                                                        | count |
| ---------- | ----------------------------------------------------------- | ----- |
| `clothing` | garments and worn accessories                               | 4581  |
| `barber`   | hair, overlays (face and skin), eye colours                 | 806   |
| `tattoos`  | tattoo patterns, on the body zone they belong to            | 1593  |
| `vehicles` | every vehicle, three-quarter view, by spawn name            | 730   |
| `parents`  | the head-blend parents                                      | 46    |

---

## URL pattern

The repository is served through [jsDelivr](https://www.jsdelivr.com/), a free
CDN. Nothing to install, nothing to pay, no bandwidth cap.

```
https://cdn.jsdelivr.net/gh/Metkan-ms/images@1.6.0/<tree>/<gender>/<bucket>/<index>.webp
```

| part     | values                                                     |
| -------- | ---------------------------------------------------------- |
| `tree`   | `clothing` · `barber`                                      |
| `gender` | `male` · `female`                                          |
| `bucket` | see the tables below — a component, prop or overlay id     |
| `index`  | the drawable or style number, `0`-based                    |

`vehicles`, `tattoos` and `parents` are addressed by name instead; all three
are described below.

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

### `tattoos` — addressed by name

```
.../tattoos/male/MP_MP_Biker_Tat_003_M.webp
.../tattoos/female/MP_Bea_F_Neck_000.webp
```

`<gender>/<overlay name>.webp`. No bucket, because the name is already the
address: all 1593 are unique, and none appears under two collections.

**The collection is deliberately not in the path.** It looks like the obvious
folder and it does not work: catalogue files spell the same collection two
ways — `mpbeach_overlays` next to `mpBeach_overlays`, 31 strings for 22 real
collections — and GitHub serves paths case-sensitively. Half the requests
would 404 on the spelling alone.

Nor is the body zone in the path, though every shot has one. A server owner
can move a pattern to another zone in their own catalogue; the photograph is
still of the same tattoo, and an address that encoded the zone would break.

### `vehicles` — addressed by spawn name

```
.../vehicles/adder.webp
.../vehicles/police.webp
.../vehicles/bmx.webp
```

Flat, and **lowercase, always**. The spawn name is the address because it is
already the key everywhere else: it is what `GetHashKey` is fed, what
`vehicles.model` holds in ESX, and what an addon declares in its `.meta`.

The capture writes what the `.meta` spells — `BMX.png`, `Vigero3.png`,
`Dynasty.png`. GitHub serves paths **case-sensitively**, so those are stored
lowercased and nothing else. Lowercase the model before building the URL and
it always resolves:

```js
const src = `${CDN}/vehicles/${model.toLowerCase()}.webp`
```

No category in the path, for the same reason the tattoo collections are not in
theirs: a server owner moves a car from `sedans` to `sports` in their own
catalogue, and the photograph is still of the same car.

**The glass is de-keyed.** The capture removes the background by flooding in
from the border, so anything the car *encloses* is never reached — 567 of the
730 came out with a bright green windscreen. Those regions are keyed
separately and replaced with a dark tint at partial alpha rather than made
transparent: these land on cards, and a card can be dark or light. Tinted
glass reads as glass on both; a hole reads as a hole on the light one.

Addon vehicles are not here, and cannot be: the set is what the base game
ships. Shoot your own with the same greenscreener and drop them in — the
address is just the spawn name.

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

Always request a **tag** (`@1.5.0`), never a branch (`@main`).

A tag is immutable, so jsDelivr caches it forever and a request never has to
go back to GitHub. A branch is revalidated every 12 hours, which is slower and
means a push can change what your customers see without warning.

Changing images means a new tag. `@1.0.0` still exists and still holds the
clothing on its own — an old pin keeps working rather than silently changing.

That matters more since `1.4.0`, which **replaced** every clothing image
rather than adding to them. `@1.3.1` still serves the older framing, so a
server that preferred it can stay there.

---

## `index.json`

Lists what actually exists, so the UI never requests a 404.

```json
{
  "clothing": { "male": { "11": [0, 1, 2, ...] }, "female": { ... } },
  "barber":   { "male": { "component_2": [0, 1, ...] }, "female": { ... } },
  "tattoos":  { "male": ["MP_MP_Biker_Tat_003_M", ...], "female": [ ... ] },
  "vehicles": ["adder", "airbus", "airtug", ...]
}
```

`tattoos` and `vehicles` are flat lists of names rather than buckets of
numbers, matching how those trees are addressed. `vehicles` has no gender
level either, for the obvious reason.

```js
const index = await (await fetch(`${CDN}/index.json`)).json()
const jackets = index.clothing.male['11']       // every male jacket
const hair = index.barber.female['component_2'] // every female hairstyle
const inked = new Set(index.tattoos.male)       // which patterns have a photo
const shot = new Set(index.vehicles)            // which models have a photo
```

Fetch it once at startup and keep it. It is 59 KB.

`vehicles` is what makes an addon car easy to handle: it is not in the list, so
a UI knows to draw its own placeholder rather than fire a request that 404s.

`parents` is not in it — see `parents.json`, which carries the order the
engine imposes and would lose its meaning sorted in here.

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

## `tattoos` — what is covered

1593 patterns, 842 male and 751 female. Each is shot on the body zone it
belongs to, with the clothing that would hide it removed.

| zone             | shots |
| ---------------- | ----- |
| torso            | 711   |
| right arm        | 257   |
| left arm         | 228   |
| head and neck    | 154   |
| left leg         | 134   |
| right leg        | 109   |
| hair             | 0     |

Every one of the 1593 has a visible subject — none came out empty, unlike
parts of the clothing set.

**The hair zone has no photographs.** Those patterns are hairline overlays
(`FM_M_Hair_003_a` and the like), and the capture pass skipped them. A UI that
asks for one gets a 404 and should fall back to whatever it draws for a
missing image.

A pattern is filed under the gender it renders on. Many exist for one gender
only, which is why the two counts differ.

---

## Notes on the images

- **Transparent background.** Put them on whatever colour your UI uses.
- **Cropped to the item**, so sizes differ. A watch is small, a coat is tall.
  Fit them in a fixed box with `object-fit: contain`, do not stretch.
- **Max 320 px wide.** Enough for a shop tile; the lossless PNG masters are
  kept outside this repository if a larger size is ever needed.
- **One texture per drawable.** The first variant is captured; recolours of
  the same garment are not separate images.

### The garment alone, since `1.4.0`

`clothing` was re-shot. The old set framed the **model wearing** the item, so
a hat came with a head attached and a pair of glasses sat somewhere inside a
517 px portrait. The new one crops to the garment itself: the same glasses
are now 60 px tall and fill their tile.

3182 of the 4581 came back with a noticeably tighter frame. Nothing moved
index, so a drawable number still means the same garment it did before.

### Empty drawables

240 of the 4581 have little or nothing in them — 146 are an empty frame, 94
are under 40 px on their long edge. These are drawables that render nothing
visible on the model: an index the game accepts and draws no geometry for.

They are kept so the numbering stays contiguous, and because the index is
what a UI looks a garment up by.

The previous set appeared to have only four of these, which was an artefact
of how it was framed rather than a difference in the game: it photographed
the whole model, so a drawable that drew nothing still produced a picture of
a bare head. 109 of those "images" were the same bare head saved under
different names. Cropping to the subject is what makes an empty drawable
look empty.

---

## Licence

The images are renders of Rockstar Games assets and are published here for use
with GTA V / FiveM servers only. Rockstar retains all rights to the underlying
artwork.
