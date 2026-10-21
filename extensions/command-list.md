# Command List

The server tells the client which chat commands it accepts and what each one
takes, so that a client can complete them as they are typed, show their help,
and ask for a missing argument.

Commands stay `/` [Chat Messages](../protocol075.md#chat-message). The list only
describes them, and the server parses and checks them as before.

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

Replaces the client's whole list.

| Field Name    | Field Type | Example | Notes                                         |
|---------------|------------|---------|-----------------------------------------------|
| Packet ID     | UByte      | `0x78`  | Always `0x78`.                                |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet.               |
| Command Count | UByte      | `12`    | Commands that follow, `0`-`255`.              |
| Commands      | Command[]  |         | The remaining bytes, see [Command](#command). |

Every Text field follows a UByte giving its length in bytes, and every range
below is in bytes, so a parser can check it before decoding. Help and param
Names are encoded as a [Chat Message](../protocol075.md#chat-message) is: Code
Page 437, or UTF-8 behind the `0xff` prefix of [UTF-8 Chat](utf-8-chat.md).
Command Names and Choices are ASCII, see [Names](#names).

A server **should** keep the packet under 16 KiB; a client may refuse one over
64 KiB.

### Command

| Field Name  | Field Type | Example              | Notes                                 |
|-------------|------------|----------------------|---------------------------------------|
| Name Length | UByte      | `8`                  | `1`-`32`.                             |
| Name        | Text       | `"teleport"`         | Without the `/`, see [Names](#names). |
| Alias Count | UByte      | `1`                  | `0`-`8`.                              |
| Aliases     | Alias[]    |                      | Other names for the same command.     |
| Help Length | UByte      | `16`                 | `0`-`255`.                            |
| Help        | Text       | `"Moves a player"`   | No line breaks; a client may cut it.  |
| Param Count | UByte      | `2`                  | `0`-`8`.                              |
| Params      | Param[]    |                      | See [Param](#param).                  |

Each Alias is a Name Length and a Name, as above. A client completes an alias
like the command, and lists the command once.

Commands are listed in the order the server prefers to show them. A server lists
only the commands this player may run now, and sends the list again when that
changes, say after a login.

### Param

| Field Name   | Field Type | Example    | Notes                                         |
|--------------|------------|------------|-----------------------------------------------|
| Type         | UByte      | `3`        | See [Types](#types).                          |
| Flags        | UByte      | `0b000`    | See below.                                    |
| Name Length  | UByte      | `6`        | `1`-`24`.                                     |
| Name         | Text       | `"player"` | What a prompt or a hint calls it.             |
| Choice Count | UByte      | `0`        | `0` unless Type is `CHOICE`, then `1`-`32`.   |
| Choices      | Choice[]   |            | Each a Choice Length, `1`-`24`, and a Choice. |

| Bit | Name       | Meaning                                                         |
|-----|------------|-----------------------------------------------------------------|
| 0   | `OPTIONAL` | The command runs without it. Every param after it is optional too. |
| 1   | `REST`     | Takes the rest of the line, spaces included. Last param, `WORD` only. |
| 2   | `SECRET`   | A password or the like: masked while typed, kept out of history. |
| 3-7 | reserved   | Must be `0`. Clients **must** ignore unknown bits.              |

Arguments are positional.

### Types

| Value     | Name      | The client sends                                       |
|-----------|-----------|--------------------------------------------------------|
| `0`       | `WORD`    | Any text without a space.                              |
| `1`       | `INTEGER` | A decimal integer, `-` allowed.                        |
| `2`       | `NUMBER`  | A decimal number, `-` and one `.` allowed.             |
| `3`       | `PLAYER`  | `#` and the player id in decimal, as in `#7`.          |
| `4`       | `CHOICE`  | One of the Choices, as listed.                         |
| `5`-`255` | reserved  | A client treats an unknown Type as `WORD`.             |

`PLAYER` is sent as `#id`, since names repeat and hold spaces.

### Names

A command Name or Alias is lowercase: `a`-`z`, `0`-`9`, `_`, `-`, `.` and `+`.
A Choice is printable ASCII, `0x21`-`0x7e`, except `"`, `'` and `\`, which
servers treat as quoting.

### Language

Help and param Names **should** be in the player's
[User locale](../protocol075.md#version-response-extended-ep); command Names,
Aliases and Choices never are. A list sent before the locale arrives is sent
again after it.

## The line a client sends

`/`, the Name, then each argument, separated by single spaces:

```
/kick #7 spawn camping
```

for a command `kick` with params `PLAYER player` and `WORD reason` flagged
`REST`.

A client adds no quotes or escapes. Each server splits the line itself, so a
`REST` argument reaches the command as that server splits it. Servers cap a chat
line, piqueserver at 108 characters, so a client caps a `REST` prompt to what
fits.

The list is advisory. A client **must not** refuse to send a command that is not
on it, or an argument that does not fit its type, since the server may accept
commands it does not list. A client that has received no list, or an empty one,
behaves as it would without the extension.

## Validation

A client parses the whole packet before applying any of it, and **a list is
applied whole or not at all**. Where one is refused the previous list stands and
the client logs it rather than disconnecting. A list is refused when:

* A record runs past the end of the packet, or bytes remain after the last
  command.
* A count or a length is outside the range in its Notes.
* A Name, Alias or Choice holds a character outside those in [Names](#names).
* A Name or Alias appears twice in the list, as either.
* A required param follows an `OPTIONAL` one, or a param other than the last,
  or one that is not a `WORD`, is `REST`.
* A `CHOICE` param has no choices, or a param of another type has some.

## Connection state

The list belongs to the connection, not to the world.
[Map Start](../protocol075.md#map-start-075) does not clear it. A server whose
commands change with the map sends a new list.

See [Extensions](extension.md) for how the extension is negotiated.
