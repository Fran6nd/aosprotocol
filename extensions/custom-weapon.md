# Custom Weapon

The server sets what each player carries: how many tools they have, in which
order, and how each firearm behaves. A loadout can be a spade and a block, a
grenade and an empty hand, or a rifle, a shotgun and a spade.

| ------------: | ------------- |
| Extension ID: | `0x39`        |
| Packet ID:    | `0x79`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

### Sub Packets:

| Sub ID | Name       | Direction         | Size |
|--------|------------|-------------------|------|
| 0      | Loadout    | Server -> Client  | 6+   |
| 1      | Hold       | Client <-> Server | 4    |
| 2      | Trace      | Server -> Client  | 20   |
| 3      | Hit Effect | Server -> Client  | 27   |

## Sub ID 0: Loadout

One player's tools. The server sends it to every client, for every player:
others need it to draw what a player holds and to time their reloads. A client
receiving one drops it.

| Field Name    | Field Type | Example | Notes                                   |
|---------------|------------|---------|-----------------------------------------|
| Packet ID     | UByte      | `0x79`  | Always `0x79`.                          |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet.         |
| Player ID     | UByte      | `7`     | Whose loadout this is.                  |
| Blocks        | UByte      | `50`    | Blocks after a [Restock](../protocol075.md#restock). |
| Grenades      | UByte      | `3`     | Grenades after a Restock.               |
| Slot Count    | UByte      | `3`     | Slots that follow.                      |
| Slots         | Slot[]     |         | See [Slot](#slot).                      |

Blocks and grenades are one pool each, shared by every slot that uses them, as
in the base game. A Restock refills them to these counts and refills every
firearm.

The server sends a player's Loadout after their
[Create Player](../protocol075.md#create-player) or
[Existing Player](../protocol075.md#existing-player), and again whenever it
changes it; the new one takes effect at once, and the player holds slot `0`.

### Slot

| Field Name | Field Type | Example | Notes                                             |
|------------|------------|---------|---------------------------------------------------|
| Tool       | UByte      | `2`     | See [Tools](#tools).                              |
| Firearm    | Firearm    |         | Only when Tool is `2`, see [Firearm](#firearm).   |

Slots are in the order the player cycles through them.

### Tools

The values of [Set Tool](../protocol075.md#set-tool), and one more.

| Value     | Tool       | Primary fire                |
|-----------|------------|-----------------------------|
| `0`       | Spade      | As in the base game.        |
| `1`       | Block      | As in the base game.        |
| `2`       | Firearm    | See [Firearm](#firearm).    |
| `3`       | Grenade    | As in the base game.        |
| `4`       | Empty hand | Nothing.                    |
| `5`-`255` | reserved   | A server does not send them. |

### Firearm

| Field Name   | Field Type | Example  | Notes                                                |
|--------------|------------|----------|------------------------------------------------------|
| Model        | UByte      | `0`      | Look, sounds and scope, see [Models](#models).       |
| Fire Type    | UByte      | `0`      | `0` hitscan, `1` simulated, see [Fire Types](#fire-types). |
| Flags        | UByte      | `0b1`    | See below.                                           |
| Reload       | UByte      | `0`      | `0` whole magazine, `1` one round at a time.         |
| Magazine     | UByte      | `10`     | Rounds in a full magazine.                           |
| Reserve      | UByte      | `50`     | Rounds carried beyond it.                            |
| Pellets      | UByte      | `1`      | Shots per round fired.                               |
| Block Damage | UByte      | `50`     | Hitscan only: taken off a block's health per hit; the client sends the destroy, as in the base game. |
| Interval     | LE Float   | `0.5`    | Seconds between rounds while primary fire is held.   |
| Reload Time  | LE Float   | `2.5`    | Seconds for a whole magazine, or per round.          |
| Spread       | LE Float   | `0.006`  | See [Aim](#aim).                                     |
| Recoil Up    | LE Float   | `0.05`   | See [Aim](#aim).                                     |
| Recoil Side  | LE Float   | `0.0001` | See [Aim](#aim).                                     |

| Bit | Name       | Meaning                                         |
|-----|------------|-------------------------------------------------|
| 0   | `STEADIED` | Crouching halves the spread.                    |
| 1-7 | reserved   | Must be `0`. Clients **must** ignore unknown bits. |

Every firearm fires for as long as primary fire is held, one round per Interval,
as the vanilla guns do and as servers count ammunition.

Each firearm slot keeps its own magazine and reserve. [Weapon
Reload](../protocol075.md#weapon-reload) and the ammunition of
[Restock](../protocol075.md#restock) refer to the slot the player holds. The
server times a reload from Reload Time and sends Weapon Reload when it is done,
as it does today; no packet from the client says a reload finished.

The three vanilla guns, in these terms:

| Gun     | Model | Fire Type | Flags | Reload | Magazine | Reserve | Pellets | Block Damage | Interval | Reload Time | Spread  | Recoil Up | Recoil Side |
|---------|-------|-----------|-------|--------|----------|---------|---------|--------------|----------|-------------|---------|-----------|-------------|
| Rifle   | `0`   | `0`       | `1`   | `0`    | `10`     | `50`    | `1`     | `50`         | `0.5`    | `2.5`       | `0.006` | `0.05`    | `0.0001`    |
| SMG     | `1`   | `0`       | `1`   | `0`    | `30`     | `120`   | `1`     | `34`         | `0.1`    | `2.5`       | `0.012` | `0.0125`  | `0.00005`   |
| Shotgun | `2`   | `0`       | `0`   | `1`    | `6`      | `48`    | `8`     | `34`         | `1.0`    | `0.5`       | `0.024` | `0.1`     | `0.0002`    |

### Fire Types

* **Hitscan.** As the vanilla guns: the client finds what each pellet hits and
  reports it with the [Hit Packet](../protocol075.md#hit-packet), and damage
  stays the server's.
* **Simulated.** The server flies every projectile itself, on any trajectory it
  likes: drag, gravity, wind, ricochets, penetration, a rocket or a grenade from
  a launcher. The client sends only its [Weapon
  Input](../protocol075.md#weapon-input) and orientation, as it does for any gun.
  It sends no Hit Packet for the firearm and draws no hit effect of its own; the
  server sends [Trace](#sub-id-2-trace) points for each projectile's path and a
  [Hit Effect](#sub-id-3-hit-effect) where one lands, and destroys blocks with
  [Block Action](../protocol075.md#block-action).

Spread and recoil still move the client's view for a simulated firearm, so that
it handles as it should; where the projectile goes is the server's.

### Aim

These are the vanilla guns' own values and work as the vanilla clients use them.

* **Spread.** Each pellet flies along the player's view direction plus a random
  offset of up to `2 × Spread` on each axis. Aiming down the sights halves it,
  and so does crouching with a `STEADIED` firearm.
* **Recoil Up.** Each round raises the view by this many radians.
* **Recoil Side.** Each round turns the view sideways by up to this many
  radians, alternating sides as the vanilla guns do.

### Models

| Value     | Model       | Drawn without it as |
|-----------|-------------|---------------------|
| `0`       | Rifle       |                     |
| `1`       | SMG         |                     |
| `2`       | Shotgun     |                     |
| `3`       | Bolt-action | Rifle               |
| `4`       | Pistol      | Rifle               |
| `5`       | Machine gun | SMG                 |
| `6`-`255` | reserved    | Rifle               |

A Model is the firearm's look, sounds, scope and kill icon, nothing else.

## Sub ID 1: Hold

Which slot a player holds. A client with a loadout sends Hold rather than
[Set Tool](../protocol075.md#set-tool) when its player switches.

| Field Name    | Field Type | Example | Notes                                    |
|---------------|------------|---------|------------------------------------------|
| Packet ID     | UByte      | `0x79`  | Always `0x79`.                           |
| Sub Packet ID | UByte      | `1`     | Always `1` for this sub-packet.          |
| Player ID     | UByte      | `7`     | Ignored from a client; the server fills it in. |
| Slot          | UByte      | `2`     | An index into the player's Slots.        |

The server relays it to clients with the extension, and sends the others a base
Set Tool for the slot's Tool, with an empty hand as the spade. A Slot beyond the
player's Slot Count, such as one still in transit when a smaller Loadout
replaced the old one, is ignored.

## Sub ID 2: Trace

One point on a simulated projectile's path. The client draws the path through
the points of each projectile in the order they arrive.

| Field Name    | Field Type | Example | Notes                                             |
|---------------|------------|---------|---------------------------------------------------|
| Packet ID     | UByte      | `0x79`  | Always `0x79`.                                    |
| Sub Packet ID | UByte      | `2`     | Always `2` for this sub-packet.                   |
| Projectile    | UByte      | `12`    | Which projectile; the server reuses numbers.      |
| X             | LE Float   | `256.0` | Position.                                         |
| Y             | LE Float   | `256.0` |                                                   |
| Z             | LE Float   | `40.0`  |                                                   |
| Value         | LE Float   | `700.0` | A value the server attaches to the point, such as the projectile's speed; a client may use it to draw the path. |
| Origin        | UByte      | `1`     | `1` for the first point of a new projectile, which forgets the old path with the same number; else `0`. |

A point for a Projectile number with no Origin yet is ignored.

## Sub ID 3: Hit Effect

Where a simulated projectile hit, for the client to show it.

| Field Name    | Field Type | Example | Notes                                  |
|---------------|------------|---------|----------------------------------------|
| Packet ID     | UByte      | `0x79`  | Always `0x79`.                         |
| Sub Packet ID | UByte      | `3`     | Always `3` for this sub-packet.        |
| X             | LE Float   | `256.4` | Where it hit.                          |
| Y             | LE Float   | `256.1` |                                        |
| Z             | LE Float   | `40.2`  |                                        |
| Block X       | LE Int     | `256`   | The block hit, for a block hit.        |
| Block Y       | LE Int     | `256`   |                                        |
| Block Z       | LE Int     | `40`    |                                        |
| Target        | UByte      | `0`     | `0` block, `1` head, `2` player.       |

Trace and Hit Effect follow TigerSpades' Trace Bullets and Hit Effects
extensions, used by the [aos-milsim](https://github.com/rzrn/aos-milsim) server
for its simulated ballistics, field for field, so that both can move here.

## Base packets

* **Create Player** and **Existing Player** carry the base weapon nearest the
  player's first firearm, or the rifle when there is none, for clients without
  the extension. A loadout survives a respawn.
* **Change Weapon** means nothing while a loadout is set.
* **Kill Action** keeps its kill types; a client draws the kill icon of the
  killer's held slot.

Clients without the extension see only base packets: the vanilla weapon above,
and Set Tool as the server translates it.

## Connection state

A player's loadout lasts until another replaces it or
[Player Left](../protocol075.md#player-left).
[Map Start](../protocol075.md#map-start-075) does not clear it.

See [Extensions](extension.md) for how the extension is negotiated.
