# Extended Spawn Packet

Per-player properties the base protocol has no room for: flags that leave a
player out of what other clients report about players, a colour that replaces
their team colour, and a cosmetic outfit.

| ------------: | ------------- |
| Extension ID: | `0x34`        |
| Packet ID:    | `0x74`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

### Sub Packets:

| Sub ID | Name                     | Direction        | Size  |
|--------|--------------------------|------------------|-------|
| 0      | Extended Create Player   | Server -> Client | `22+` |
| 1      | Extended Existing Player | Server -> Client | `18+` |
| 2      | Set Player               | Server -> Client | `8`   |

To a client that negotiated this extension, the server sends sub 0 instead of
[Create Player](../protocol075.md#create-player) and sub 1 instead of
[Existing Player](../protocol075.md#existing-player). Other extensions treat
them as the packets they replace.

## Flags

| Bit | Name            | Meaning                                                                  |
|-----|-----------------|--------------------------------------------------------------------------|
| 0   | `HIDE_ROSTER`   | Left out of the scoreboard, player counts and spectator camera cycling. |
| 1   | `HIDE_PRESENCE` | No join, team change or leave notification.                              |
| 2   | `HIDE_KILLFEED` | No kill feed line for a kill this player made or suffered.               |
| 3   | `NO_STATS`      | Ignored by client-side statistics such as kill counters and streaks.     |
| 4   | `CUSTOM_COLOR`  | Drawn in the player's [colour](#colour) instead of the team colour.      |
| 5-7 | reserved        | Must be `0`. Clients ignore unknown bits.                                |

Bits 0 to 3 change what is reported, not what is drawn: the player is still
rendered, heard and hit as usual. A client ignores them for its own player, and
still tells its player about their own kills and deaths.

## Colour

Blue, green, red, as in [Set Colour](../protocol075.md#set-colour). Used only
while `CUSTOM_COLOR` is set. It replaces the team colour on the player model,
the tool or weapon they hold, and their corpse. It does not change their team.

## Outfit

An outfit changes how a player looks and sounds, and nothing else. It keeps the
player's outline and draws the held tool or weapon as it is. With
`CUSTOM_COLOR` set, the colour tints the outfit. A client draws an unknown
outfit, or one it has no art for, as `0`.

| Value  | Name      | Look                                |
|--------|-----------|-------------------------------------|
| 0      | Soldier   | The normal player model.            |
| 1      | Undead    | Rotting skin, torn uniform, groans. |
| 2      | Scout     | Light kit, no helmet.               |
| 3      | Royal     | Crown and cape.                     |
| 4      | Vampire   | Pale, high-collared cloak.          |
| 5      | Miner     | Hard hat with a lamp, dusty.        |
| 6      | Ghillie   | Camouflage suit.                    |
| 7      | Brawler   | Bare arms, headband.                |
| 8      | Scientist | Lab coat, goggles.                  |
| 9      | Butcher   | Bloodied apron, rubber boots.       |
| 10     | Convict   | Striped prison uniform.             |
| 11-255 | reserved  | Drawn as `0`.                       |

## Sub ID 0: Extended Create Player

| Field Name    | Field Type   | Example  | Notes                                          |
|---------------|--------------|----------|------------------------------------------------|
| Packet ID     | UByte        | `0x74`   | Always `0x74`.                                 |
| Sub Packet ID | UByte        | `0`      | Always `0` for this sub-packet.                |
| Player ID     | UByte        | `254`    |                                                |
| Flags         | UByte        | `0b1011` | See [Flags](#flags).                           |
| Weapon        | UByte        | `0`      | As in Create Player.                           |
| Team          | UByte        | `0`      | As in Create Player.                           |
| X position    | LE Float     | `256.0`  | As in Create Player.                           |
| Y position    | LE Float     | `256.0`  | As in Create Player.                           |
| Z position    | LE Float     | `40.0`   | As in Create Player.                           |
| Colour        | UByte[3]     |          | See [Colour](#colour).                         |
| Outfit        | UByte        | `1`      | See [Outfit](#outfit).                         |
| Name          | CP437 String | `Wolf`   | As in Create Player, to the end of the packet. |

## Sub ID 1: Extended Existing Player

| Field Name    | Field Type   | Example  | Notes                                            |
|---------------|--------------|----------|--------------------------------------------------|
| Packet ID     | UByte        | `0x74`   | Always `0x74`.                                   |
| Sub Packet ID | UByte        | `1`      | Always `1` for this sub-packet.                  |
| Player ID     | UByte        | `254`    |                                                  |
| Flags         | UByte        | `0b1011` | See [Flags](#flags).                             |
| Team          | UByte        | `0`      | As in Existing Player.                           |
| Weapon        | UByte        | `0`      | As in Existing Player.                           |
| Held item     | UByte        | `0`      | As in Existing Player.                           |
| Kills         | LE UInt      | `0`      | As in Existing Player.                           |
| Block Colour  | UByte[3]     |          | Blue, green, red, as in Existing Player.         |
| Colour        | UByte[3]     |          | See [Colour](#colour).                           |
| Outfit        | UByte        | `1`      | See [Outfit](#outfit).                           |
| Name          | CP437 String | `Wolf`   | As in Existing Player, to the end of the packet. |

## Sub ID 2: Set Player

Changes the properties of a player the client already knows.

| Field Name    | Field Type | Example | Notes                           |
|---------------|------------|---------|---------------------------------|
| Packet ID     | UByte      | `0x74`  | Always `0x74`.                  |
| Sub Packet ID | UByte      | `2`     | Always `2` for this sub-packet. |
| Player ID     | UByte      | `254`   |                                 |
| Flags         | UByte      | `0`     | See [Flags](#flags).            |
| Colour        | UByte[3]   |         | See [Colour](#colour).          |
| Outfit        | UByte      | `0`     | See [Outfit](#outfit).          |

## Lifetime

The properties belong to the player id, and each sub-packet replaces all of
them. [Player Left](../protocol075.md#player-left) resets them to `0` once it is
applied, so a silent player leaves silently.
[Map Start](../protocol075.md#map-start-075) resets every id.

## Notes

Silent players are left out of the master server
[Count Update](../protocolmaster.md#count-update). Ids above `31` need
[Player Limit](player-limit.md), so servers allocate silent ids downwards from
`254`.

See [Extensions](extension.md) for how the extension is negotiated.
