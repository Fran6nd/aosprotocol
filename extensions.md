## Extension Negotiation Packet

### Overview

Each extension is given an unique id that is decided when it is first
registered. We differentiate between two types of packets:

*Those that:*

| Type          | Purpose                               | Extension id range |
| ------------- | ------------------------------------- | ------------------ |
| `HAS_PACKETS` | introduce new packets to the protocol | 0-191              |
| `PACKETLESS`  | don't use and need any packets        | 192-255            |

Each extension is given one legacy packet id equal to `64+extension_id`.
For `PACKETLESS` extensions this would mean that their packet ids are out
of spec `>255`, thus they don't have any.
An example for a packetless extension would be *OpenSpades'* UnicodeExt.

Each extension packet will contain 1 additional byte in its data, which is a
subpacket id, used to have multiple packets available for each extension. This
is always the case, even if an extension only needs 1 packet in total.

General extension packet structure:

| Field name    | Type      | Notes          |
| ------------- | --------- | -------------- |
| Packet id     | UByte     | 64-255         |
| Sub packet id | UByte     | 0-255          |
| Data          | UByte[]   | extension data |

### ExtInfo Packet

* Packet ID: 60
* Total size: `2+2*length`

| Field name | Field type   | Notes                        |
| ---------- | ------------ | ---------------------------- |
| length     | UByte        | `length` entries will follow |
| entries    | ExtInfoEntry | see below                    |

**ExtInfoEntry**

| Field name | Field type | Notes               |
| ---------- | ---------- | ------------------- |
| ext. ID    | UByte      | see #Overview       |
| version    | UByte      | Usually starts at 1 |

## Protocol Flow

The server should send an `ExtInfo` packet (optimally) after the Version Info response has been received to compatible clients
(OpenSpades versions > 0.1.3, see https://github.com/piqueserver/piqueserver/issues/504),
assuming it supports any. The client can store the list of extensions for later use and should
reply with an `ExtInfo` packet that lists the extensions it supports (if it does actually support any).

The client can omit any extensions that the server does not support from its
reply, but this is not necessary as the server can simply ignore them itself.


# HAS_PACKETS Packets
* [Player Properties](#player-properties)
* [Teamplay](#teamplay)

## Player Properties

Sends additional player attributes from the server to the client.

| ---------: |-----|
| Packet ID: | 64  |
| Version:   | 1   |

| Sub ID | Name              | Direction        | Size |
|--------|-------------------|------------------|------|
| 0      | Player Properties | Server -> Client | 12   |

### Sub ID 0: Player Properties

The server sends this packet to inform the client of authoritative stats for
a single player. There is no client-to-server form. The 12-byte total
includes the Packet ID and Sub Packet ID as described in the
[general extension packet structure](#overview).

| Field Name    | Field Type | Example | Notes                                                                                                     |
|---------------|------------|---------|-----------------------------------------------------------------------------------------------------------|
| Packet ID     | UByte      | `64`    | Always `64`.                                                                                              |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet.                                                                           |
| Player ID     | UByte      | `0`     | The id of the player these stats describe.                                                                |
| HP            | UByte      | `100`   | Health. A `UByte`, so the wire range is `0`..`255`; vanilla servers use `0`..`100`. Whether `0` HP marks the player as dead is client-specific (betterspades does not — the kill packet is authoritative for death; openspades does, but does not support this packet). |
| Blocks        | UByte      | `50`    | Block count currently carried by the player. Typically `0`..`50` for vanilla servers.                     |
| Grenades      | UByte      | `3`     | Grenade count currently carried by the player. Typically `0`..`3` for vanilla servers.                    |
| Magazine Ammo | UByte      | `10`    | Ammunition currently loaded in the player's equipped weapon clip.                                         |
| Reserve Ammo  | UByte      | `50`    | Spare ammunition the player has for the equipped weapon (outside the clip).                               |
| Score         | LE Uint    | `7`     | The player's score, as defined by the server (e.g. kills, or kills plus objective points). 32-bit little-endian integer. Reference clients treat it as unsigned (BetterSpades, KyroSpades); zerospades reads it into a signed `int`. Signedness is only observable for values >= 2^31. |

Byte layout (offsets are within the packet, including the Packet ID byte):

| Offset | Size | Field         |
|--------|------|---------------|
| 0      | 1    | Packet ID     |
| 1      | 1    | Sub Packet ID |
| 2      | 1    | Player ID     |
| 3      | 1    | HP            |
| 4      | 1    | Blocks        |
| 5      | 1    | Grenades      |
| 6      | 1    | Magazine Ammo |
| 7      | 1    | Reserve Ammo  |
| 8      | 4    | Score         |

## Teamplay

Gathers the features a team needs to play together and a client cannot provide on
its own: seeing teammates through walls, marking a place in the world, being
pointed at a player by the server, and saying a short thing to the team in a
language every player reads in their own. Each client-side feature is permitted
independently by the server through a feature bitmask, so a server can enable any
combination (or none) of them.

| ---------: |-----|
| Packet ID: | 66  |
| Version:   | 1   |

The extension id, the number carried by `ExtInfo`, is `2`; the packet id is
`64 + extension id` as described in [#Overview](#overview).

#### Sub Packets:

| Sub ID | Name     | Direction         | Size |
|--------|----------|-------------------|------|
| 0      | Config   | Server -> Client  | 3    |
| 1      | Ping     | Client <-> Server | 21+  |
| 2      | ESP Mark | Server -> Client  | 9+   |
| 3      | Message  | Client <-> Server | 6    |

A client sends two of these and no others: a **Ping** to point at a place, and a
**Message** to say one of the [predefined messages](#predefined-messages). They
are separate packets because they are separate acts — a message is not a marker
that happens to carry words, and most of the catalogue points at nothing. A
message id may also label a Ping, which is the marker being named, not a second
way of speaking.

Config and ESP Mark are the server speaking; a server that receives either from a
client drops it. Whatever a client sends is a request: the server decides whether
it happens at all, who it reaches, and how it is presented.

### Sub ID 0: Config

The server announces which features are permitted. The server MAY send this
sub-packet at any time; the client applies the new bitmask immediately (for
example to change policy on a map rotation). A feature is only available to the
client while its bit is set.

| Field Name    | Field Type | Example | Notes                                              |
|---------------|------------|---------|----------------------------------------------------|
| Packet ID     | UByte      | `66`    | Always `66`.                                       |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet.                    |
| Features      | UByte      | `0b110` | Feature bitmask, see below.                        |

Feature bitmask:

| Bit | Name           | Meaning                                                                 |
|-----|----------------|-------------------------------------------------------------------------|
| 0   | `TEAM_ESP`     | Client may render teammates through walls, see [What ESP renders](#what-esp-renders). |
| 1   | `PING_WORLD`   | Client may render received pings as 3D markers in the world.            |
| 2   | `PING_MINIMAP` | Client may render received pings on the minimap.                        |
| 3   | `QUICK_CHAT`   | Client may send Message packets, which is its quick chat menu.          |
| 4-7 | reserved       | Must be `0`. Clients must ignore unknown bits.                          |

`TEAM_ESP` only renders teammate positions the client already receives from the
base protocol, so it discloses nothing new; the bit is a fair-play policy toggle,
not a data gate.

`PING_WORLD` and `PING_MINIMAP` are independent and not alternatives. Setting
both — the expected configuration — means the same ping is drawn in the world
*and* on the minimap at once, which is how a marker is both something you turn
towards and something you locate on the map. Either one alone is equally valid:
world only for a server that wants players to look up rather than down, minimap
only for one that wants no marker cluttering the view.

Each of the two client-sent packets has its own switch. If both `PING_WORLD` and
`PING_MINIMAP` are clear the client must not send Ping packets, and the server
must ignore any it receives; if `QUICK_CHAT` is clear the same goes for Message
packets. A server can therefore run pings without quick chat, quick chat without
pings, or neither.

A client always renders what the server sends it, exactly as it always renders
received chat. No bit turns that off; the server simply stops sending.

After the server changes the config, a Ping the client sent before receiving the
new bitmask may still be in transit. The server simply drops such pings; this is
not a protocol violation and needs no special handling.

### Durations

Pings and ESP marks both carry a lifetime, encoded the same way: an `LE float32`
number of seconds, counted by the client from the moment it receives the packet.

| Value             | Meaning                                                             |
|-------------------|---------------------------------------------------------------------|
| `0`               | Remove: clears the ping or the mark this packet refers to.          |
| positive, finite  | Lifetime in seconds, after which the client removes it by itself.   |
| `+inf` (`0x7F800000`) | Stays until the server removes it or the target leaves.         |
| negative, NaN     | Invalid. The receiver drops the packet.                             |

A float because both ends of the range are real: a spotting ping worth 1.5
seconds and an objective marker worth an hour are written the same way, with no
unit to agree on and no rounding.

The point of the encoding is to let a server pick how much work it wants to do. A
server with nothing special in mind sends a finite duration and forgets the whole
thing: no timer, no removal packet, no state. A server that wants a marker to
follow its own logic sends `+inf` and removes it when its logic says so, which is
the on/off behaviour with no extra packet type. Both spellings cost the same four
bytes.

A server keeping `+inf` marks must remember them anyway, if only to send them to
players who join later; a server using finite durations should re-send an active
mark to a joining client with the time that is left, not the time it started
with. Pings are not re-sent: they are events, not state, and a player who was not
there when the ping was made has nothing to catch up on.

Expiry itself is always client-side. A client may cap how many pings and marks it
displays at once, dropping the oldest first, and is never obliged to render an
absurd number of them.

### Sub ID 1: Ping

A single packet used in both directions. A client sends it to ping the world
position its crosshair points at, naming the player it is aiming at and the label
it wants; the server validates all of that and relays it to the players of its
choice, filling in the originating Player ID. The server may also originate a
Ping on its own — objective markers, scripted events, admin callouts — with no
client involved. The position uses the same coordinate frame and `LE float32`
encoding as the base protocol position packets.

| Field Name    | Field Type | Example     | Notes                                              |
|---------------|------------|-------------|----------------------------------------------------|
| Packet ID     | UByte      | `66`        | Always `66`.                                       |
| Sub Packet ID | UByte      | `1`         | Always `1` for this sub-packet.                    |
| Player ID     | UByte      | `0`         | On relay, the player that pinged. `255` means the ping originated from the server itself. Ignored on the client -> server direction; the server fills it in authoritatively. |
| X             | LE float32 | `256.0`     | World X coordinate.                                |
| Y             | LE float32 | `256.0`     | World Y coordinate.                                |
| Z             | LE float32 | `40.0`      | World Z coordinate.                                |
| Duration      | LE float32 | `5.0`       | Display time, see [Durations](#durations). Server -> client only: a client sends `0` here and the server, which is authoritative, fills it in. |
| Message ID    | UByte      | `33`        | [Predefined message](#predefined-messages) the ping carries, `0` for none. |
| Reason        | UTF-8 text | `""`        | Free-form label, used only when Message ID is `0`. |

A player has one active ping at a time: a new ping from the same Player ID
replaces the previous one and restarts its lifetime, and a Duration of `0`
removes it without placing another. The server's own pings (Player ID `255`)
behave the same way, so a server that wants several permanent markers at once
needs ESP marks or a ping per originating id, not several pings from `255`.

A ping always points somewhere. A player who wants to say something without
pointing at anything sends a [Message](#sub-id-3-message) instead — that is what
it is for, and it is why the two are separate packets.

A ping carries no audience. The client has no way to ask for team only, for the
whole server, or for one player, because it has no business knowing: it points at
a place and the server decides who is shown it. The same is true of the label —
a client attaches the id it likes, and the server keeps it, replaces it or drops
the ping entirely.

The label of a ping is either a catalogue id or a free string. When Message ID is
not `0` the client renders the [predefined message](#predefined-messages) for
that id, translated, and ignores Reason, which should then be empty. When it is
`0` the client renders Reason, and an empty Reason is a neutral "look here"
marker.

Which id a client attaches to a ping is the client's own business. The catalogue
groups its entries by what they usually mean, not by what may label a marker, and
this extension puts no restriction on the combination. The one thing a client may
not do is invent ids: only the ones the catalogue defines.

The Reason occupies the remaining bytes of the packet (no length prefix or
terminator); its length is implied by the packet length, and it may be empty. It
is a free-form, client-defined UTF-8 string, consistent with the
[UTF-8 Chat](#utf-8-chat) convention. Before relaying, the server must validate
it as well-formed UTF-8 and drop (or sanitise) anything malformed rather than
forwarding bytes verbatim. The server should cap its length, truncating on a
codepoint boundary if needed. Clients render the string as received and fall back
to a neutral marker for anything they do not recognise.

A client is free to ping with a Reason it typed rather than a catalogue id, and
the field exists precisely so that anything the catalogue does not cover can
still be said — a place name, a plan, a joke, a language the catalogue was never
going to hold. The cost is that **free text falls out of translation**: it
travels as the bytes the sender wrote and every reader sees those same bytes,
in the sender's language, however many of them speak it. A catalogue id is read
by each player in their own language; a Reason string is read by whoever happens
to share the writer's. Clients should prefer an id whenever one fits, and servers
that want a room where everybody understands everybody can clear the Reason and
keep the id.

A dead player does not ping. A client must not send a Ping while its player is
dead, and the server drops any it receives from a dead player, exactly as it
drops one sent while the feature is switched off. Waiting to respawn is not a
vantage point, and a corpse pointing at things the living cannot see is a way of
spectating the enemy. Whether spectators may ping is the server's call.

To identify the target of a ping, the server performs a raycasting check from the
client's position through their crosshair direction, rather than relying on client
data.

Server handling of a client -> server Ping:

* The server should rate-limit requests (a sane default is at most one per player
  per second).
* The label a client asks for (Message ID, or Reason) is a request like the rest:
  the server may replace it, empty it, or drop the ping over it.
* The server should sanity-check the coordinates (at least the map bounds of
  512 x 512 x 64, optionally a line-of-sight or solid check). The server is
  authoritative on placement.
* Who receives the relayed Ping is left to the server's discretion (team only,
  everyone, spectators, etc.); the protocol does not mandate a distribution
  policy. Relaying it to nobody is a valid decision too.

How long a ping stays is the server's call alone, never the pinging client's:
Duration is only meaningful on the relay. A client asks for a ping, the server
decides what that ping is worth — `5.0` seconds is the conventional value and
what a server with no opinion should send, while a server building something of
its own can make a ping last a round or vanish in half a second, without the
client needing to know that policy or the server needing a removal packet.

### Sub ID 2: ESP Mark

Marks a player as visible through walls to whoever receives the packet. Unlike
`TEAM_ESP`, which lets a client reveal its own team on its own initiative, a mark
is decided by the server, targets one player, and is unrelated to teams: the
server can reveal a player to their own team, to the other team, to everyone, or
to a single player. That covers spotting the scoreboard leader, showing a hunted
player to the hunters, exposing a cheater to the whole server, or a game mode that
reveals a carrier while they hold the intel.

This sub-packet travels **server -> client only**. Unlike the Ping and the
Message, it has no client-sent form: a client never asks for a mark and never
sends sub-packet `2`. A server receiving it from a client must drop it, since the
only thing such a packet can be is an attempt to reveal a player to somebody the
server did not choose.

| Field Name    | Field Type | Example    | Notes                                          |
|---------------|------------|------------|------------------------------------------------|
| Packet ID     | UByte      | `66`       | Always `66`.                                   |
| Sub Packet ID | UByte      | `2`        | Always `2` for this sub-packet.                |
| Player ID     | UByte      | `7`        | The player to reveal.                          |
| Duration      | LE float32 | `10.0`     | Lifetime, see [Durations](#durations).         |
| Flags         | UByte      | `0b1`      | Lifetime modifiers, see below.                 |
| Message ID    | UByte      | `0`        | [Predefined message](#predefined-messages) labelling the mark, `0` for none. |
| Reason        | UTF-8 text | `"leader"` | Free-form label, used only when Message ID is `0`. |

Flags:

| Bit | Name               | Meaning                                                                     |
|-----|--------------------|-----------------------------------------------------------------------------|
| 0   | `CLEAR_ON_RESPAWN` | The mark ends the next time the marked player spawns.                       |
| 1   | `SHOW_NAME`        | The client shows the marked player's name. Clear, it shows the outline alone.|
| 2-7 | reserved           | Must be `0`. Clients must ignore unknown bits.                              |

`CLEAR_ON_RESPAWN` is keyed to the spawn rather than to the death: any
[Create Player](protocol075.html#create-player) for that id ends the mark,
whether the player was killed, changed team, changed weapon or was moved by a
script, and the client needs no death bookkeeping to implement it. A mark on a
player who dies and stays dead lasts until they come back, which is what
"reveal them until they get away" wants anyway.

`SHOW_NAME` is the server's call, not the client's, because only the server knows
what it is revealing. Marking the scoreboard leader for their own team is worth a
name; marking an enemy for the team hunting them may not be, when the outline
says where and the name would hand over who. A client shows the name when the bit
is set and must not when it is clear — a mark is an instruction, and this part of
it is no different.

Duration and the lifetime flag are independent, so every lifetime a server is
likely to want falls out of the same five bytes:

| Intent                                      | Duration | Flags              |
|---------------------------------------------|----------|--------------------|
| Reveal for a while                          | `3.5`    | `0`                |
| Reveal until the server clears it           | `+inf`   | `0`                |
| Reveal until they respawn                   | `+inf`   | `CLEAR_ON_RESPAWN` |
| Reveal for a while, or until they respawn   | `3.5`    | `CLEAR_ON_RESPAWN` |
| Clear the mark now                          | `0`      | `0`                |

`SHOW_NAME` is orthogonal to all of these and may be set with any of them.

The label works exactly as on a Ping: a non-zero Message ID renders the
[predefined message](#predefined-messages) for that id, translated, and Reason is
then ignored and should be empty. Otherwise the client renders Reason, which
follows the Ping's rules — remaining bytes of the packet, no length prefix or
terminator, validated by the server as well-formed UTF-8, capped and truncated on
a codepoint boundary — and falls back to a neutral highlight for anything it does
not recognise. `"cheater"`, `"leader"` and `"carrier"` are examples, not assigned
values. Both empty is valid and is the common case: the packet is then 9 bytes and
the client shows the player highlighted with no label.

The audience is the set of clients the server sends the packet to; there is no
audience field. A field would have to be enforced by the client, and a client
that ignores it would reveal players it was never meant to see — the same reason
the Ping relay leaves distribution to the server. It also means a player is not
told they are being revealed to others unless the server includes them in the
recipients, which is what the punishment case wants; a client that receives a
mark for its own player id may show it as a "you are marked" indicator.

A mark is an instruction rather than a permission, so the client renders it
whatever the `TEAM_ESP` bit says, and marks are not gated by the Config bitmask.
A client with `TEAM_ESP` clear still shows a marked teammate.

#### What ESP renders

The two paths are not held to the same rendering, because one is the client
showing its own team and the other is the server pointing at somebody.

`TEAM_ESP` is left to the client: a body outline, a box around the player, a
chevron above them, a dot on the edge of the screen — whatever the client
considers good practice — as long as it is drawn in that player's team colour,
the one the server sent in [State Data](protocol075.html#state-data), so the
highlight reads as team information at a glance. Showing the name alongside is
recommended and also the client's call.

An **ESP Mark must draw the player's body outline through walls**, following the
body and its pose rather than standing in for it with a box, a dot or a floating
marker. The server marked one specific player, and the audience has to see where
exactly they are, not merely that somebody is around there.

Whether the name goes with it is the server's decision, carried by the
`SHOW_NAME` flag: set, the client shows the marked player's name; clear, it shows
the outline and nothing else. A mark has no imposed colour, though — the client
is free to distinguish it from a plain teammate highlight, which is also what the
label is there for.

The rest is the client's business in both cases: outline thickness, opacity,
whether the highlight fades with distance, and how the label is rendered beside
it, if at all.

Marks are state held per player id, and one mark per player: a new mark replaces
the previous one and restarts its timer. A mark is dropped when its Duration
expires, when a mark with Duration `0` arrives for that id, when its target
spawns and `CLEAR_ON_RESPAWN` is set, or under the general rules of
[Per-player state](#per-player-state). Without `CLEAR_ON_RESPAWN` it survives
death and respawn, so a punishment mark does not have to be re-sent every time
its target is killed.

### Sub ID 3: Message

Says one of the [predefined messages](#predefined-messages) by its catalogue id.
This is how a player speaks: the client sends the id it wants, the server decides
what to do with it and relays it to the players it chooses, and each of them
renders it in their own language.

A single packet used in both directions, as for the Ping. Nothing but ids travels
here — there is no text field at all, so a client cannot smuggle words through it,
and everything it can say is something every other client can already read.

| Field Name    | Field Type | Example | Notes                                                                   |
|---------------|------------|---------|---------------------------------------------------------------------------|
| Packet ID     | UByte      | `66`    | Always `66`.                                                              |
| Sub Packet ID | UByte      | `3`     | Always `3` for this sub-packet.                                           |
| Player ID     | UByte      | `4`     | On relay, the player speaking; `255` means the server itself. Ignored on the client -> server direction; the server fills it in authoritatively. |
| Target ID     | UByte      | `9`     | The player the message is addressed to, `255` for nobody in particular. Set by the client. |
| Chat Type     | UByte      | `1`     | Where the message lands, see below. Server -> client only: a client sends `0` and the server assigns it. |
| Message ID    | UByte      | `52`    | Catalogue id, `1`-`255`. `0` carries no message and must not be sent.     |

**Target ID** is what makes the *"behind you"* family readable: every client
knows who "you" is, the addressed player sees the message aimed at them and the
others read it as aimed at that name. It says nothing about who receives the
packet, which is a separate decision the server makes by choosing recipients.

**Chat Type** is how the message reaches the player, and the server is the only
one who chooses it. A client asks to say something; it does not decide whether
its words reach its team, the whole server, or one player, any more than it
decides who sees its pings. The values are those of the base
[Chat Message](protocol075.html#chat-message) packet, plus the
[Message Types](#message-types) ones and one this extension adds:

| Value | Name           | Lands as                                                     |
|-------|----------------|--------------------------------------------------------------|
| 0     | `CHAT_ALL`     | A global chat line.                                          |
| 1     | `CHAT_TEAM`    | A team chat line.                                            |
| 2     | `CHAT_SYSTEM`  | A server line.                                               |
| 3-6   | Message Types  | Big, info, warning, error — the coloured and centred forms.  |
| 7     | `CHAT_DIRECT`  | A message to the Target ID alone, rendered as a private one. |

Values `3`-`6` need the [Message Types](#message-types) extension; a server sends
them only to clients that negotiated it and downgrades to `0` or `2` for the
rest. `7` is defined here, so it needs only this extension.

A client renders each type where its existing chat already puts it — a chat line
in the chat window, `3`-`6` in whatever alert or centre-screen view it uses for
them — and adds nothing new. The extension changes what travels on the wire, not
where a message appears on screen.

The packet is a fixed 6 bytes and carries no text: the id *is* the message. That
is the whole point, and it is why a message costs 6 bytes where the same sentence
costs 20 to 60 as chat.

#### English fallback

The catalogue is normative on both ends, so either side can always fall back to
words when its peer cannot speak in ids:

* **Server without the extension.** The client has no Message packet to send, so
  its quick chat sends the [canonical English text](#predefined-messages) as an
  ordinary [Chat Message](protocol075.html#chat-message) — which is what such a
  menu does today anyway. Nothing is lost but the translation.
* **Client without the extension.** The server sends that client an ordinary Chat
  Message containing the canonical English text, with the base chat type matching
  what it chose, so the line still appears in the right place and colour. A
  `CHAT_DIRECT` message becomes whatever private form that server already uses.
* **Client with an older version of the extension.** `ExtInfo` carries a version
  per extension, so the server knows a client's catalogue. Ids beyond it are sent
  to that client as English text rather than as a Message packet.

All canonical strings are pure ASCII, so the fallback survives the base
protocol's CP437 chat encoding without needing the
[UTF-8 Chat](#utf-8-chat) convention.

A client that receives an id it does not know renders nothing for it. It must not
invent a placeholder line, and it must not disconnect.

#### Server handling

* Rate-limit messages as you rate-limit chat. A menu makes them cheap to spam;
  a sane default is the chat limit.
* The server may refuse individual ids, whole blocks or the whole catalogue — a
  server that wants no banter drops those ids and passes the rest on.
* Recipients and chat type are the server's alone: the same incoming message can
  go out as a team line to one side, a direct message to the player it names, and
  nothing at all to the enemy team.

### Predefined messages

The catalogue is a fixed table of short phrases, each with a one byte id. A
client sends the id — in a Message, or as the label of a Ping — and every client
renders the phrase in its own player's language; the canonical English text below
is what goes on the wire when a peer does not support the extension, and what a
client displays if it has no translation.

The table is the same for everybody, which is the only reason this works: a
French player sends `52`, a Japanese player reads *"援護してくれ"*, and neither
client learned anything from the other. It also means **ids are frozen forever**.
Version 1 of this extension defines the ids below; a later version may only
append into the reserved space. An id is never renumbered and never reused for
another meaning, because clients record demos of games they replay years later.

The entries are chosen for how Ace of Spades is actually played — blocks run out,
spades dig, tents restock, intel gets carried, and someone is always digging under
someone — rather than translated from a generic shooter. Ids are grouped in blocks
of sixteen, and each block keeps its tail free so a later version can extend a
category without scattering it. Ids not listed here are reserved and must not be
sent.

Id `0` is not a message. It is the "no message" value the Ping and ESP Mark
packets use, and it must never appear in a Message packet.

#### 1-15 — Answers and courtesy

| Id | English text | Id | English text |
|----|--------------|----|--------------|
| 1  | Yes          | 9  | Hello        |
| 2  | No           | 10 | Good Game    |
| 3  | Affirmative  | 11 | Nice Shot    |
| 4  | Negative     | 12 | Well Played  |
| 5  | Understood   | 13 | Wait         |
| 6  | Thank You    | 14 | Ready        |
| 7  | Sorry        | 15 | Not Ready    |
| 8  | My Bad       |    |              |

#### 16-31 — Numbers

| Id      | English text        |
|---------|---------------------|
| 16-25   | `1` … `10`          |
| 26      | Many                |
| 27-31   | reserved            |

A bare number usually answers "how many of them?", but the catalogue does not fix
that; it is the one entry whose meaning comes from what it answers. Clients
should render it exactly as the digits, which every language reads.

#### 32-47 — Enemy contact

| Id | English text    | Id | English text              |
|----|-----------------|----|---------------------------|
| 32 | Enemy Spotted   | 39 | They Are Pushing          |
| 33 | Enemies Here    | 40 | They Are Flanking         |
| 34 | Enemy Sniper    | 41 | Spawnkiller!              |
| 35 | Enemy Camping   | 42 | Clear                     |
| 36 | Enemy Tunneling | 43 | All Clear                 |
| 37 | Enemy Building  | 44 | They Are Digging Under Us |
| 38 | Enemy Tower     | 45 | Enemy In Our Base         |

#### 48-63 — Directions, relative to the addressed player

These are read against the **Target ID** of the message: "you" is that player, and
the direction is the one they are facing.

| Id | English text  | Id | English text     |
|----|---------------|----|------------------|
| 48 | Behind You    | 53 | In Front Of You  |
| 49 | On Your Left  | 54 | They See You     |
| 50 | On Your Right | 55 | Watch Your Back  |
| 51 | Above You     | 56 | Look Behind You  |
| 52 | Below You     |    |                  |

#### 64-79 — Directions, relative to the sender

These are read against the sender, the **Player ID** of the message. Splitting the
two families is deliberate: "on the right" is the oldest ambiguity in team games,
and here the id itself says whose right it is.

| Id | English text   | Id | English text    |
|----|----------------|----|-----------------|
| 64 | Behind Us!     | 71 | They Are Below  |
| 65 | On Our Left    | 72 | Over There      |
| 66 | On Our Right   | 73 | Up There        |
| 67 | Above Us       | 74 | Down There      |
| 68 | Below Us       | 75 | Everywhere!     |
| 69 | In Front Of Us | 76 | Right Here      |
| 70 | They Are Above |    |                 |

#### 80-95 — Requests

| Id | English text   | Id | English text     |
|----|----------------|----|------------------|
| 80 | Help Me        | 88 | Help Me Build    |
| 81 | Cover Me       | 89 | Give Me A Boost  |
| 82 | Follow Me      | 90 | Open The Wall    |
| 83 | Wait For Me    | 91 | Close The Wall   |
| 84 | Come Here      | 92 | Let Me Through   |
| 85 | I Need Blocks  | 93 | Guard This Spot  |
| 86 | I Need Ammo    | 94 | Heal Me          |
| 87 | I Need Health  | 95 | Dig Me Out       |

#### 96-111 — Orders and tactics

| Id  | English text          | Id  | English text     |
|-----|-----------------------|-----|------------------|
| 96  | Attack                | 104 | On My Command    |
| 97  | Defend                | 105 | Prepare For Assault |
| 98  | Fall Back             | 106 | Flank Left       |
| 99  | Regroup               | 107 | Flank Right      |
| 100 | Push Up               | 108 | Go Around        |
| 101 | Hold Position         | 109 | Rush Them        |
| 102 | Spread Out            | 110 | Take Cover       |
| 103 | All Together          | 111 | Kick Their Butt  |

#### 112-127 — Objective

| Id  | English text         | Id  | English text        |
|-----|----------------------|-----|---------------------|
| 112 | Get The Intel        | 119 | Capture This Point  |
| 113 | I Have The Intel     | 120 | Defend This Point   |
| 114 | They Have Our Intel  | 121 | Enemy Base          |
| 115 | Defend Our Intel     | 122 | Our Base            |
| 116 | Bring It Home        | 123 | Meet At Our Base    |
| 117 | I Am Escorting You   | 124 | Go Restock          |
| 118 | Cover The Carrier    | 125 | Restock First       |

#### 128-143 — Building and digging

| Id  | English text        | Id  | English text     |
|-----|---------------------|-----|------------------|
| 128 | Build Here          | 136 | Do Not Dig Here  |
| 129 | Dig Here            | 137 | Tear It Down!    |
| 130 | Build A Bridge Here | 138 | Nice Build       |
| 131 | Dig A Tunnel Here   | 139 | Watch The Fall   |
| 132 | Build A Wall Here   | 140 | I Am Building    |
| 133 | Build A Tower Here  | 141 | Block The Way    |
| 134 | Make Stairs Here    |     |                  |
| 135 | Fill This Hole      |     |                  |

#### 144-159 — Own status

| Id  | English text       | Id  | English text        |
|-----|--------------------|-----|---------------------|
| 144 | On My Way          | 152 | I Am Dead           |
| 145 | In Position        | 153 | Respawning          |
| 146 | I Follow You       | 154 | I Am Sniping Here   |
| 147 | I Am Low           | 155 | I Am Flanking       |
| 148 | I Am Out Of Blocks | 156 | I Am Digging In     |
| 149 | I Am Out Of Ammo   | 157 | Going Back To Base  |
| 150 | Reloading          | 158 | I Am Lost           |
| 151 | I Am Hit           |     |                     |

#### 160-175 — Warnings

| Id  | English text     | Id  | English text     |
|-----|------------------|-----|------------------|
| 160 | Grenade!         | 167 | Cave In!         |
| 161 | Watch Out!       | 168 | Do Not Shoot     |
| 162 | Look Up          | 169 | Hold Your Fire   |
| 163 | Look Down        | 170 | Friendly Fire!   |
| 164 | Careful          | 171 | Stop             |
| 165 | Do Not Go There  | 172 | Stop Griefing    |
| 166 | Falling Blocks!  | 173 | Do Not Fall      |

#### 176-191 — Reactions

| Id  | English text | Id  | English text |
|-----|--------------|-----|--------------|
| 176 | Lol          | 183 | Good Try     |
| 177 | Nice         | 184 | Revenge!     |
| 178 | Wow          | 185 | Camper       |
| 179 | Oops         | 186 | Too Easy     |
| 180 | Close One    | 187 | Rip          |
| 181 | Let's Go     |     |              |
| 182 | We Got This  |     |              |

This block is the one a server is most likely to filter, and filtering it is
expected: a server that considers `Too Easy` or `Camper` an invitation to
needling drops those ids and relays the rest. That decision belongs to the
server, which is why the catalogue carries them rather than pretending players
will not find a way to say them.

#### 192-207 — Places

Landmarks every Ace of Spades map has, whatever it looks like. These pair
naturally with a Ping, which supplies the *where* while the id supplies the
*what*.

| Id  | English text     | Id  | English text     |
|-----|------------------|-----|------------------|
| 192 | At The Tent      | 200 | On The Hill      |
| 193 | At The Intel     | 201 | In The Trench    |
| 194 | On The Bridge    | 202 | In Their Base    |
| 195 | Under The Bridge | 203 | In Our Base      |
| 196 | In The Tunnel    | 204 | At The Wall      |
| 197 | In The Water     | 205 | In The Open      |
| 198 | On The Roof      |     |                  |
| 199 | Behind The Wall  |     |                  |

#### 208-255 — Reserved

Free for later versions of this extension. A version 1 client or server must not
send them and ignores them on receipt.

#### Notes for implementers

The canonical strings are the translation keys. A client translates by **id**,
never by matching the English text it received, and never localises anything a
peer might parse back.

### Per-player state

Everything this extension creates belongs to a player id, and all of it is freed
the moment that player leaves. When a client receives
[Player Left](protocol075.html#player-left) for an id it drops, for that id: the
player's active ping, the ESP mark on them if any, and any predefined message
still displayed that named them as sender or as target. The server drops the same
things on its side and stops referring to the id.

The reason is that ids are recycled. A mark or a ping outliving its owner does
not fade away quietly — it lands on whoever takes the id next, and that player is
suddenly revealed to the enemy team, or pinned to a marker they never made.

A map change ([Map Start](protocol075.html#map-start-075)) clears everything for
every id, on both ends. Nothing this extension holds is meant to survive a world.


# Packetless Packets
* [Player Limit](#player-limit)
* [Message Types](#message-types)
* [Kick Reason](#kick-reason)

## Player Limit

Tells client server supports up to 256 players.

| ---------: |-----|
| Packet ID: | 192 |
| Version:   | 1   |

## Message Types

This packet is an extension to the [Chat Message](protocol075.html#chat-message), it adds new chat types.
So clients can handle it how they want, in most clients it will display the message in different area/size/color/sound in
player's screen.

| ---------: |-----|
| Packet ID: | 193 |
| Version:   | 1   |

#### New Types:

| Value | Type         | Notes                                 |
|-------|--------------|---------------------------------------|
| 3     | CHAT_BIG     | Displayed on the center of the screen |
| 4     | CHAT_INFO    | Displays a notice                     |
| 5     | CHAT_WARNING | Displays a warning                    |
| 6     | CHAT_ERROR   | Displays a error                      |

## Kick Reason

Send a [Chat Message](protocol075.html#chat-message) with type 2 (CHAT_SYSTEM) and player id 255, before
kicking a player out of the server.

| ---------: |-----|
| Packet ID: | 194 |
| Version:   | 1   |

# Other Extensions
* [UTF-8 Chat](#utf-8-chat)

## UTF-8 Chat

Has no registered extension id. A [Chat Message](protocol075.html#chat-message)
whose data starts with a `0xff` byte is encoded in UTF-8 instead of Code Page
437. Messages without the prefix are interpreted as Code Page 437 as in the
base protocol.