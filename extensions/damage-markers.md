# Damage Markers

Tells the client how much damage a hit did and who took it, so the client can
float the number over that player. The 0.75 protocol tells a client that somebody
was hit only through [Set HP](../protocol075.md#set-hp), which covers the local
player alone and carries no amount; a client shooting somebody else learns
nothing until they die. This extension fills that gap with one small packet per
hit.

| ------------: | ------------- |
| Extension ID: | `0x20`        |
| Packet ID:    | `0x60`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

The packet id is `0x40 + extension id`, see
[Extension IDs](extension.md#extension-ids).

### Sub Packets:

| Sub ID | Name          | Direction        | Size |
|--------|---------------|------------------|------|
| none   | Damage Marker | Server -> Client | 3    |

This extension carries **no sub packet id**, which departs from the
[general extension packet structure](extension.md#extension-packets): the damage
amount sits at offset 2, where a sub packet id would otherwise be. Both
implementations predate the rule being written down, and the one packet leaves
nothing to disambiguate, so the wire format is documented as it is rather than as
the rule would have it. A future version that needs a second packet has to
introduce the byte and bump the version.

The implementations are [TigerSpades](https://github.com/rzrn/tigerspades)
(`src/network.c`, `src/main.c`) on the client side and
[aos-arena](https://github.com/rzrn/aos-arena)
(`arenalib/packets.py`, `game_modes/arena.py`) on the server side.

## Damage Marker

Sent by the server when a player takes damage. There is no client-to-server form;
a server that receives this packet drops it.

| Field Name    | Field Type | Example | Notes                                                    |
|---------------|------------|---------|----------------------------------------------------------|
| Packet ID     | UByte      | `0x60`  | Always `0x60`.                                           |
| Player ID     | UByte      | `7`     | The player who **took** the damage.                      |
| Hit Amount    | UByte      | `45`    | Health points the hit removed, see [Amount](#amount).    |

Always 3 bytes.

| Offset | Size | Field      |
|--------|------|------------|
| 0      | 1    | Packet ID  |
| 1      | 1    | Player ID  |
| 2      | 1    | Hit Amount |

## Amount

Hit Amount is the damage the server is about to apply, once it has resolved the
hit its own way — hit area, distance falloff, melee — and after any script has
had the chance to adjust it. In aos-arena it is the value handed to the victim's
`hit()`, sent just before that call.

It is the damage, **not** the health the victim actually lost. The two differ on
the hit that kills: a hit of `80` against `30` remaining HP is sent as `80`, so
the last marker of a kill overstates what it took. Clients must not sum markers
and expect the victim's health to fall out of it.

The field is a `UByte`, which caps a single hit at `255`. Vanilla weapon damage
and melee stay far below that, but a script that scales damage up has to clamp
before sending rather than let the value wrap.

`0` means a hit that landed and did no damage — in practice, a script that zeroed
it. A client may render it differently, or not at all.

One packet per hit. Nothing merges or deduplicates them, so a burst that lands
several hits in the same frame produces several packets and several markers,
which is what a client wants to show.

## Audience

The packet has no audience field. The server sends it to whichever clients should
see the marker and to nobody else, and a client draws every marker it receives.

**The audience is the attacker alone.** aos-arena sends the packet only back down
the connection the hit arrived on, and only when that player negotiated the
extension. A server is free to do otherwise, but it should know what it discloses
before it does: a marker names a player id and an amount, so sending them more
widely hands out an enemy's remaining health and reveals that an unseen player is
being shot.

The victim is not told they were hit unless the server includes them in the
recipients. [Set HP](../protocol075.md#set-hp) remains the packet that informs
them, and this extension neither replaces nor duplicates it.

## Rendering

The client decides what a marker looks like. The extension carries a number and a
player id, and nothing about the presentation is normative.

What TigerSpades does, as a reference rather than a requirement: the number
appears above the named player's head with a small random offset and drifts
upwards, red, prefixed with a minus sign, fading out over about three seconds. It
is drawn only while the world is on screen.

Because the packet carries no position, the client anchors the marker to wherever
it currently believes that player is. A marker for a player the client does not
know, or has not had a position for recently, lands somewhere stale — clients
should drop markers for player ids they do not have a live player for.

## Negotiation and state

A marker is a momentary event, not state. Nothing is retained, so there is
nothing to clear on [Player Left](../protocol075.md#player-left) or
[Map Start](../protocol075.md#map-start-075) beyond the markers a client happens
to have on screen, which it discards for a player who leaves and for all players
on a map change.

Advertising the extension means a client can display markers, not that it will:
in TigerSpades damage markers are a setting, off by default, and the extension is
advertised either way. Servers cannot tell whether a given player sees them, and
nothing in the game may depend on it.

See [Extensions](extension.md) for how the extension is negotiated.
