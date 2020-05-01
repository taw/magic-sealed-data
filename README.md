This repository contains machine readable Sealed data generated from:

* https://github.com/taw/magic-search-engine

### Set codes and card numbers

Cards are identified by set code, card number, and foil status.

For overwmelming majority of cards, [mtg.wtf](https://mtg.wtf/), [Gatherer](https://gatherer.wizards.com/Pages/Default.aspx), [mtgjson](https://mtgjson.com/), and [scryfall](http://scryfall.com/) have identical set codes and card numbers.

In case of disagreements, sets codes and card numbers follow [mtg.wtf](https://mtg.wtf/) system, but such disagreements are very rare.

Set codes are all lowercase.

Numbers as printed on the physical card are sometimes not unique (like Unstable variants), and tend to get appropriate suffix to ensure their uniqueness.

For cards with multiple parts (split, DFCs etc.) whichever one is designated as "primary" is used to represent the whole card.

### Data Structure

There are two files, `sealed_basic_data.json` and `sealed_extended_data.json`.

Each file contains list of all known boosters.

Every element of that list represents one booster pack you can get - and has some metadata field, `boosters` field explaining all combinations of how many cards from each sheet you can get, and `sheets` field explaining contents of each possible sheet.

### Metadata

There are following metadata fields:

* `name` - name of the booster
* `code` - code of the booster
* `set_code` - set code of set booster is for
* `set_name` - set name of set booster is for

For regular boosters, `name` and `set_name` are the same; and so are `code` and `set_code`.

The format supports booster variants, where there were multiple booster types for same set. For example for Alara Premium Foil Booster metadata looks like this:

* `"name": "Alara Premium Foil Booster"`
* `"code": "ala-premium"`
* `"set_code": "ala"`
* `"set_name": "Shards of Alara"`

### `boosters` field

`boosters` is an unordered list of possible booster contents.

Each possibility has a weight, and a map of how many cards from each sheet should be taken.

For example for M10, there are two possibilities:

```
    "boosters": [
      {
        "sheets": {
          "m10_basic": 1,
          "m10_common": 10,
          "m10_uncommon": 3,
          "m10_rare_mythic": 1
        },
        "weight": 3
      },
      {
        "sheets": {
          "m10_basic": 1,
          "m10_common": 9,
          "m10_uncommon": 3,
          "m10_rare_mythic": 1,
          "m10_foil": 1
        },
        "weight": 1
      }
    ]
```

* 3/4 of boosters are 1 basic, 10 commons, 3 uncommons, and 1 rare/mythic
* 1/4 of boosters are 1 basic, 9 commons, 3 uncommons, 1 rare/mythic, and 1 foil

It is implied that if multiple cards from same sheet are requested, the algorithm will never pick the same card multiple times, regardless of their relative weights.

### `sheets` field in `sealed_basic_data.json`

`sheets` field is unordered list of sheets from which cards are taken. Roughly one per rarity.

A sheet has code as key (as referenced by `boosters` field), `cards` field listing every possible card that could go into it with their relative probabilities, and `total_weight` adding those weights up for easier processing.

For a very simple sheet like Unhinged basics it looks like this:

```
"unh_basic": {
  "total_weight": 5,
  "cards": {
    "unh:136": 1,
    "unh:137": 1,
    "unh:138": 1,
    "unh:139": 1,
    "unh:140": 1
  }
}
```

It means, when a booster is specified to take a card from `unh_basic` sheet, any of `unh:136`, `unh:137`, `unh:138`, `unh:139`, or `unh:140` are equally possible.

Order of cards on this list doesn't matter.

Every card is indicated by `<set_code>:<card_number>` (like `thb:100`) or `<set_code>:<card_number>:foil` (like `thb:100:foil`).

Sheet can also have optional `"balance_colors": true`, which recommends using color-balancing algorithm to pick cards.

Weights are typically very low, but for some sheets they can be very high. This is especially true for foil sheets (which aren't reals physical sheets, just our estimated ratios of different kinds of foil cards) and software should be able to handle `total_weight` in millions.

### `sheets` field in `sealed_extended_data.json`

The information is the same, but format indicating each card is extended to include mtgjson `uuid`.

For a simple sheet like Unhinged basics again:

```
"unh_basic": {
  "total_weight": 5,
  "cards": [
    {
      "set": "unh",
      "number": "136",
      "weight": 1,
      "foil": false,
      "uuid": "63171a1b-4dfa-50ec-847b-aa5f664deadc"
    },
    {
      "set": "unh",
      "number": "137",
      "weight": 1,
      "foil": false,
      "uuid": "d09f95e6-9de6-53be-83e9-6465173f244e"
    },
    {
      "set": "unh",
      "number": "138",
      "weight": 1,
      "foil": false,
      "uuid": "d260e03d-b5a3-5439-a005-bbaf2bab3e69"
    },
    {
      "set": "unh",
      "number": "139",
      "weight": 1,
      "foil": false,
      "uuid": "98bbffb8-73d5-5234-8c3e-43cca73f2b64"
    },
    {
      "set": "unh",
      "number": "140",
      "weight": 1,
      "foil": false,
      "uuid": "1ffcb8fa-afc7-599f-b5aa-c4c92d032bf0"
    }
  ]
}
```

`cards` instead of being a map from specially formatted card identifier to weight, is not an unordered list of cards, each with `set`, `number`, `weight`, `foil`, and `uuid` fields.

If you use mtgjson, matching by `uuid` and by `set`/`number` should match the same card, but `uuid` is likely to cause fewer surprises.

### Recommended Algorithms

For picking booster version, just pick proportional to its weight.

For picking a single card from a sheet, just pick proportional to its weight.

For picking multiple cards from a sheet, avoid duplicates. Rerolling a card when you get duplicate is the easiest way.

This is not quite mathematically correct if cards have different weights, but errors are very small.

There's no need to run any kind of deduplication between sheets.

If the sheet specifies color balancing, it is recommended to pick one card of each color, and then pick the rest at random, again rerolling card in case of a duplicate.

However, for this to work, weight of cards need to be significantly adjusted so expected values match. If you do it naively, colorless cards will be picked at much lower.

This adjustment is currently outside scope of this project, but at some point it might get incorporated.

### Limitations

Actual boosters don't use fully random process, and instead use carefully created manual collation for improved draft balance. As details are generally unknown (except to extent they are reverse engineered), this isn't simulated.

Many ratios such as foil rarities are only approximately known.

Non-playable cards like tokens, ad cards, and checklist cards are not included.

The data is in many case close approximation, and currently ignores issues like 101th common.

Report any bugs or suggestions as [github issues](https://github.com/taw/magic-sealed-data/issues).

For most extensive reference, check [The Collation Project](http://www.lethe.xyz/mtg/collation/).
