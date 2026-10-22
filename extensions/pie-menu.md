# Pie Menu

A pie menu is a ring that opens around the crosshair while a key is held. It is
divided into wedges, and a player picks one by flicking the aim towards it and
releasing the key. Each wedge sends one thing: a chat line, a ping on the world,
or a command for the server to read.

One ring is a **pie** and one wedge is a **slice**. The point of the extension is
that the server chooses what is on offer, since it knows its mode, its commands
and its vocabulary.

| ------------: | ------------- |
| Extension ID: | `0x31`        |
| Packet ID:    | `0x71`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

### Sub Packets:

| Sub ID | Name | Direction        | Size |
|--------|------|------------------|------|
| 0      | Menu | Server -> Client | 3+   |

Server to client only; a server that receives one drops it.

## Sub ID 0: Menu

The whole menu, every time: a Menu replaces the one the client was showing,
entire.

| Field Name    | Field Type | Example | Notes                                 |
|---------------|------------|---------|---------------------------------------|
| Packet ID     | UByte      | `0x71`  | Always `0x71`.                        |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet.       |
| Pie Count     | UByte      | `4`     | Pies that follow, `0`-`10`.           |
| Pies          | Pie[]      |         | The remaining bytes, see [Pie](#pie). |

Theme, Text, Label and Tooltip are NUL-terminated strings, as elsewhere in the
protocol, and carry text the way a
[Chat Message](../protocol075.md#chat-message) does: Code Page 437, or UTF-8
behind the `0xff` prefix of [UTF-8 Chat](utf-8-chat.md). Each limit below is in
bytes, counting a `0xff` prefix but not the NUL.

### Pie

One ring, captioned in the middle by its theme.

| Field Name       | Field Type | Example    | Notes                                   |
|------------------|------------|------------|-----------------------------------------|
| Contexts         | UByte      | `0b011`    | Where it is offered, see [Contexts](#contexts). |
| Theme Message ID | UByte      | `0`        | Reserved. Must be `0`; a client ignores it. |
| Theme            | String     | `"Social"` | The centre caption, up to `32` bytes. Empty means none. |
| Slices           | Slice[6]   |            | Exactly six, see [Slice](#slice).       |

Every pie has exactly six slices, so there is no slice count on the wire: a
direction only means something if it means the same thing every time. A server
wanting fewer fills the spare wedges with [`NONE`](#actions); one wanting more
uses another pie.

Pies are offered in the order sent, within each context. Which opens first is the
client's business.

### Contexts

What the crosshair is on when the menu opens decides which pies are offered. A
bitmask, so a ring that suits more than one context is sent once.

| Bit | Name        | Offered when the menu opens on                     |
|-----|-------------|----------------------------------------------------|
| 0   | `WORLD`     | Terrain, the sky, or nothing in particular.        |
| 1   | `TEAMMATE`  | A player on the opening player's own team.         |
| 2   | `ENEMY`     | A player on the other team.                        |
| 3   | `SPECTATOR` | Anything, while the opening player is a spectator. |
| 4-7 | reserved    | Must be `0`. Clients **must** ignore unknown bits. |

A spectator is team `255`, see [Team IDs](../protocol075.md#team-ids). They have
no team, so bits `0`-`2` never apply to one and a spectator is offered
`SPECTATOR` pies alone. A staff member spectating from a team is a spectator
here too; the server knows its staff and offers them `SPECTATOR` pies to suit.

The context is fixed when the menu opens and does not change while it is held. A
context with no pies has no menu, and no other context's rings stand in.

### Slice

One wedge. It draws a Label and sends exactly one thing.

| Field Name   | Field Type | Example   | Notes                                      |
|--------------|------------|-----------|--------------------------------------------|
| Action       | UByte      | `1`       | See [Actions](#actions).                   |
| Message ID   | UByte      | `0`       | Reserved. Must be `0`; a client ignores it. |
| Flags        | UByte      | `0b0`     | See below.                                 |
| Text         | String     | `"/apoc"` | What it sends, up to `255` bytes.          |
| Label        | String     | `"Nuke"`  | What it draws, up to `48` bytes. **Empty means draw the Text.** |
| Tooltip      | String     | `"Calls in an airstrike"` | Up to `128` bytes. Empty means none. |

The Tooltip says more about the slice than its Label can. A client shows it
while the slice is highlighted, before the key is released, and never sends it.
Where and how it is drawn is the client's business, and a client may leave
tooltips out.

| Bit | Name     | Meaning                                                      |
|-----|----------|--------------------------------------------------------------|
| 0   | `GLOBAL` | Goes to the whole server rather than to the team.             |
| 1-7 | reserved | Must be `0`. Clients **must** ignore unknown bits.            |

`GLOBAL` is the team/global bit of the base
[Chat Message](../protocol075.md#chat-message) and nothing more, kept per slice
so that one ring may hold a line for the room and a line for the team. **There is
no enemy channel**, so the [`ENEMY`](#contexts) context decides which pies are
offered, never where their words go.

### Actions

| Value     | Name      | Choosing the slice sends                                   |
|-----------|-----------|------------------------------------------------------------|
| `0`       | `NONE`    | Nothing at all.                                            |
| `1`       | `CHAT`    | Text, as a [Chat Message](../protocol075.md#chat-message). |
| `2`       | `PING`    | A [Teamplay Ping](teamplay.md#sub-id-1-ping), Text as its Reason. |
| `3`       | `COMMAND` | Text, as a chat message the server reads as a command.     |
| `4`-`255` | reserved  | A client treats an unknown Action as `NONE`.               |

The Action is also what a client styles a slice by, so that a wedge which pings,
one which speaks and one which runs a command need not look alike.

What a slice sends goes out however the client normally sends chat.

**`PING`** marks the world position the crosshair was on when the menu opened,
not where it points when the slice is chosen. The Text becomes the ping's Reason,
which is plain UTF-8: the client drops a `0xff` prefix, or decodes Code Page 437.
It may be empty for a neutral marker. It needs
[Teamplay](teamplay.md) negotiated with its `PING` bit set; without either, the
client sends the Text as chat instead, so the words still arrive and only the
marker is lost.

**`COMMAND`** always goes on the team channel, whatever `GLOBAL` says. On the
wire it is an ordinary chat packet: the client interprets none of it, does not
look for a leading `/`, and has no local command language reachable from here.

A `NONE` slice does nothing at all — no packet, no message, no sound, no error.
How an empty wedge looks is the client's business, but it must not stand in for a
slice that is missing by inventing one.

A menu sends ordinary chat, so rate-limiting it is the server's business. Nothing
here asks for a limit, and a server is free to apply one.

### Substitutions

The Text of a slice may name the player the menu was opened on, so that a command
acting on a player can be told which one.

| Token | Expands to                                                            |
|-------|-----------------------------------------------------------------------|
| `%p`  | The player aimed at when the menu opened, as `#` and its id in decimal. |
| `%%`  | A literal percent sign.                                               |

So `/votekick %p` reaches the server as `/votekick #7`, the form a command's
player argument takes. A `%` followed by
anything else is reserved, and version 1 leaves both characters as they are.

The client expands once, immediately before sending, and never re-scans the
result, so a name that looks like a token is not one. The player aimed at is the
one under the crosshair when the menu opened, whether the opening player is
playing or spectating, so a [`SPECTATOR`](#contexts) pie can name one too. `%p`
expands to nothing when no player was aimed at, as always in the
[`WORLD`](#contexts) context, leaving the argument missing rather than a bare
`#`. The Text limit counts the bytes in the packet,
before expansion.

### Language

Themes, Labels and Tooltips **should** be in the player's User locale, from a
[Version Response (Extended)](../protocol075.md#version-response-extended-ep): the
server matches the language and region, then the language alone, then falls back
to its own default, which it also uses with no locale. The Text of a `CHAT` or
`PING` slice may be translated too, since it is what the player says; the Text
of a `COMMAND` slice never is, so such a slice gives a Label. A Menu sent before
the locale arrives is sent again after it.

## Replacing the client's menu

* **No Menu has arrived.** The client keeps its own.
* **A Menu with pies.** It replaces the client's own entirely. The two are never
  merged and never both reachable.
* **A Menu with a Pie Count of `0`.** There is no pie menu here, and the client's
  own menu does not come back.

A client **must not** add slices, reorder them, or drop one it dislikes. It may
refuse the menu outright, but it may not show a menu that claims to be the
server's and is not.

A server sends a Menu whenever it likes; each one replaces the whole menu, so the
last to arrive is the menu. The client applies a new one **the next time the menu
is opened**, not the instant it arrives, so that a ring cannot change under a held
thumb.

## Geometry

* Slice `0` is at the top, the rest **clockwise**.
* The six slices divide the ring evenly, `60` degrees each, so slice `0` is
  centred on straight up and slice `3` on straight down.
* The centre selects nothing, so releasing the key without choosing is always
  possible.

Everything else — radius, colour, animation, how rings are cycled, hold or toggle
— is the client's and is not on the wire.

## Validation

A client parses the whole packet before applying any of it, and **a menu is
applied whole or not at all**. Where one is refused the previous menu stands and
the client logs it rather than disconnecting. A Menu is refused when:

* A record or a string runs past the end of the packet, or bytes remain after
  the last pie.
* Pie Count is outside its range, or a string is longer than its limit.
* A string behind the `0xff` prefix is not well-formed UTF-8.
* A string holds a control character, `0x01`-`0x1f` or `0x7f`.

## Connection state

The menu belongs to the connection, not to the world and not to a player.
[Map Start](../protocol075.md#map-start-075) does not clear it, the rule the
[Teamplay Config](teamplay.md#sub-id-0-config) already follows.
[Player Left](../protocol075.md#player-left) clears nothing, since the menu names
no player.

See [Extensions](extension.md) for how the extension is negotiated.
