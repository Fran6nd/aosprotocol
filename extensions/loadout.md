# Loadout

The server sets what a player carries in hand: which tools, in which order, and
how many blocks and grenades. A loadout can be a spade and a block, a grenade
and an empty hand, or a rifle, a shotgun and a spade. Each tool behaves as its
base tool does.

| ------------: | ------------- |
| Extension ID: | `0x3B`        |
| Packet ID:    | `0x7B`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

### Sub Packets:

| Sub ID | Name    | Direction         | Size |
|--------|---------|-------------------|------|
| 0      | Loadout | Server -> Client  | 7+   |
| 1      | Hold    | Client <-> Server | 4    |

## Sub ID 0: Loadout

One player's slots. The server sends it to every client, for every player, so
that each can draw what others hold. A server receiving one drops it.

| Field Name    | Field Type | Example | Notes                                   |
|---------------|------------|---------|-----------------------------------------|
| Packet ID     | UByte      | `0x7B`  | Always `0x7B`.                          |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet.         |
| Player ID     | UByte      | `7`     | Whose loadout this is.                  |
| Flags         | UByte      | `0b00`  | See below.                              |
| Blocks        | UByte      | `50`    | Blocks after a [Restock](../protocol075.md#restock). |
| Grenades      | UByte      | `3`     | Grenades after a Restock.               |
| Slot Count    | UByte      | `3`     | Slots that follow.                      |
| Slots         | Slot[]     |         | See [Slot](#slot).                      |

| Bit | Name                | Meaning                                              |
|-----|---------------------|------------------------------------------------------|
| 0   | `INFINITE_BLOCKS`   | Placing a block never uses one up.                   |
| 1   | `INFINITE_GRENADES` | Throwing a grenade never uses one up.                |
| 2   | `INFINITE_RESERVE`  | Reloading a gun never uses up its reserve.           |
| 3-7 | reserved            | Must be `0`. Clients **must** ignore unknown bits.   |

With an infinite flag the matching count is ignored, a reload is always
allowed, and the client shows no count.

Blocks and grenades are one pool each, shared by every slot that uses them, as
in the base game. The server sends a player's Loadout after their
[Create Player](../protocol075.md#create-player) or
[Existing Player](../protocol075.md#existing-player), and again whenever it
changes it, each time followed by a [Hold](#sub-id-1-hold). A new Loadout takes
effect at once and starts full.

### Slot

| Field Name | Field Type | Example | Notes                                        |
|------------|------------|---------|----------------------------------------------|
| Tool       | UByte      | `2`     | See below.                                   |
| Weapon     | UByte      | `1`     | Only when Tool is `2`: `0` rifle, `1` SMG, `2` shotgun, as in [Change Weapon](../protocol075.md#change-weapon). |

| Value     | Tool       | Primary fire                               |
|-----------|------------|--------------------------------------------|
| `0`       | Spade      | As in the base game.                       |
| `1`       | Block      | As in the base game.                       |
| `2`       | Gun        | As the base weapon in Weapon.              |
| `3`       | Grenade    | As in the base game.                       |
| `4`       | Empty hand | Nothing.                                   |
| `5`-`255` | reserved   | A client treats the slot as an empty hand. |

Slots are in the order the player cycles through them. Each gun slot keeps its
own magazine and reserve; [Weapon and Tool State](weapon-tool-state.md) counts
them by slot, and [Cosmetic Weapon and Tool](cosmetic-weapon-tool.md) gives each
slot its look.

## Sub ID 1: Hold

Which slot a player holds. A client with a loadout sends Hold rather than
[Set Tool](../protocol075.md#set-tool) when its player switches.

| Field Name    | Field Type | Example | Notes                                          |
|---------------|------------|---------|------------------------------------------------|
| Packet ID     | UByte      | `0x7B`  | Always `0x7B`.                                 |
| Sub Packet ID | UByte      | `1`     | Always `1` for this sub-packet.                |
| Player ID     | UByte      | `7`     | Ignored from a client; the server fills it in. |
| Slot          | UByte      | `2`     | An index into the player's Slots.              |

The server relays it to clients with the extension, and sends the others a base
Set Tool for the slot's Tool, with an empty hand as the spade. A Slot beyond the
player's Slot Count is ignored.

## Base packets

* **Create Player** and **Existing Player** carry, for clients without the
  extension, the weapon of the player's first gun slot, or the rifle when there
  is none. A respawn refills the loadout, magazines included.
* **Set Tool** from a client is handled as in the base game until its Loadout
  arrives. Once a loadout is set, the server reads it as a Hold for the first
  slot of that Tool.
* **Kill Action** keeps its kill types.

## Connection state

A player's loadout lasts until another replaces it or
[Player Left](../protocol075.md#player-left).
[Map Start](../protocol075.md#map-start-075) does not clear it.

See [Extensions](extension.md) for how the extension is negotiated.
