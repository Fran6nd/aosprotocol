# Player Outfit

A cosmetic outfit for each player.

| ------------: | ------------- |
| Extension ID: | `0x35`        |
| Packet ID:    | `0x75`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

### Sub Packets:

| Sub ID | Name         | Direction        | Size          |
|--------|--------------|------------------|---------------|
| 0      | Set Outfit   | Server -> Client | 4             |
| 1      | Team Outfits | Server -> Client | `2+2*entries` |

## Outfit

An outfit changes how a player looks, and nothing else: hits, collision and
everything else are those of the normal player. An outfit is drawn in solid
voxels and shows the player's team colour where it can be seen from a distance,
unless its page says otherwise. The held tool or weapon is drawn as it is, unless
the page says otherwise.

An outfit may have its own sounds. A client that gives one outfit its own sounds
gives every outfit theirs.

Outfits come in pages of 16. The first outfit of each page is its generic one. A
client draws an outfit it has no art for as its page's generic, and a generic it
has no art for as `0`. A client can support a page by drawing only its generic.

### Page 0: Humans

| Value | Name      | Look                              |
|-------|-----------|-----------------------------------|
| 0     | Soldier   | The normal player model.          |
| 1     | Scout     | Light kit, no helmet.             |
| 2     | Ghillie   | Camouflage suit.                  |
| 3     | Bodyguard | Dark suit, sunglasses, earpiece.  |
| 4     | Terrorist | Balaclava, chest rig.             |
| 5     | Convict   | Striped prison uniform.           |
| 6     | Brawler   | Bare arms, headband.              |
| 7     | Miner     | Hard hat with a lamp, dusty.      |
| 8     | Builder   | High-visibility vest, tool belt.  |
| 9     | Scientist | Lab coat, goggles.                |
| 10    | Butcher   | Bloodied apron, rubber boots.     |
| 11    | Royal     | Crown, fur-trimmed tunic.         |
| 12    | Robot     | Metal plating, visor.             |

### Page 1: Supernatural

| Value | Name    | Look                                    |
|-------|---------|-----------------------------------------|
| 16    | Undead  | Rotting skin, torn uniform.             |
| 17    | Vampire | Pale skin, high-collared coat.          |
| 18    | Mummy   | Wrapped in bandages.                    |
| 19    | Demon   | Red skin, small horns.                  |
| 20    | God     | White toga, laurel wreath.              |

### Page 2: Costumes

| Value | Name    | Look                                       |
|-------|---------|--------------------------------------------|
| 32    | Mascot  | A plush full-body suit with a big head.    |
| 33    | Tiger   | Orange with black stripes.                 |
| 34    | Penguin | Black and white, orange beak and feet.     |
| 35    | Bear    | Brown fur, round ears.                     |
| 36    | Unicorn | White, a horn and a rainbow mane.          |
| 37    | T-rex   | Green scales, small arms, big jaw.         |
| 38    | Banana  | A yellow banana with the face cut out.     |

### Page 3: Turrets

A turret stands on a tripod in place of the body, and its barrel points where the
player aims. It is not drawn holding a tool or weapon.

| Value | Name             | Look                                   |
|-------|------------------|----------------------------------------|
| 48    | Turret           | A gun on a tripod.                     |
| 49    | Rifle Turret     | A rifle on a tripod.                   |
| 50    | SMG Turret       | A short gun with a drum magazine.      |
| 51    | Sniper Turret    | A long barrel with a scope.            |
| 52    | Grenade Turret   | A wide tube with a grenade belt.       |

### Page 4: Disguises

A disguise is drawn without the team colour and without a tool or weapon.

| Value | Name        | Look                                                         |
|-------|-------------|--------------------------------------------------------------|
| 64    | Block Stack | Three blocks stacked, in the player's held block colour.     |

### Reserved

Every value from `0` to `254` not listed above is reserved and drawn as its
page's generic. `255` is the player's [team outfit](#sub-id-1-team-outfits).

## Sub ID 0: Set Outfit

| Field Name    | Field Type | Example | Notes                           |
|---------------|------------|---------|---------------------------------|
| Packet ID     | UByte      | `0x75`  | Always `0x75`.                  |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet. |
| Player ID     | UByte      | `254`   |                                 |
| Outfit        | UByte      | `1`     | See [Outfit](#outfit).          |

The outfit belongs to the player id and takes effect when it arrives. A player
with no Set Outfit wears `255`. The server sends a player's Set Outfit before the
[Create Player](../protocol075.md#create-player) or
[Existing Player](../protocol075.md#existing-player) that introduces them, so a
client draws a player in the outfit it holds when they are created and never
waits for one.

[Player Left](../protocol075.md#player-left) resets the id to `255`, and
[Map Start](../protocol075.md#map-start-075) resets every id. The server sends
Existing Player for every player after Map Start anyway, and sends their Set
Outfit with it.

## Sub ID 1: Team Outfits

The outfit of each team's players whose outfit is `255`. The server sends it
before the Create Player or Existing Player that introduces the first player, and
may send it again at any time.

| Field Name    | Field Type        | Example | Notes                           |
|---------------|-------------------|---------|---------------------------------|
| Packet ID     | UByte             | `0x75`  | Always `0x75`.                  |
| Sub Packet ID | UByte             | `1`     | Always `1` for this sub-packet. |
| Entries       | TeamOutfitEntry[] |         | To the end of the packet.       |

**TeamOutfitEntry** (2 bytes)

| Field Name | Field Type | Example | Notes                                |
|------------|------------|---------|--------------------------------------|
| Team ID    | UByte      | `1`     | `0` or `1`; other teams are ignored. |
| Outfit     | UByte      | `16`    | `0` to `254`, see [Outfit](#outfit). |

Each Team Outfits replaces the previous one and takes effect when it arrives. A
team it does not list wears `0`. It lasts for the connection.

See [Extensions](extension.md) for how the extension is negotiated.
