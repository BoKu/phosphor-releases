# Make your own PHOSPHOR missions

Open Contracts are missions and companies made by players. They live in one JSON file that BoKu publishes on GitHub. The game downloads it by itself. Your missions then show up on the contract board, tagged with your author name.

You do not need to be a programmer. You need a text editor and a little care with commas.

## How to submit

1. Write your missions in a JSON file (see below).
2. Read it over against the tables below.
3. Open an issue on [BoKu/phosphor-releases](https://github.com/BoKu/phosphor-releases/issues) and attach the JSON file.
4. BoKu checks it with the game's own validator and reviews it. If it is good, BoKu adds it to the published pack.

Do not use real company names or real people. Keep everything fictional.

Text may not start or end with a space or contain two spaces in a row (a `brief` may double-space). File names cannot reuse the names of the game's own files.

## The file shape

A pack has three keys.

```json
{ "version": 1, "companies": [], "missions": [] }
```

- `version` must be `1`.
- `companies` holds your own companies (optional, up to 50).
- `missions` holds your missions (up to 200).

The whole file must stay under 1 MB. The game checks every entry on its own. A bad entry is skipped and the rest still load, with these knock-on effects:

- One bad item inside a company skips the whole company. Examples are a duplicate id, an over-long name or a non-whole `mb`.
- Every mission that targets a skipped company is then skipped too. This is the most common mistake: one typo in a company hides all its missions.
- A wrong `version` or invalid JSON drops the whole file.
- An unknown top-level key is only reported as a problem; the rest of the file still loads. An unknown key inside a company or mission skips that entry.

A complete pack that uses every key is in [examples/full-mission.json](examples/full-mission.json).

## Company fields

A company is a server you invent. Every player gets the same one. The game builds its security layers, so you do not write them.

| Field | Type | Required | Allowed values and limits | Notes |
|---|---|---|---|---|
| `id` | text | yes | 1-32 characters: a-z, 0-9, `-` | Becomes `oc-<author>-<id>`. |
| `author` | text | yes | 1-24 characters: a-z, 0-9, `-` | Your name. Shown on the board. |
| `name` | text | yes | 1-40 characters | Must be unique in the pack. Cannot match a game company, and cannot start with a word the game already uses for its own story. |
| `kind` | text | yes | `corp`, `bank`, `lab`, `media`, `telecom`, `datacentre`, `fleet` | Only `bank` may have accounts. Only `fleet` may have vehicles. |
| `city` | text | yes | One of the cities listed below | Exact spelling. |
| `tier` | whole number | yes | 1 to 5 | Higher means more layers and tougher ones. |
| `blurb` | text | no | 1-160 characters | One line about the company. |
| `ceo` | text | no | 1-40 characters | A made-up name. |
| `files` | list | no | At most 10 | See sub-list fields. |
| `records` | list | no | At most 10 | See sub-list fields. |
| `accounts` | list | no | At most 4, banks only | See sub-list fields. A bank always has 4 accounts: yours replace generated ones. |
| `vehicles` | list | no | At most 10, fleets only | See sub-list fields. |

## Sub-list fields

Each entry in a list needs an `id` (1-32 characters: a-z, 0-9, `-`, unique within that list). Missions point at these ids.

| List | Keys | Limits |
|---|---|---|
| files | `id`, `name`, `mb`, optional `enc` | `name` up to 40 characters. `mb` is a whole number of at least 1. The game lowers it to `40 x tier x tier + 60` if larger. `enc: true` means the file is encrypted (the player needs a Decrypter). |
| records | `id`, `subject`, `field`, `value` | Each text is 1-40 characters. |
| accounts | `id`, `owner`, `balance` | `balance` is a whole number from 0. The game lowers it to `400,000 x tier` if larger. |
| vehicles | `id`, `model`, `driver`, `role`, optional `plate` | Texts up to 40 characters. `plate` up to 12. |

## Mission fields

| Field | Type | Required | Allowed values and limits | Notes |
|---|---|---|---|---|
| `id` | text | yes | 1-32 characters: a-z, 0-9, `-` | Becomes `oc-<author>-<id>`. Must be unique. |
| `author` | text | yes | 1-24 characters: a-z, 0-9, `-` | Shown as the client. |
| `kind` | text | yes | `copy`, `delete`, `edit`, `transfer`, `track`, `vehicle`, `relay`, `leak` | See "What each kind uses". |
| `target` | object | yes | `{ "pack": "<company id>" }` or `{ "company": "<game company name>" }` | Exactly one of the two. |
| `title` | text | yes | 1-80 characters | Shown on the board. |
| `brief` | text | yes | 1-400 characters | The client's message. |
| `pay` | whole number | yes | At least 1 | Credits at Runner difficulty. Rounded to 10 and lowered to the cap below. |
| `requires` | object | yes | Tool id to version, from 1 up to the tool's top version (max 5) | Must beat every layer of a pack target (and of a `relay` destination). Players only see the mission once they own every listed version. See "Tools against layers". |
| `tier` | whole number | no | 1 to 5 | Must equal the target's tier. The game uses the target's tier anyway. |
| `wipe` | true/false | no | Not for `relay` or `leak` | Whether the player must also wipe the logs. |
| `days` | whole number | no | 3 to 14 | Days to finish. Default is 7. |
| `copyTo` | text | no | `gateway` or `vault`; `copy` only | Where the copy must go. |
| `file` | text | no | A file id on the target; `copy`, `delete`, `relay`, `leak` | `relay` and `leak` files cannot be encrypted. |
| `record` | text | no | A record id on the target; `edit` only | The record to change. |
| `newValue` | text | no | 1-40 characters; `edit` only, and only with `record` | The value the record must end up with. |
| `account` | text | no | An account id on the target; `transfer` only | The account to drain. |
| `amount` | whole number | no | At least 1; `transfer` only | The game lowers it to `40,000 x tier` and to the account's balance. |
| `vehicle` | text | no | A vehicle id on the target; `track` and `vehicle` only | The vehicle to follow or hijack. |
| `action` | text | no | `stall` or `brakes`; `vehicle` only | What to do to the vehicle. |
| `to` | object | no | Same shape as `target`; `relay` only | Must differ from `target`. Its tier can be at most one above the target's. |

Keys that do not belong to a kind make the mission invalid. Leave them out.

If the target is a game company, you cannot name `file`, `record`, `account` or `vehicle`. Those exist only on your own companies. The game picks something at random for any detail you leave out.

## What each kind uses

| Kind | Target must be | Uses |
|---|---|---|
| copy | any | `file`, `copyTo`, `wipe` |
| delete | any | `file`, `wipe` |
| edit | A pack company with records, or a game company of kind corp, bank or media | `record`, `newValue`, `wipe` |
| transfer | A bank | `account`, `amount`, `wipe` |
| track | A fleet | `vehicle`, `wipe` |
| vehicle | A fleet | `vehicle`, `action`, `wipe` |
| relay | any | `file`, `to` |
| leak | any | `file` |

`track` and `vehicle` contracts only appear for players who own a Skyloom link, as on the normal board.

## Maximum pay

Your `pay` is lowered to these caps (credits, at Runner difficulty, by target tier). The table is Runner, the normal difficulty. Tourist and Ghost scale pay by their own multipliers, and the cap scales with them.

| Kind | Tier 1 | Tier 2 | Tier 3 | Tier 4 | Tier 5 |
|---|---|---|---|---|---|
| copy | 480 | 810 | 1370 | 2330 | 3970 |
| delete | 430 | 720 | 1230 | 2090 | 3550 |
| edit | 540 | 910 | 1550 | 2640 | 4490 |
| transfer | 700 | 1190 | 2020 | 3440 | 5850 |
| track | 450 | 770 | 1300 | 2210 | 3760 |
| vehicle | 980 | 1660 | 2820 | 4790 | 8140 |
| relay | 880 | 1490 | 2530 | 4300 | 7310 |
| leak | 650 | 1110 | 1880 | 3190 | 5430 |

## Tools against layers

Each layer on a pack company has a level. Version `v` of a tool beats layers up to level `v x 2`. Your `requires` must beat every layer of a pack target. Tier `t` servers have layers up to level `t x 2`, so version `t` of each layer tool is enough.

| Layer | Tool id | Tool name |
|---|---|---|
| proxy | `proxy` | Proxy Sidestep |
| firewall | `firewall` | Firewall Disabler |
| password | `password` | Dictation Cracker |
| twofa | `twofa` | Token Spoofer |
| ids | `ids` | Monitor Mute |

Other tools you can require, for the job itself. Use `telematics` for `track` and `vehicle` (version 2 for stall and brakes).

| Tool id | Top version |
|---|---|
| `probe` | 3 |
| `tracker` | 1 |
| `decrypt` | 5 |
| `copy` | 1 |
| `delete` | 1 |
| `edit` | 1 |
| `transfer` | 3 |
| `logwipe` | 5 |
| `sftp` | 1 |
| `tor` | 3 |
| `telematics` | 3 |
| `node` | 4 |

The five layer tools above all top out at version 5.

The game companies have layers that differ from save to save. BoKu's check warns when your tools might not beat one of them. Such a mission is skipped in saves where they cannot.

## The 20 game companies

Name one in `target` by its exact name.

| Company | Kind | Tier |
|---|---|---|
| Pellam Freight | corp | 1 |
| Dunmore Packaging | corp | 1 |
| Tarn Savings | bank | 1 |
| Skerry Mobile | telecom | 1 |
| Ravenscar Mining | corp | 2 |
| Fennimore Pharma | lab | 2 |
| Brasswick Broadcasting | media | 2 |
| Underhill Compute | datacentre | 2 |
| Swiftcar Mobility | fleet | 2 |
| Eastlake Shipping | corp | 3 |
| Wexcombe Merchant Bank | bank | 3 |
| Ardent Materials | lab | 3 |
| Ashgrove Data Halls | datacentre | 3 |
| Quillon Genomics | lab | 4 |
| Loudwater Press | media | 4 |
| Tallis Networks | telecom | 4 |
| Orchard Rideshare | fleet | 4 |
| Corvane Aerospace | corp | 5 |
| Ostrander Reserve | bank | 5 |
| Blackfen Compute | datacentre | 5 |

## Cities

New York, Los Angeles, Chicago, Toronto, Mexico City, São Paulo, Buenos Aires, Santiago, London, Dublin, Paris, Berlin, Amsterdam, Madrid, Stockholm, Warsaw, Reykjavik, Lagos, Cairo, Nairobi, Cape Town, Moscow, Istanbul, Dubai, Mumbai, Bangalore, Singapore, Jakarta, Hong Kong, Shenzhen, Seoul, Tokyo, Sydney, Perth, Auckland.

## Ids and the board

The game adds your author name in front of every id: mission `proofs` by `demo` becomes `oc-demo-proofs`. Ids must be unique in the pack. In `target` and `to`, a short id refers to a company by the same author as the mission. To use another author's company, write its full `oc-` id.

- At most 2 Open Contracts are open on a player's board at once.
- The board only offers a mission to a player who owns every tool version in `requires`.
- The board only offers a mission when the player's rating allows its tier: tier 2 needs 20, tier 3 needs 70, tier 4 needs 160, tier 5 needs 400. Tier 1 has no gate.
- One and done: once a player completes a mission, it never comes back.
- If a player fails it or lets it expire, it can return after 7 days.
- Frozen once seen: when a mission first reaches a player's board, that copy is fixed. Later edits do not change it. To change a mission, publish it under a new id.
- Rival crews never compete for Open Contracts.
- Your companies stay in a save once added.

## One short example per kind

These use the companies in the full example.

```json
{ "id": "proofs", "author": "demo", "kind": "copy", "target": { "pack": "lumen-print" }, "title": "Proof of print", "brief": "Copy the leaflet proofs.", "pay": 450, "requires": { "proxy": 1, "firewall": 1, "password": 1 }, "file": "leaflets", "copyTo": "vault" }
```

```json
{ "id": "shred", "author": "demo", "kind": "delete", "target": { "pack": "lumen-print" }, "title": "Shred the memo", "brief": "Delete the board memo.", "pay": 400, "requires": { "proxy": 1, "firewall": 1, "password": 1 }, "file": "memo" }
```

```json
{ "id": "promote", "author": "demo", "kind": "edit", "target": { "pack": "lumen-print" }, "title": "Quiet promotion", "brief": "Change a visitor's access level.", "pay": 500, "requires": { "proxy": 1, "firewall": 1, "password": 1 }, "record": "badge", "newValue": "staff" }
```

```json
{ "id": "reserve", "author": "demo", "kind": "transfer", "target": { "pack": "harbour-trust" }, "title": "Dry dock", "brief": "Move money out of the reserve.", "pay": 1900, "requires": { "proxy": 3, "firewall": 3, "password": 3, "twofa": 3 }, "account": "reserve", "amount": 60000 }
```

```json
{ "id": "vanwatch", "author": "demo", "kind": "track", "target": { "pack": "tessel-couriers" }, "title": "Where is van one", "brief": "Find the van and watch it.", "pay": 750, "requires": { "proxy": 2, "firewall": 2, "password": 2, "telematics": 1 }, "vehicle": "van1" }
```

```json
{ "id": "brakes", "author": "demo", "kind": "vehicle", "target": { "pack": "tessel-couriers" }, "title": "Emergency stop", "brief": "Stop a van, suddenly.", "pay": 1600, "requires": { "proxy": 2, "firewall": 2, "password": 2, "telematics": 2 }, "vehicle": "van2", "action": "brakes" }
```

```json
{ "id": "handoff", "author": "demo", "kind": "relay", "target": { "pack": "lumen-print" }, "to": { "pack": "tessel-couriers" }, "title": "Hand-off", "brief": "Move the proofs to the Tessel server.", "pay": 800, "requires": { "proxy": 2, "firewall": 2, "password": 2 }, "file": "leaflets" }
```

```json
{ "id": "daylight", "author": "demo", "kind": "leak", "target": { "pack": "harbour-trust" }, "title": "Daylight", "brief": "Take the audit draft and let it out.", "pay": 1700, "requires": { "proxy": 3, "firewall": 3, "password": 3, "twofa": 3 }, "file": "audit" }
```

A mission on a game company needs no company in your pack:

```json
{ "id": "freight", "author": "demo", "kind": "copy", "target": { "company": "Pellam Freight" }, "title": "Barge manifests", "brief": "Copy a file from Pellam Freight.", "pay": 420, "requires": { "proxy": 1, "firewall": 1, "password": 1 } }
```

BoKu runs the game's own check on every submission before publishing, so you do not need to run anything yourself.
