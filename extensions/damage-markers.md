# Damage Markers

Tells the client how much damage a hit did and who took it, so the client can
float the number over that player. The 0.75 protocol tells a client that somebody
was hit only through [Set HP](../protocol075.md#set-hp), which covers the local
player alone and carries no amount; a client shooting somebody else learns
nothing until they die. This extension fills that gap with one small packet per
hit.

| ------------: | ------------- |
| Extension ID: | 32            |
| Packet ID:    | 96            |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

The packet id is `64 + extension id`, see
[Extension IDs](extension.md#extension-ids).

### Sub Packets:

| Sub ID | Name          | Direction        | Size |
|--------|---------------|------------------|------|
| none   | Damage Marker | Server -> Client | 3    |

This extension carries **no sub packet id**, which departs from the
[general extension packet structure](extension.md#extension-packets): the damage
amount sits at offset 2, where a sub packet id would otherwise be. The only
implementation, [TigerSpades](https://github.com/rzrn/tigerspades), predates the
rule being written down, and the one packet leaves nothing to disambiguate, so
the wire format is documented as it is rather than as the rule would have it. A
future version that needs a second packet has to introduce the byte and bump the
version.

## Damage Marker

Sent by the server when a player takes damage. There is no client-to-server form;
a server that receives this packet drops it.

| Field Name    | Field Type | Example | Notes                                                    |
|---------------|------------|---------|----------------------------------------------------------|
| Packet ID     | UByte      | `96`    | Always `96`.                                             |
| Player ID     | UByte      | `7`     | The player who **took** the damage.                      |
| Hit Amount    | UByte      | `45`    | Health points the hit removed, see [Amount](#amount).    |

Always 3 bytes.

| Offset | Size | Field      |
|--------|------|------------|
| 0      | 1    | Packet ID  |
| 1      | 1    | Player ID  |
| 2      | 1    | Hit Amount |

## Amount

Hit Amount is the health the hit actually removed, after the server has applied
whatever it applies — falloff, armour, friendly-fire scaling — not the weapon's
nominal damage. It is what the victim's HP went down by, so a client can add
consecutive markers up and arrive at the damage it has done.

The field is a `UByte`. A server whose damage can exceed `255` clamps to `255`
rather than wrapping; a single hit that large has killed the target anyway.
Overkill is clamped the same way: a hit of `80` against `30` remaining HP is sent
as `30`, so the numbers a client shows never exceed the health the player had.

`0` means a hit that landed and did no damage, and is worth sending: it tells the
attacker they connected with something that absorbed the shot. A client may
render it differently, or not at all.

One packet per hit. Nothing merges or deduplicates them, so a burst that lands
several hits in the same frame produces several packets and several markers,
which is what a client wants to show.

## Audience

The packet has no audience field. The server sends it to whichever clients should
see the marker and to nobody else, and a client draws every marker it receives.

Normally that is the attacker alone, which is what "damage markers" means and all
the extension was built for. A server is free to do otherwise — sending a
player's own damage to them, or every hit to spectators — but it should know what
it discloses before it does: a marker names a player id and an amount, so
broadcasting them hands out an enemy's remaining health and reveals that an
unseen player is being shot.

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
