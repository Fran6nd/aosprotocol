# Command List

The server tells the client which chat commands it accepts and what each one
takes, so that a client can complete them as they are typed, show their help,
and ask for a missing argument instead of letting the server reject the line.

Commands stay what they are in the base protocol: a
[Chat Message](../protocol075.md#chat-message) that starts with `/`. Nothing here
adds a way to run one. The list only describes them, and the server goes on
parsing, checking rights and refusing exactly as it did before.

| ------------: | ------------- |
| Extension ID: | `0x38`        |
| Packet ID:    | `0x78`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

### Sub Packets:

| Sub ID | Name     | Direction        | Size |
|--------|----------|------------------|------|
| 0      | Commands | Server -> Client | 3+   |

Server to client only; a server that receives one drops it.

## Sub ID 0: Commands

The whole list, every time: a Commands packet replaces the list the client had,
entire.

| Field Name    | Field Type | Example | Notes                                         |
|---------------|------------|---------|-----------------------------------------------|
| Packet ID     | UByte      | `0x78`  | Always `0x78`.                                |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet.               |
| Command Count | UByte      | `12`    | Commands that follow, `0`-`255`.              |
| Commands      | Command[]  |         | The remaining bytes, see [Command](#command). |

The count is stated rather than implied by the packet length, as in the
[Pie Menu](pie-menu.md#menu), so that one wrong length byte cannot eat the
command behind it and still parse.

Help and parameter names carry text the way a
[Chat Message](../protocol075.md#chat-message) does: Code Page 437, or UTF-8
behind the `0xff` prefix of [UTF-8 Chat](utf-8-chat.md). Command names and
choices do not, see [Names](#names).

### Command

| Field Name  | Field Type | Example          | Notes                                  |
|-------------|------------|------------------|----------------------------------------|
| Name Length | UByte      | `4`              | Bytes of Name, `1`-`32`.               |
| Name        | Text       | `"kick"`         | Without the `/`, see [Names](#names).  |
| Help Length | UByte      | `16`             | Bytes of Help, `0`-`255`.              |
| Help        | Text       | `"Removes a player"` | One line. May be empty.            |
| Param Count | UByte      | `2`              | Params that follow, `0`-`8`.           |
| Params      | Param[]    |                  | See [Param](#param).                   |

Commands are listed in the order the server prefers to show them. A server lists
only the commands this player may run now; when that changes, say after a login,
it sends the list again.

Aliases are separate commands. A server that wants `/k` to complete like `/kick`
lists both.

### Param

| Field Name   | Field Type | Example    | Notes                                      |
|--------------|------------|------------|--------------------------------------------|
| Type         | UByte      | `3`        | See [Types](#types).                       |
| Flags        | UByte      | `0b00`     | See below.                                 |
| Name Length  | UByte      | `6`        | Bytes of Name, `1`-`24`.                   |
| Name         | Text       | `"player"` | What a prompt or a hint calls it.          |
| Choice Count | UByte      | `0`        | `0` unless Type is `CHOICE`, then `1`-`16`. |
| Choices      | Choice[]   |            | See [Choices](#choices).                   |

| Bit | Name       | Meaning                                                       |
|-----|------------|---------------------------------------------------------------|
| 0   | `OPTIONAL` | The command runs without it.                                  |
| 1   | `REST`     | Takes the rest of the line, spaces included.                  |
| 2-7 | reserved   | Must be `0`. Clients **must** ignore unknown bits.            |

Arguments are positional. Once a param is `OPTIONAL`, every param after it is
too, and only the last param may be `REST`.

### Types

| Value     | Name      | The client sends                                       |
|-----------|-----------|--------------------------------------------------------|
| `0`       | `WORD`    | Any text without a space.                              |
| `1`       | `INTEGER` | A decimal integer, `-` allowed.                        |
| `2`       | `NUMBER`  | A decimal number, `-` and one `.` allowed.             |
| `3`       | `PLAYER`  | `#` and the player id in decimal, as in `#7`.          |
| `4`       | `CHOICE`  | One of the [Choices](#choices), as listed.             |
| `5`-`255` | reserved  | Refuses the list, see [Validation](#validation).       |

The type is what the client offers and how it checks what is typed, not a
promise: a client may still send a line that does not fit, and the server
answers it as it would any other.

`PLAYER` is always sent as `#id`, never as a name, because names need not be
unique and may hold spaces. A client picks the player however it likes, from the
names it knows, and writes the id.

Only a `WORD` may be `REST`. It then takes the rest of the line, spaces
included, so a reason or a message is one param.

### Choices

| Field Name    | Field Type | Example  | Notes                          |
|---------------|------------|----------|--------------------------------|
| Choice Length | UByte      | `4`      | Bytes of Choice, `1`-`24`.     |
| Choice        | Text       | `"rain"` | Sent as is, see [Names](#names). |

### Names

A command Name and a Choice are typed by the player and parsed by the server, so
they are restricted to printable ASCII, `0x21`-`0x7e`: no space, no `/` at the
start of a Name, and no `0xff` prefix. Names are compared as the server sends
them; a client may match them case-insensitively while completing, but sends
the server's spelling.

### Language

A server **should** write Help and param Names in the player's language. It
learns it from the User locale of a
[Version Response (Extended)](../protocol075.md#version-response-extended-ep),
asked for with a [Version Get](../protocol075.md#version-get-ep) listing
property `1`: the client's interface language, such as `en_us` or `fr`. A
server matches the language and region it has, falls back to the language
alone, and then to its own default. With no locale, it uses its default.

Command Names and Choices are never translated. The player types them back, and
the server parses them the same for everyone.

Each connection is sent its own list, so players in different languages read
the same commands in their own. A server that sends the list before the locale
arrives sends it again once it does.

## The line a client sends

`/`, the Name, then each argument, separated by single spaces:

```
/kick #7 spawn camping
```

for a command `kick` with params `PLAYER player` and `WORD reason` flagged
`REST`. It goes out as a [Chat Message](../protocol075.md#chat-message), however
the client normally sends commands.

The list is advisory. A client **must not** refuse to send a command that is not
on it, or an argument that does not fit its type, since the server may accept
commands it does not list. A client that has received no list, or an empty one,
behaves as it would without the extension.

## With the Pie Menu

When [Pie Menu](pie-menu.md) is negotiated as well, a
[`COMMAND`](pie-menu.md#actions) slice may leave arguments for the player to
give. The Pie Menu itself interprets none of a slice's Text; this extension
lets the client read it against the list, and only then.

Once the slice's [Substitutions](pie-menu.md#substitutions) are expanded, the
client looks up the first word, less its `/`, in the list:

* **Not listed, or every required param given.** The slice is sent as it is,
  exactly as without this extension.
* **Listed, with required params missing.** The client asks for the missing
  ones, in order, by their Types, and sends the full line once they are given.
  Closing the prompt sends nothing, like releasing a pie on its centre.

If the menu was opened on a player, the first missing `PLAYER` param is that
player, and the client fills it in without asking. So a slice `/kick` in an
[`ENEMY`](pie-menu.md#contexts) pie asks only for the reason, and the same
slice in a [`WORLD`](pie-menu.md#contexts) pie asks for the player too. A
server that wants the target placed elsewhere still writes `%p` itself.

Optional params are never asked for. A prompt is the client's to draw; it opens
once the slice is chosen, never while the ring is held.

## Validation

A client parses the whole packet before applying any of it, and **a list is
applied whole or not at all**. Where one is refused the previous list stands and
the client logs it rather than disconnecting. A list is refused when:

* A record runs past the end of the packet, or bytes remain after the last
  command.
* Command Count does not match the commands that follow.
* A Name, Help, param Name or Choice exceeds its cap, see [Limits](#limits), or
  a Name or Choice is empty.
* A Name or Choice holds a byte outside `0x21`-`0x7e`, or a Name starts with `/`.
* Two commands share a Name.
* A Type is `5` or above, or a reserved bit is set in Flags.
* A required param follows an `OPTIONAL` one, or a param other than the last,
  or one that is not a `WORD`, is `REST`.
* A `CHOICE` param has no choices, or a param of another type has some.

## Limits

| Thing                 | Cap   |
|-----------------------|-------|
| Commands per list     | `255` |
| Params per command    | `8`   |
| Choices per param     | `16`  |
| Command Name          | `32`  |
| Help                  | `255` |
| Param Name, Choice    | `24`  |

The string caps are in bytes, not characters, so a parser can enforce them before
decoding anything.

## Connection state

The list belongs to the connection, not to the world.
[Map Start](../protocol075.md#map-start-075) does not clear it, the rule the
[Pie Menu](pie-menu.md#connection-state) follows. A server whose commands
change with the map sends a new list.

See [Extensions](extension.md) for how the extension is negotiated.
