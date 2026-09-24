# Pie Menu

A pie menu is a ring that opens around the crosshair while a key is held. It is
divided into wedges, and a player picks one by flicking the aim towards it and
releasing the key — a direction rather than a word, chosen by hand and at speed,
without reading anything.

One ring is a **pie** and one wedge is a **slice**. Every pie has **exactly six
slices**, always, see [Six slices](#six-slices). A menu holds up to ten pies,
usually cycled in place while the key is held so that every slice stays one
gesture from the centre, and which of them are offered depends on what the
crosshair was on when the menu opened — terrain, a teammate, or an enemy.

Each slice sends one thing: a chat line, a ping on the world, or a command for
the server to read. None of it is new, since a player could type all of it. What
changes is that it takes one gesture, and that **the server chose what is on
offer**. A client shipping its own menu ships one set of phrases for every server
it will ever connect to, in one language, for whatever mode its author had in
mind; the server knows its mode, its commands and its vocabulary, so here it
sends the menu and the client draws it.

| ------------: | ------------- |
| Extension ID: | 49            |
| Packet ID:    | 113           |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

The packet id is `64 + extension id`, see
[Extension IDs](extension.md#extension-ids).

### Sub Packets:

| Sub ID | Name | Direction        | Size |
|--------|------|------------------|------|
| 0      | Menu | Server -> Client | 3+   |

Server to client only; a server that receives one drops it. Nothing here is a
request, so there is no permission bit and no way to ask for the menu again: a
server that wants no pie menu sends none, and one that changes its mind sends
another.

## Sub ID 0: Menu

The whole menu, every time. A Menu replaces the one the client was showing,
entire. There is no way to change a single pie or slice: a menu assembled from
fragments is one nobody has seen whole.

| Field Name    | Field Type | Example | Notes                                 |
|---------------|------------|---------|---------------------------------------|
| Packet ID     | UByte      | `113`   | Always `113`.                         |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet.       |
| Pie Count     | UByte      | `4`     | Pies that follow, `0`-`10`.           |
| Pies          | Pie[]      |         | The remaining bytes, see [Pie](#pie). |

The count is stated rather than implied by the packet length: pies nest slices
and slices carry lengths, so one wrong length byte would otherwise eat the pie
behind it and the packet would still parse.

### Pie

One ring, captioned in the middle by its theme.

| Field Name       | Field Type | Example    | Notes                                   |
|------------------|------------|------------|-----------------------------------------|
| Contexts         | UByte      | `0b011`    | Where it is offered, see [Contexts](#contexts). |
| Theme Message ID | UByte      | `0`        | Reserved. Must be `0`, see [Room for predefined messages](#room-for-predefined-messages). |
| Theme Length     | UByte      | `6`        | Bytes of Theme, `0`-`32`.               |
| Theme            | UTF-8 text | `"Social"` | The centre caption. May be empty.       |
| Slices           | Slice[6]   |            | Exactly six, see [Slice](#slice).       |

There is no slice count on the wire. Six is fixed by this specification, so the
six slices simply follow the theme and a pie is `3` bytes plus its theme plus its
six slices.

Pies are offered in the order sent, within each context. Which opens first is the
client's business. An empty Theme means no caption; the client invents none.

### Six slices

Every pie has six, on every server and in every client. The gesture is the
feature: a player reaches for a direction, and a direction only means something
if it means the same thing every time. A count the server could set would move
every slice on the ring each time it changed, so the phrase a player has learnt
to flick up-right for would sit somewhere else on the next pie, on the next
server, or after the next Menu packet.

A server wanting fewer than six things on a ring fills the spare wedges with
[`NONE`](#actions), which leaves everything else where the player learnt it. A
server wanting more uses another pie, up to the ten a menu holds.

### Contexts

What the crosshair is on when the menu opens decides which pies are offered. A
bitmask, so a ring that suits more than one context is sent once.

| Bit | Name       | Offered with the crosshair on                      |
|-----|------------|----------------------------------------------------|
| 0   | `WORLD`    | Terrain, the sky, or nothing in particular.        |
| 1   | `TEAMMATE` | A player on the opening player's own team.         |
| 2   | `ENEMY`    | A player on the other team.                        |
| 3-7 | reserved   | Must be `0`. Clients **must** ignore unknown bits. |

The context is fixed when the menu opens and does not change while it is held:
the aim is free to leave, and a message meant for somebody must not need them
held under the crosshair. A context with no pies has no menu — the key does
nothing there, and no other context's rings stand in.

### Slice

One wedge. It draws a Label and sends exactly one thing.

| Field Name   | Field Type | Example   | Notes                                      |
|--------------|------------|-----------|--------------------------------------------|
| Action       | UByte      | `1`       | See [Actions](#actions).                   |
| Message ID   | UByte      | `0`       | Reserved. Must be `0`, see [Room for predefined messages](#room-for-predefined-messages). |
| Flags        | UByte      | `0b0`     | See below.                                 |
| Text Length  | UByte      | `5`       | Bytes of Text, `0`-`128`.                  |
| Text         | UTF-8 text | `"/apoc"` | What it sends.                             |
| Label Length | UByte      | `4`       | Bytes of Label, `0`-`48`.                  |
| Label        | UTF-8 text | `"Nuke"`  | What it draws. **Empty means draw the Text.** |

| Bit | Name     | Meaning                                                      |
|-----|----------|--------------------------------------------------------------|
| 0   | `GLOBAL` | Goes to the whole server rather than to the team.             |
| 1-7 | reserved | Must be `0`. Clients **must** ignore unknown bits.            |

Two strings because `"Nuke"` is what a player looks for and `"/apoc"` is what the
server must receive. Where they are the same, which is every chat slice and every
ping, the Label is empty and costs one byte.

`GLOBAL` is the team/global bit of the base
[Chat Message](../protocol075.md#chat-message) and nothing more, kept per slice
so that one ring may hold a line for the room and a line for the team. One bit is
all it needs, because those are the only two channels there are. **There is no
enemy channel**: a line aimed at the player under the crosshair is a line the
whole server reads. So the [`ENEMY`](#contexts) context decides which pies are
offered, never where their words go.

### Actions

| Value     | Name      | Choosing the slice sends                                   |
|-----------|-----------|------------------------------------------------------------|
| `0`       | `NONE`    | Nothing at all, see [Empty slices](#empty-slices).         |
| `1`       | `CHAT`    | Text, as a [Chat Message](../protocol075.md#chat-message). |
| `2`       | `PING`    | A [Teamplay Ping](teamplay.md#sub-id-1-ping), Text as its Reason. |
| `3`       | `COMMAND` | Text, as a chat message the server reads as a command.     |
| `4`-`255` | reserved  | Refuses the menu, see [Validation](#validation).           |

**`PING`** marks the world position the crosshair was on **when the menu opened**,
not where it points when the slice is chosen — the marker belongs to the place
the player called out as they reached for the menu. The Text is the ping's
Reason, and may be empty for a neutral marker.

It needs [Teamplay](teamplay.md) negotiated with its `PING` bit set. Without
either, the client sends the Text as chat instead, so the words still arrive and
only the marker is lost. Who sees a ping is the server's decision, so `GLOBAL`
governs the fallback alone.

**`COMMAND`** always goes on the team channel, whatever `GLOBAL` says: a command
is not speech and does not belong in the room. On the wire it is an ordinary chat
packet — the server's command language is its own, and the client interprets none
of it, see [Command slices](#command-slices).

### Empty slices

**The client does nothing with an empty slice.** It sends no packet, shows no
message, plays no sound and reports no error — choosing one is the same as
releasing the key over the dead centre. How an empty wedge looks is the client's
business, but it must not stand in for a slice that is missing by inventing one.

A `CHAT` or `COMMAND` slice with an empty Text has nothing to send and is an
empty slice too, whatever its Action says: a chat packet carrying no words is
noise in the room and an empty command is noise at the server. A `PING` with an
empty Text is not — the Text is only its Reason, and a ping with no reason is the
neutral marker [Teamplay](teamplay.md#sub-id-1-ping) defines.

### Substitutions

The Text of a `COMMAND` slice may name the player the menu was opened on. A
command acting on a player has to be told which one, and a menu that could not
fill that in would leave the player reading an id off the screen and typing it,
which is the work a menu exists to remove.

| Token | Expands to                                                            |
|-------|-----------------------------------------------------------------------|
| `%p`  | The id of the player the menu was opened on, in decimal, without `#`.  |
| `%%`  | A literal percent sign.                                               |

So `/votekick #%p` reaches the server as `/votekick #7`, the command being the
server's own and its spelling with it. The client expands once, immediately
before sending, and never re-scans the result, so a name that looks like a token
is not one.

**A player is the only thing worth substituting.** Everything else a command
might want from where the player is standing, the server can work out for itself
at the moment the command arrives — the block under the crosshair above all,
which it raycasts from the position and orientation it already receives, the way
[Teamplay](teamplay.md#sub-id-1-ping) has it validate a ping. The player the menu
was opened on is the one value it cannot recover: the menu fixed it when it
opened and the aim has been free to leave ever since, so a raycast taken when the
command lands finds somebody else, or nobody.

### Notation

Substitutions are expanded in `COMMAND` Text and nowhere else. In a `CHAT` or
`PING` Text a `%` is a percent sign and nothing more, which is what
[Teamplay](teamplay.md) says of free text everywhere else: prose is never a
template, and naming a player inside prose belongs to the catalogue, where an id
arrives as a name each reader sees in their own language.

* `%%` is a literal percent sign, the same rule the catalogue follows.
* A `%` followed by a digit is **never** a substitution. That is the catalogue's
  positional form — `%1$p` — and this notation stays out of its way.
* A `%` followed by anything else is reserved, and version 1 leaves both
  characters as they are.

A `COMMAND` slice whose Text contains `%p` has nothing to expand in the `WORLD`
context. The client **must not** send it unexpanded; it treats the slice as
empty, see [Empty slices](#empty-slices). That is a server's mistake, not a
malformed packet, so the rest of the menu stands.

## Replacing the client's menu

* **No Menu has arrived.** The client keeps its own. Negotiating the extension
  says a menu can be sent, not that one has been.
* **A Menu with pies.** It replaces the client's own entirely. The two are never
  merged and never both reachable: a menu half chosen by the server and half by
  the client is one the player cannot tell apart.
* **A Menu with a Pie Count of `0`.** There is no pie menu here. The key does
  nothing, and the client's own menu does not come back.

A client **must not** add slices, reorder them, or drop one it dislikes. It may
refuse the menu outright, but it may not show a menu that claims to be the
server's and is not.

### Changing the menu mid-game

A server sends a Menu whenever it likes and as often as it likes: at the join, on
a map rotation, when a round starts, when a mode changes what there is to say,
when a player takes an objective or is handed admin rights. Each one replaces the
whole menu, so there is no state to reconcile and no order to get right — the
last Menu to arrive is the menu, and a server changing one ring resends the ten.

The client applies a new Menu **the next time the menu is opened**, not the
instant it arrives: a ring that changed under a held thumb would send a phrase
the player never read, on a slice they had already aimed at.

## Geometry

* Slice `0` is at the top, the rest **clockwise**.
* The six slices divide the ring evenly, `60` degrees each, so slice `0` is
  centred on straight up and slice `3` on straight down.
* The centre selects nothing, so releasing the key without choosing is always
  possible.

Everything else — radius, colour, animation, how rings are cycled, hold or toggle
— is the client's and is not on the wire. Servers should spend the layout on the
player: opposites on the vertical axis, the same phrase in the same slot on every
pie that carries it, the ordinary answers on the first pie of each context.

## Validation

A client parses the whole packet before applying any of it, and **a menu is
applied whole or not at all**. Where one is refused the previous menu stands and
the client logs it rather than disconnecting. A Menu is refused when:

* The packet is longer than a Menu can be, see [Limits](#limits).
* A record runs past the end of the packet, or bytes remain after the last pie.
* Pie Count is above `10`, or does not match the pies that follow, each of which
  carries exactly six slices.
* A Theme, Text or Label exceeds its cap, see [Limits](#limits), or is not
  well-formed UTF-8.
* An Action is `4` or above, or a reserved bit is set in Flags or Contexts.
* A Message ID is not `0`, see
  [Room for predefined messages](#room-for-predefined-messages).

Strings are UTF-8, consistent with [UTF-8 Chat](utf-8-chat.md), and drawn as they
arrive. A client strips control characters and line breaks — a Theme is one
caption, a Label one line — and treats an all-whitespace string as empty, which
every field allows. A client applies its ordinary chat rate limit to what a menu
sends.

## Command slices

A `COMMAND` slice sends text, and that is the whole of it. The client does not
interpret the Text, does not look for a leading `/`, and has no local command
language reachable from here: what leaves is a chat packet. A server cannot reach
into a client through this extension, because there is nothing on this side to
reach.

What it can do is put words in a player's mouth unread, since the Label is drawn
and the Text is sent. So the client **shows the player what went out in their
name**: the expanded Text is echoed in its own chat view as an outgoing line.
Where and how is the client's business; that it happens is not.

## Room for predefined messages

Version 1 carries its words in the packet, so every player reads them in the
language the server wrote them in. Fixing that belongs to
[Teamplay](teamplay.md) — a catalogue of short phrases with a fixed id each,
translated by every client. None of that is here yet. **The two bytes it needs
are.**

The `Message ID` on each [Pie](#pie) and each [Slice](#slice) is where a
catalogue id will go: on a slice, the phrase it says and draws; on a pie, the
heading in the middle of the ring. A later version spends those bytes without
moving anything.

Because every string here is length-prefixed, a slice can carry an id **and** the
words to use instead, so a version 2 server fills in both for a version 1 client.
That is why version 1 refuses a non-zero id rather than ignoring it: a client
meeting one would have to guess whether the Text beside it was a fallback or a
leftover, and no version has to. The server knows which it is talking to from the
version byte [`ExtInfo`](extension.md#extinfo-packet) carried.

Two things that version must settle:

* **The catalogue has no headings.** It holds phrases players say, nothing that
  reads as the name of a ring. Either a block of headings is reserved before the
  ids freeze, or a theme stays untranslated while its slices do not.
* **Parametric entries have no value to carry.** A phrase taking a number has
  nowhere to get one in a menu written before the round, and no bytes are
  reserved for one. A phrase taking a player is different: the client already
  knows who the menu was opened on, so that value costs nothing on the wire and
  is the natural first to support.

## Limits

| Thing         | Cap   | Why                                                      |
|---------------|-------|----------------------------------------------------------|
| Pies per menu | `10`  | Rings are cycled one at a time, see below.               |
| Theme         | `32`  | One caption, read at a glance from the middle of a ring. |
| Label         | `48`  | One line inside a wedge.                                 |
| Text          | `128` | A command with arguments, or a sentence.                 |

The string caps are in bytes, not characters, so a parser can enforce them before
decoding anything.

Ten pies because rings are reached by cycling through them, one flip at a time,
so the tenth is already further away than anything on it is worth.

These caps bound the packet without a cap of its own: a slice is at most `181`
bytes, a pie at most `1121`, and a Menu at most `11213`. A client may refuse a
longer packet without parsing it. A realistic menu — six pies — is under two
kilobytes.

## Connection state

The menu belongs to the connection, not to the world and not to a player.

* [Map Start](../protocol075.md#map-start-075) does not clear it, the rule the
  [Teamplay Config](teamplay.md#sub-id-0-config) already follows. A server whose
  menu differs between maps sends a new Menu after the change.
* [Player Left](../protocol075.md#player-left) clears nothing. The menu names no
  player: `%p` resolves against whoever the crosshair is on as the menu opens, so
  a recycled id cannot be held here.
* Nothing is per-player, so there is nothing to free and nothing to remember.

## Version growth

Version 1 defines sub id `0`. A receiver drops a sub id it does not know, so a
later version may add sub-packets — but not change the layout of `0`, which is
what makes that safe. Inside the Menu the room is in the reserved bytes and bits:
the two [Message ID](#room-for-predefined-messages) bytes, Flags `1`-`7`,
Contexts `3`-`7`, and Actions `4`-`255`. A version 1 client refuses a menu using
any of them, so a server sends each client the menu its version can read — and
every version can describe the same rings, because the words are in the packet.

The [six slices](#six-slices) are not among that room. There is no field to grow
and no value to spend: a version that wants rings of another size says so in the
version byte and the whole server moves together.

See [Extensions](extension.md) for how the extension is negotiated.
