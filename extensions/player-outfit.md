# Player Outfit

A cosmetic outfit for each player.

| ------------: | ------------- |
| Extension ID: | `0x35`        |
| Packet ID:    | `0x75`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

## Outfit

An outfit changes how a player looks, and nothing else. It is drawn like the
normal player: solid voxels on the same body parts, keeping the player's outline,
with the held tool or weapon drawn as it is. A client draws an unknown outfit, or
one it has no art for, as `0`.

An outfit may have its own sounds. A client that gives one outfit its own sounds
gives every outfit theirs.

| Value  | Name      | Look                                |
|--------|-----------|-------------------------------------|
| 0      | Soldier   | The normal player model.            |
| 1      | Undead    | Rotting skin, torn uniform.         |
| 2      | Scout     | Light kit, no helmet.               |
| 3      | Royal     | Crown, fur-trimmed tunic.           |
| 4      | Vampire   | Pale skin, high-collared coat.      |
| 5      | Miner     | Hard hat with a lamp, dusty.        |
| 6      | Ghillie   | Camouflage suit.                    |
| 7      | Brawler   | Bare arms, headband.                |
| 8      | Scientist | Lab coat, goggles.                  |
| 9      | Butcher   | Bloodied apron, rubber boots.       |
| 10     | Convict   | Striped prison uniform.             |
| 11     | Builder   | High-visibility vest, tool belt.    |
| 12     | Robot     | Metal plating, visor.               |
| 13     | God       | White toga, laurel wreath.          |
| 14     | Demon     | Red skin, small horns.              |
| 15     | Bodyguard | Dark suit, sunglasses, earpiece.    |
| 16     | Terrorist | Balaclava, chest rig.               |
| 17-255 | reserved  | Drawn as `0`.                       |

## Sub ID 0: Set Outfit

Sent before the [Create Player](../protocol075.md#create-player) or
[Existing Player](../protocol075.md#existing-player) that introduces the player,
and at any time after.

| Field Name    | Field Type | Example | Notes                           |
|---------------|------------|---------|---------------------------------|
| Packet ID     | UByte      | `0x75`  | Always `0x75`.                  |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet. |
| Player ID     | UByte      | `254`   |                                 |
| Outfit        | UByte      | `1`     | See [Outfit](#outfit).          |

The outfit belongs to the player id.
[Player Left](../protocol075.md#player-left) resets it to `0`, and
[Map Start](../protocol075.md#map-start-075) resets every id.

See [Extensions](extension.md) for how the extension is negotiated.
