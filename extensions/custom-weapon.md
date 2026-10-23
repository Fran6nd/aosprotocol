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
| 0      | Loadout    | Server -> Client  | 7+   |
| 1      | Hold       | Client <-> Server | 4    |
| 2      | Weapon State | Client <-> Server | 10 |

## Sub ID 0: Loadout

One player's tools. The server sends it to every client, for every player:
others need it to draw what a player holds and to time their reloads. A server
receiving one drops it.

| Field Name    | Field Type | Example | Notes                                   |
|---------------|------------|---------|-----------------------------------------|
| Packet ID     | UByte      | `0x79`  | Always `0x79`.                          |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet.         |
| Player ID     | UByte      | `7`     | Whose loadout this is.                  |
| Flags         | UByte      | `0b00`  | See below.                              |
| Blocks        | UByte      | `50`    | Blocks after a [Restock](../protocol075.md#restock). |
| Grenades      | UByte      | `3`     | Grenades after a Restock.               |
| Slot Count    | UByte      | `3`     | Slots that follow.                      |
| Slots         | Slot[]     |         | See [Slot](#slot).                      |

| Bit | Name                | Meaning                                         |
|-----|---------------------|-------------------------------------------------|
| 0   | `INFINITE_BLOCKS`   | Placing a block never uses one up.              |
| 1   | `INFINITE_GRENADES` | Throwing a grenade never uses one up.           |
| 2-7 | reserved            | Must be `0`. Clients **must** ignore unknown bits. |

With an infinite flag, the matching count is ignored and the client shows none.

Blocks and grenades are one pool each, shared by every slot that uses them, as
in the base game. A Restock refills them to these counts and refills every
firearm's reserve, leaving its magazine as it is, as in the base game.

The server sends a player's Loadout after their
[Create Player](../protocol075.md#create-player) or
[Existing Player](../protocol075.md#existing-player), and again whenever it
changes it, each time followed by a [Hold](#sub-id-1-hold) for the slot the player
holds. A new Loadout takes effect at once and starts full: every magazine and
reserve, the blocks and the grenades at their counts. A Create Player refills
it the same way, magazines included, as a respawn does in the base game.

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
| `5`-`255` | reserved   | A client treats the slot as an empty hand. |

### Firearm

| Field Name   | Field Type | Example  | Notes                                                |
|--------------|------------|----------|------------------------------------------------------|
| Model        | UByte      | `0`      | Look, sounds and scope, see [Models](#models).       |
| Flags        | UByte      | `0b0`    | See below.                                           |
| Reload       | UByte      | `0`      | `0` whole magazine, `1` one round at a time; other values are reserved and read as `0`. |
| Magazine     | LE UShort  | `10`     | Rounds in a full magazine.                           |
| Reserve      | LE UShort  | `50`     | Rounds carried beyond it.                            |
| Pellets      | UByte      | `1`      | Shots per round fired.                               |
| Block Damage | UByte      | `50`     | Taken off a block's health per hit; the client sends the destroy, as in the base game. |
| Interval     | LE Float   | `0.5`    | Seconds between rounds while primary fire is held.   |
| Reload Time  | LE Float   | `2.5`    | Seconds for a whole magazine, or per round.          |
| Spread       | LE Float   | `0.006`  | See [Aim](#aim).                                     |
| Recoil Up    | LE Float   | `0.05`   | See [Aim](#aim).                                     |
| Recoil Side  | LE Float   | `0.0001` | See [Aim](#aim).                                     |

| Bit | Name               | Meaning                                     |
|-----|--------------------|---------------------------------------------|
| 0   | `INFINITE_RESERVE` | Reloading never uses up the reserve. Reserve is ignored, a reload is always allowed, and the client shows no reserve. |
| 1   | `STEADIED`         | Crouching halves the spread.                |
| 2-7 | reserved           | Must be `0`. Clients **must** ignore unknown bits. |

Every firearm fires for as long as primary fire is held, one round per Interval,
as the vanilla guns do. piqueserver times the vanilla SMG at `0.11` seconds
rather than `0.1`, which is part of the drift Weapon State corrects.

Each firearm slot keeps its own magazine and reserve, and its counts travel in
[Weapon State](#sub-id-2-weapon-state). A client asks to reload the slot it holds
with [Weapon Reload](../protocol075.md#weapon-reload); switching to another slot
cancels a reload in progress. The
ammunition in [Player Properties](player-properties.md) is that of the slot the
player holds; its counts are bytes, so they are capped at `255`, and an infinite
count is sent as `255`.

The three vanilla guns, in these terms:

| Gun     | Model | Flags | Reload | Magazine | Reserve | Pellets | Block Damage | Interval | Reload Time | Spread  | Recoil Up | Recoil Side |
|---------|-------|-------|--------|----------|---------|---------|--------------|----------|-------------|---------|-----------|-------------|
| Rifle   | `0`   | `2`   | `0`    | `10`     | `50`    | `1`     | `50`         | `0.5`    | `2.5`       | `0.006` | `0.05`    | `0.0001`    |
| SMG     | `1`   | `2`   | `0`    | `30`     | `120`   | `1`     | `34`         | `0.1`    | `2.5`       | `0.012` | `0.0125`  | `0.00005`   |
| Shotgun | `2`   | `0`   | `1`    | `6`      | `48`    | `8`     | `34`         | `1.0`    | `0.5`       | `0.024` | `0.1`     | `0.0002`    |

Damage to players stays the server's: the client reports hits with the
[Hit Packet](../protocol075.md#hit-packet) as it does today.

### Aim

These are the vanilla guns' own values and work as the vanilla clients use them.

* **Spread.** Each pellet flies along the player's view direction plus a random
  offset of up to `2 × Spread` on each axis. OpenSpades and ZeroSpades add each
  pellet's offset to the previous pellet's direction, as 0.75 does;
  BetterSpades starts every pellet from the view direction. Aiming down the sights halves it,
  and so does crouching with a `STEADIED` firearm, as ZeroSpades and BetterSpades
  do for every vanilla gun but the shotgun.
* **Recoil Up** and **Recoil Side** are the constants the vanilla clients
  scale per round: Recoil Up raises the view, in radians, and Recoil Side is
  multiplied by a triangle wave running between `-255.5` and `255.5` over
  1024 milliseconds before it turns the view. Both are doubled while walking
  without aiming and doubled again in the air, and halved while crouching on the
  ground, as for the vanilla guns.

### Models

| Value     | Model       | Looks and animates like                          | Drawn without it as |
|-----------|-------------|--------------------------------------------------|---------------------|
| `0`       | Rifle       | The vanilla rifle.                               |                     |
| `1`       | SMG         | The vanilla SMG.                                 |                     |
| `2`       | Shotgun     | The vanilla shotgun, pumped between rounds.      |                     |
| `3`       | Bolt-action | A long rifle whose bolt is worked between rounds. | Rifle              |
| `4`       | Pistol      | A handgun, held in one hand.                     | Rifle               |
| `5`       | Machine gun | A heavy belt-fed gun.                            | SMG                 |
| `6`-`255` | reserved    |                                                  | Rifle               |

A Model is the firearm's look, animation, sounds, scope and kill icon, nothing
else: how it fires comes from the other fields. Its animation between rounds,
such as working a bolt or a pump, lasts the firearm's Interval. A client draws a model it lacks
as the one in the last column.

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

## Sub ID 2: Weapon State

One firearm slot's ammunition. From a client it is a claim, which lets the
client act at once; from the server it is the truth, which replaces the
client's.

| Field Name    | Field Type | Example | Notes                                           |
|---------------|------------|---------|-------------------------------------------------|
| Packet ID     | UByte      | `0x79`  | Always `0x79`.                                  |
| Sub Packet ID | UByte      | `2`     | Always `2` for this sub-packet.                 |
| Player ID     | UByte      | `7`     | Ignored from a client; the server fills it in.  |
| Sequence      | UByte      | `41`    | See below.                                      |
| Slot          | UByte      | `1`     | A firearm slot of that player.                  |
| State         | UByte      | `0`     | `0` ready, `1` reloading. Other values are reserved and read as `0`. |
| Magazine      | LE UShort  | `30`    | Rounds in the magazine.                         |
| Reserve       | LE UShort  | `64`    | Rounds carried beyond it.                       |

**The client predicts.** It reloads on its own clock: when its reload ends it
fills the magazine, may fire at once, and sends its Weapon State. It sends one
too when primary fire is released, and after each round of a one-round-at-a-time
reload. Each slot counts its own Sequence: a Loadout or a respawn sets it to
`0`, the first claim after it carries `1`, and later claims count on from `2`,
wrapping after `255` back to `2`. After a Loadout or respawn the server ignores
a slot's claims until one with `1` arrives, so a claim still in transit from
before is not taken for a new one. A Weapon State for a Slot beyond Slot Count,
or that is not a firearm, is ignored.

**The server corrects.** It keeps its own count and never takes the client's.
It answers a claim when they differ by more than it tolerates. How much that
is, in rounds and in how long after a reload a State may still differ, is the
server's choice: network jitter decides it, and piqueserver already tolerates
some drift in its count.
It also sends its own Weapon State when it changes a slot itself: when a reload
it was asked for starts, ends or is cancelled by a slot switch, a Restock, a
pickup, and to a joining client.
A new Loadout needs none: it starts full.

The server's Weapon State carries the Sequence of the last claim it received
for that slot, or `0` when there has been none since the Loadout or respawn. It
describes the slot as it was at that claim, or at the Loadout or respawn for
`0`, with only the server's own changes since applied: a reload, a Restock, a
pickup. Rounds fired
after the claim are left out. The client adopts it and takes off the rounds it
has fired since that claim, so none is counted twice.

**A press during the server's reload.** Jitter can make a whole-magazine
reload end on the client a moment before the server's. A fire press that reaches
the server in the last moment of such a reload, as long as the server allows,
counts rounds from the reload's end rather than being refused. During a
one-round-at-a-time reload a press stops the reload, as in every client.

**Replies and relays.** To clients with the extension the server answers a
Weapon Reload with Weapon State, not Weapon Reload. To the other clients it
sends its own Weapon State when a reload starts and ends, for their animation,
and never relays a client's claims; they may ignore the counts. Clients
without the extension get Weapon Reload as today, its byte counts capped at
`255`.

## Base packets

* **Create Player** and **Existing Player** carry, for clients without the
  extension, the vanilla weapon the player's first firearm is
  [drawn as](#models), or the rifle when there is none. A loadout survives a
  respawn.
* **Set Tool** from a client is handled as in the base game until its Loadout
  arrives; the Hold that follows the Loadout settles the slot. Once a loadout is
  set, the server reads a client's Set Tool as a Hold for the first slot of that
  Tool, and handles Change Weapon as it sees fit.
* **Kill Action** keeps its kill types. For a weapon or headshot kill, a client
  draws the kill icon of the killer's slot as last told by Hold; the other types
  keep their own.

Clients without the extension see only base packets: the vanilla weapon above,
and Set Tool as the server translates it. BetterSpades counts down other
players' rounds with the vanilla weapon's magazine and stops drawing their shots
at `0`, so a larger magazine goes silent early there. A server may keep its
count up with [Player Properties](player-properties.md), which BetterSpades
applies silently; a Weapon Reload would play a reload sound each time.

## Connection state

A player's loadout lasts until another replaces it or
[Player Left](../protocol075.md#player-left).
[Map Start](../protocol075.md#map-start-075) does not clear it.

See [Extensions](extension.md) for how the extension is negotiated.
