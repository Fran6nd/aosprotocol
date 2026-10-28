# Team Flags

Flags that leave a whole team out of what a client reports about players: the
scoreboard, join and leave messages, and the kill feed. A server uses them for a
team of bots, such as players fighting zombies.

| ------------: | ------------- |
| Extension ID: | `0x3C`        |
| Packet ID:    | `0x7C`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

### Sub Packets:

| Sub ID | Name       | Direction        | Size          |
|--------|------------|------------------|---------------|
| 0      | Team Flags | Server -> Client | `2+2*entries` |

A client sends no sub-packet. The server drops any it receives from a client.

## Flags

| Bit | Name              | Meaning                                                      |
|-----|-------------------|--------------------------------------------------------------|
| 0   | `HIDE_SCOREBOARD` | The team and its players are left out of the scoreboard.     |
| 1   | `HIDE_PRESENCE`   | No join, team change or leave message for its players.       |
| 2   | `HIDE_KILLFEED`   | No kill feed line for a kill its players made or suffered.   |
| 3-7 | reserved          | Must be `0`. Clients ignore unknown bits.                    |

A team change is hidden if either the old or the new team has `HIDE_PRESENCE`,
and a kill is hidden if either player's team has `HIDE_KILLFEED`.

The flags change what is reported, not what is drawn: the team's players are
still rendered, heard and hit as usual, and keep their team colour and name. A
client ignores them for its own player, and still tells its player about their
own kills and deaths.

## Sub ID 0: Team Flags

| Field Name    | Field Type       | Example | Notes                           |
|---------------|------------------|---------|---------------------------------|
| Packet ID     | UByte            | `0x7C`  | Always `0x7C`.                  |
| Sub Packet ID | UByte            | `0`     | Always `0` for this sub-packet. |
| Entries       | TeamFlagsEntry[] |         | To the end of the packet.       |

**TeamFlagsEntry** (2 bytes)

| Field Name | Field Type | Example  | Notes                 |
|------------|------------|----------|-----------------------|
| Team ID    | UByte      | `1`      | `255` for spectators. |
| Flags      | UByte      | `0b0011` | See [Flags](#flags).  |

The server sends it before [State Data](../protocol075.md#state-data), and again
whenever the flags change. Each Team Flags replaces the previous one, and a team
it does not list has no flags. It lasts for the connection:
[Map Start](../protocol075.md#map-start-075) does not reset it.

## PvE

Players on team `0`, bots on team `1`. The server sends one entry:

| Team ID | Flags                              |
|---------|------------------------------------|
| `1`     | `HIDE_SCOREBOARD`, `HIDE_PRESENCE` |

The scoreboard lists only the players, bots come and go silently, and the kill
feed still shows them. How the bots look, such as zombies, is not part of this
extension; the Player Outfit extension's Team Outfits can set it.

A client without the extension shows the bots as a normal team.

See [Extensions](extension.md) for how the extension is negotiated.
