# Weapon and Tool State

Keeps a player's ammunition, blocks and grenades in step between client and
server. Today each side counts on its own: the server from how long fire is
held, the client from the rounds it fires, and a reload the server refuses
silently is finished by the client anyway. Here the client predicts and the
server corrects.

| ------------: | ------------- |
| Extension ID: | `0x39`        |
| Packet ID:    | `0x79`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

### Sub Packets:

| Sub ID | Name  | Direction         | Size |
|--------|-------|-------------------|------|
| 0      | State | Client <-> Server | 9    |

## Sub ID 0: State

A player's counts. From a client it is a claim, which lets the client act at
once; from the server it is the truth, which replaces the client's.

| Field Name    | Field Type | Example | Notes                                          |
|---------------|------------|---------|------------------------------------------------|
| Packet ID     | UByte      | `0x79`  | Always `0x79`.                                 |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet.                |
| Player ID     | UByte      | `7`     | Ignored from a client; the server fills it in. |
| Sequence      | UByte      | `41`    | See [Sequence](#sequence).                     |
| Reloading     | UByte      | `0`     | `1` while the gun reloads, else `0`. Other values read as `0`. |
| Magazine      | UByte      | `10`    | Rounds in the gun's magazine.                  |
| Reserve       | UByte      | `50`    | Rounds carried beyond it.                      |
| Blocks        | UByte      | `50`    | Blocks carried.                                |
| Grenades      | UByte      | `3`     | Grenades carried.                              |

### The client predicts

It counts on its own clock: when its reload ends it fills the magazine and may
fire at once. It sends a State when its reload ends, when primary fire is
released, after each round of the shotgun's reload, and after it places blocks
or throws a grenade. It still asks for a reload with
[Weapon Reload](../protocol075.md#weapon-reload).

### The server corrects

It keeps its own counts and never takes the client's. It answers a claim when
they differ by more than it tolerates; how much that is, in rounds, blocks or
grenades, and how long after a reload Reloading may still differ, is the
server's choice. It also sends its own State when it changes the counts itself:
when a reload starts, ends or is cancelled, a Restock, a pickup, a respawn, and
to a joining client.

To a client with the extension the server answers a Weapon Reload with a State,
not a Weapon Reload. To other clients with the extension it sends its own State
when a player's reload starts and ends, for their animation, and never relays a
client's claims; they may ignore the counts. Clients without the extension get
Weapon Reload as today.

### Sequence

A respawn sets the Sequence to `0`. The client's first claim after it carries
`1`, and later claims count on from `2`, wrapping after `255` back to `2`. After
a respawn the server ignores claims until one with `1` arrives, so a claim still
in transit from before is not taken for a new one.

The server's State carries the Sequence of the last claim it received, or `0`
when there has been none since the respawn. It describes the counts as they
were at that claim, or at the respawn for `0`, with only the server's own changes
since applied. Rounds fired, blocks placed and grenades thrown after the claim
are left out. The client adopts it and takes off what it has used since that
claim, so nothing is counted twice.

### Reloads

Switching away from the gun cancels its reload on both sides.

Jitter can make a magazine reload end on the client a moment before the
server's. A fire press that reaches the server in the last moment of such a
reload, as long as the server allows, counts rounds from the reload's end rather
than being refused. During the shotgun's reload a press stops the reload, as in
every client.

## Connection state

The counts belong to the player and last until
[Player Left](../protocol075.md#player-left).
[Map Start](../protocol075.md#map-start-075) does not clear them.

See [Extensions](extension.md) for how the extension is negotiated.
