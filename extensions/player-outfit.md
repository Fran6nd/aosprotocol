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
everything else are those of the normal player. It follows the player's stance,
so a crouching player's outfit is drawn crouching, and their corpse wears it. The
first-person view does not change.

An outfit is coloured in the player's team colour where the normal model's
uniform is, unless its row says otherwise. An outfit may hide the held tool or
weapon. A client draws an outfit it has no art for as `0`.

| Value  | Name        | Look                                                                 |
|--------|-------------|----------------------------------------------------------------------|
| 0      | Soldier     | The normal player model.                                             |
| 1      | Scout       | Light kit, no helmet.                                                |
| 2      | Ghillie     | Camouflage suit.                                                     |
| 3      | Bodyguard   | Dark suit, sunglasses, earpiece.                                     |
| 4      | Terrorist   | Balaclava, chest rig.                                                |
| 5      | Convict     | Striped prison uniform.                                              |
| 6      | Brawler     | Bare arms, headband.                                                 |
| 7      | Miner       | Hard hat with a lamp, dusty.                                         |
| 8      | Builder     | High-visibility vest, tool belt.                                     |
| 9      | Scientist   | Lab coat, goggles.                                                   |
| 10     | Butcher     | Bloodied apron, rubber boots.                                        |
| 11     | Royal       | Crown, fur-trimmed tunic.                                            |
| 12     | Robot       | Metal plating, visor.                                                |
| 13     | Undead      | Rotting skin, torn uniform.                                          |
| 14     | Vampire     | Pale skin, high-collared coat.                                       |
| 15     | Mummy       | Wrapped in bandages.                                                 |
| 16     | Demon       | Red skin, small horns.                                               |
| 17     | God         | White toga, laurel wreath.                                           |
| 18     | Mascot      | A plush full-body suit with a big head.                              |
| 19     | Tiger       | Orange with black stripes.                                           |
| 20     | Penguin     | Black and white, orange beak and feet.                               |
| 21     | Bear        | Brown fur, round ears.                                               |
| 22     | Unicorn     | White, a horn and a rainbow mane.                                    |
| 23     | T-rex       | Green scales, small arms, big jaw.                                   |
| 24     | Banana      | A yellow banana with the face cut out.                               |
| 25     | Turret      | The held weapon on a team-coloured tripod, pointing where the player aims. |
| 26     | Block Stack | Three blocks stacked, in the player's held block colour. No team colour, hides the held tool or weapon. |
| 27     | Projector   | A lamp on a team-coloured tripod, pointing where the player aims. Hides the held tool or weapon, and gives no light. |
| 28-254 | reserved    | Drawn as `0`.                                                        |
| 255    | Team        | The player's [team outfit](#sub-id-1-team-outfits).                  |

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
[Map Start](../protocol075.md#map-start-075) resets every id, since the server
introduces every player again after it.

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
team it does not list, or any team before the first Team Outfits, wears `0`. It
lasts for the connection.

See [Extensions](extension.md) for how the extension is negotiated.
