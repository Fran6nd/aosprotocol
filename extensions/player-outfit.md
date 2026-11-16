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
| 0      | Set Outfit   | Server -> Client | 5             |
| 1      | Team Outfits | Server -> Client | `2+2*entries` |

## Outfit

An outfit changes how a player looks, and nothing else: hits, collision and
everything else are those of the normal player. It follows the player's stance,
so a crouching player's outfit is drawn crouching, and their corpse wears it. The
first-person view does not change.

An outfit is coloured in a team colour where the normal model's uniform is,
unless its row says otherwise. An outfit may hide the held tool or weapon. A
client draws an outfit it has no art for as `0`, so the art may ship apart from
the client, as an optional download.

Some themes have a range of their own, with room left to grow.

### General

| Value | Name        | Look                                                       |
|-------|-------------|------------------------------------------------------------|
| 0     | Soldier     | The normal player model.                                   |
| 1     | Scout       | Light kit, no helmet.                                      |
| 2     | Ghillie     | Camouflage suit.                                           |
| 3     | Bodyguard   | Dark suit, sunglasses, earpiece.                           |
| 4     | Insurgent   | Balaclava, chest rig.                                      |
| 5     | Convict     | Striped prison uniform.                                    |
| 6     | Builder     | High-visibility vest, tool belt.                           |
| 7     | VIP         | Suit and tie, a gold badge.                                |
| 8     | Medic       | Red cross armband, medical bag.                            |
| 9     | Terminator  | Metal endoskeleton, red eyes.                              |
| 10    | Cowboy      | Wide-brimmed hat, vest, spurs.                             |
| 11    | Cosmonaut   | Spacesuit, helmet with a visor.                            |
| 12    | Block Stack | Three blocks stacked, in the player's held block colour. No team colour, hides the held tool or weapon. |
| 13-63 | reserved    |                                                            |

### Zombies

| Value | Name               | Look                                              |
|-------|--------------------|---------------------------------------------------|
| 64    | Zombie             | Rotting skin, torn uniform.                       |
| 65    | Spitter            | Swollen throat, a dripping green mouth.           |
| 66    | Leaper             | Long limbs, hunched, claws.                       |
| 67    | Caller             | Gaping jaw, swollen chest.                        |
| 68    | Breaker            | Hulking, oversized fists.                         |
| 69    | Infected Cosmonaut | Torn spacesuit, cracked visor, a rotting face behind it. |
| 70-79 | reserved           |                                                   |

### Deployables

A deployable stands on a team-coloured tripod or base in place of the body. One
on a tripod points where the player aims. A deployable hides the held tool or
weapon.

| Value | Name         | Look                                                         |
|-------|--------------|--------------------------------------------------------------|
| 80    | Turret       | A gun on a tripod.                                           |
| 81    | Projector    | A lamp on a tripod. It gives no light.                       |
| 82    | Dish Antenna | A parabolic dish on a tripod.                                |
| 83    | Mast Antenna | A straight mast on a base.                                   |
| 84    | Target       | A paper target on a hinged base. It does not turn, and when the player dies it falls flat in place of a corpse. |
| 85    | Weathervane  | A vane on a mast.                                            |
| 86-95 | reserved     |                                                              |

### Other

| Value  | Name     | Look                                                |
|--------|----------|-----------------------------------------------------|
| 96-254 | reserved |                                                     |
| 255    | Team     | The player's [team outfit](#sub-id-1-team-outfits). |

## Sub ID 0: Set Outfit

| Field Name    | Field Type | Example | Notes                                               |
|---------------|------------|---------|-----------------------------------------------------|
| Packet ID     | UByte      | `0x75`  | Always `0x75`.                                      |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet.                     |
| Player ID     | UByte      | `254`   |                                                     |
| Outfit        | UByte      | `1`     | See [Outfit](#outfit).                              |
| Colour        | UByte      | `255`   | The team whose colour it wears. `255`, or a team the client does not know, is the player's own. |

To dress a player as another, the server sends them the other player's outfit
and team: `255` resolved to that team's outfit, and the team as Colour. The
client draws what it is sent and does not follow the other player afterwards.

The outfit belongs to the player id and takes effect when it arrives. A player
with no Set Outfit wears `255` in their own colour. The server sends a player's
Set Outfit before the [Create Player](../protocol075.md#create-player) or
[Existing Player](../protocol075.md#existing-player) that introduces them, so a
client draws a player in the outfit it holds when they are created and never
waits for one.

[Player Left](../protocol075.md#player-left) resets the id, and
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
| Team ID    | UByte      | `1`     | A team the client does not know is ignored. |
| Outfit     | UByte      | `64`    | `0` to `254`, see [Outfit](#outfit). |

Each Team Outfits replaces the previous one and takes effect when it arrives. A
team it does not list, or any team before the first Team Outfits, wears `0`. It
lasts for the connection.

See [Extensions](extension.md) for how the extension is negotiated.
