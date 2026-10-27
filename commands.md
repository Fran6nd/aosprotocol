# Commands

A command is a [Chat Message](protocol075.md#chat-message) that starts with `/`.
The server reads it instead of relaying it, and answers in chat. That is the
whole mechanism, and every client since Voxlap already speaks it.

So commands are the cheapest way to add gameplay: a server script registers
`/medkit`, `/squad` or `/airstrike`, and every client can use it the same day,
with no new packet, no negotiation and no client release. This page lists the
commands servers already share, and the conventions that keep new ones usable
from any client, whether typed or bound to a key.

The table is drawn from [piqueserver](https://github.com/piqueserver/piqueserver)
(core commands and bundled scripts) and [LSD](https://66.135.15.57/lsd/) (its
`scripts/` folder), as of October 2026. A command both ship under the same name
is listed; one only a single server has is listed when it is widely used on
that server or is a pattern worth copying.

## Command or extension?

Reach for a command first. Write an extension only when the client must do
something it cannot do by sending chat and reading chat back:

| A command is enough                          | An extension is needed                          |
|----------------------------------------------|-------------------------------------------------|
| The player triggers something on the server. | The client must draw something new.             |
| The answer is a line of text.                | The answer is data the client must act on.      |
| It runs now and then, by hand.               | It runs every frame, or on its own.             |
| `/medkit`, `/votekick`, `/squad 2`           | Pings, damage markers, a flashlight's beam      |

The two meet: an extension can carry the *what* (a marker, a prompt) and leave
the *do it* to a command, sent as a plain chat line.

## The line

```
/name arg1 arg2 rest of the line
```

* **Name.** Lowercase ASCII, one word, no `/` inside. Servers match it
  case-insensitively. Give a long name and, if it is typed often, a short alias:
  `/togglebuild` and `/tb`.
* **Arguments.** Separated by single spaces, positional. Optional ones come
  last. Only the last argument may hold spaces, and it takes the rest of the
  line — a reason, a message.
* **Answer.** The server always replies, even if only to say it refused, and
  replies to the sender alone unless the command is about everyone. A reply is
  ordinary chat; [Message Types](extensions/message-types.md) may colour it.
* **Rights.** The server checks them on every call. A client never hides or
  blocks a command on its own guess about rights.

### Argument conventions

These are the forms both piqueserver and LSD already accept. A new command
should take the same ones, and a client sending a command should write them.

| Kind     | Write it as                    | Both servers accept                       | Notes |
|----------|--------------------------------|-------------------------------------------|-------|
| Player   | `#7`                           | `#id`, or part of a name                  | Always send `#id` from a client: names repeat, change and hold spaces. |
| Team     | the team's name, or `spectator`| team name, `spectator`                    | LSD also takes `1`, `2`, `spec`; piqueserver does not. |
| Duration | a number and a unit: `30min`, `2h`, `1d`, `1w` | `s`, `min`, `h`, `d`, `w`  | **Always give the unit.** A bare `30` is minutes on piqueserver and seconds on LSD. |
| Switch   | `on` / `off`                   | piqueserver: `on`, `off`, `yes`, `no`     | Leave it out to toggle. |
| Text     | as typed                       | the rest of the line                      | Last argument only. |

## Common commands

Arguments in `<angle brackets>` are required, `[square]` optional, and `...`
takes the rest of the line. **P** is piqueserver, **L** is LSD; a mark in
brackets means the command comes from a script the server ships but does not
load by default. "Admin" means a rights check: an admin on piqueserver, a named
capability on LSD.

### Information

| Command          | Aliases            | Arguments   | Does                                         | P | L | Admin |
|------------------|--------------------|-------------|----------------------------------------------|---|---|-------|
| `/help`          | `/man` (L)         | `[command]` | Usage of a command. LSD requires the command.| ✓ | ✓ |       |
| `/commands`      | `/cmds` (L)        | `[page]`    | Lists the commands. The page is LSD's.       | ✓ | ✓ |       |
| `/ping`          |                    | `[player]`  | Round-trip time, yours by default.           | ✓ | ✓ |       |
| `/ratio`         |                    | `[player]`  | Kills per death, yours by default.           |(✓)| ✓ |       |
| `/client`        | `/cli`             | `[player]`  | What client a player claims to run.          | ✓ | ✓ |       |
| `/mapname`       | `/mapinfo` (L)     |             | The current map.                             | ✓ | ✓ |       |
| `/showrotation`  | `/mapqueue` (L)    |             | The maps that come next.                     | ✓ | ✓ |       |
| `/analyze`       | `/an`              | `[player]`  | Echoes a player's hits, for spotting cheats. |(✓)| ✓ |       |
| `/time`          |                    |             | Time left in the round.                      | ✓ |   |       |
| `/streak`        |                    |             | Your current and best kill streak.           | ✓ |   |       |
| `/intel`         |                    |             | Who holds the intel.                         | ✓ |   |       |
| `/who`           | `/lscon`           |             | Lists the players.                           |   | ✓ |       |

### Talking

| Command   | Aliases | Arguments           | Does                                              | P | L | Admin |
|-----------|---------|---------------------|---------------------------------------------------|---|---|-------|
| `/pm`     |         | `<player> <text...>`| A message one player sees.                        | ✓ | ✓ |       |
| `/admin`  |         | `<text...>`         | A message to the staff online.                    | ✓ | ✓ |       |
| `/login`  |         | `<password>`        | Log in. LSD takes `<name> <password> [otp]`.      | ✓ | ✓ |       |
| `/say`    |         | `<text...>`         | A message shown as from the server.               | ✓ | ✓ | ✓     |
| `/mute`   | `/tc` (L) | `<player>`        | Silences a player for everyone. LSD toggles; piqueserver has `/unmute`. | ✓ | ✓ | ✓ |
| `/deaf`   |         | `[player]`          | Toggles chat reaching you; an admin may name another player. | ✓ |   |       |
| `/ignore` |         | `<player>`          | Stops one player's chat reaching you.             |   | ✓ |       |

### Playing

| Command      | Aliases | Arguments  | Does                                                 | P | L | Admin |
|--------------|---------|------------|------------------------------------------------------|---|---|-------|
| `/kill`      |         | `[player]` | Kills you; an admin may name another player.         | ✓ | ✓ |       |
| `/medkit`    | `/m`    |            | Uses a medkit.                                       |(✓)|(✓)|       |
| `/votekick`  |         | `<player> <reason...>` | Starts a vote to kick a player.          |(✓)|   |       |
| `/y`         |         |            | Votes yes in the running votekick.                   |(✓)|   |       |
| `/cancel`    |         |            | Cancels your votekick.                               |(✓)|   |       |
| `/votemap`   |         |            | Starts a vote on the next map.                       |(✓)|   |       |
| `/vote`      |         | `<choice>` | Votes in the running map vote.                       |(✓)|   |       |
| `/squad`     |         | `[name]`   | Joins a squad; you respawn near it.                  |(✓)|   |       |
| `/follow`    |         | `[player]` | Respawns you near a squadmate.                       |(✓)|   |       |
| `/airstrike` | `/a`    |            | Calls an airstrike earned by a streak.               |(✓)|   |       |
| `/drop`      |         |            | Drops the intel you carry.                           |   |(✓)|       |
| `/ping`      |         | `<x> <y> <z> [reason...]` | Marks a place, as a [Teamplay](extensions/teamplay.md#the-ping-command) ping. Comes with the extension. |   |   |       |

### Moderation

| Command         | Aliases      | Arguments                        | Does                                    | P | L | Admin |
|-----------------|--------------|----------------------------------|-----------------------------------------|---|---|-------|
| `/kick`         |              | `<player> [reason...]`           | Disconnects a player. LSD takes no reason. | ✓ | ✓ | ✓ |
| `/ban`          |              | `<player> <duration> <reason...>`| Bans a player. Duration optional on piqueserver. | ✓ | ✓ | ✓ |
| `/hban`         |              | `<player> <reason...>`           | Bans for an hour.                       | ✓ | ✓ | ✓     |
| `/dban`         |              | `<player> <reason...>`           | Bans for a day.                         | ✓ | ✓ | ✓     |
| `/wban`         |              | `<player> <reason...>`           | Bans for a week.                        | ✓ | ✓ | ✓     |
| `/pban`         |              | `<player> <reason...>`           | Bans for good.                          | ✓ | ✓ | ✓     |
| `/banip`        |              | `<address> <duration> <reason...>` | Bans an address or range.             | ✓ | ✓ | ✓     |
| `/unban`        |              | `<address or ban id>`            | Lifts a ban. LSD takes the ban id.      | ✓ | ✓ | ✓     |
| `/switch`       |              | `[player] [team]`                | Moves a player, yourself by default, to a team, the other one by default. | ✓ | ✓ | ✓ |
| `/togglebuild`  | `/tb`        | `[player]`                       | Stops a player, or everyone, building. LSD requires the player. | ✓ | ✓ | ✓     |
| `/togglekill`   | `/tk`        | `[player]`                       | Stops a player, or everyone, hurting others. LSD requires the player. | ✓ | ✓ | ✓ |
| `/teleport`     | `/tp`        | `[player] <to player>`           | Moves a player, yourself by default, to another. LSD has only `/tp`. | ✓ | ✓ | ✓ |
| `/god`          |              | `[player]`                       | Makes a player, yourself by default, unhurtable. LSD takes no player. | ✓ | ✓ | ✓ |
| `/invisible`    | `/invis`     | `[player]`                       | Hides a player from everyone else.      | ✓ |   | ✓     |
| `/fly`          |              | `[player]`                       | Lets a player fly.                      | ✓ |   | ✓     |

### Map and server

| Command        | Aliases      | Arguments    | Does                                         | P | L | Admin |
|----------------|--------------|--------------|----------------------------------------------|---|---|-------|
| `/map`         |              | `<map...>`   | Sets the next map or maps.                   | ✓ | ✓ | ✓     |
| `/advancemap`  | `/advance`   |              | Ends the round and loads the next map.       | ✓ | ✓ | ✓     |
| `/loadmap`     |              | `<map>`      | Loads a map now.                             | ✓ | ✓ | ✓     |
| `/fog`         |              | `<r> <g> <b>`| Sets the fog colour. piqueserver also takes a hex colour. | ✓ | ✓ | ✓ |
| `/savemap`     |              | `[name]`     | Saves the map as it stands. LSD requires the name. |(✓)|(✓)| ✓ |
| `/disco`       |              |              | Toggles a flashing fog party.                |(✓)| ✓ | ✓     |

## Scripting gameplay with commands

What makes a script's command feel native rather than bolted on:

* **One verb, one command.** `/medkit` heals, `/squad 2` joins squad 2. A
  command that does several things by mode is hard to bind to a key.
* **Toggles take an optional `on`/`off`.** With no argument they flip. A bound
  key then works either way, and one bound to `on` means it.
* **Act on the sender by default.** `[player]` defaults to whoever typed it, so
  `/kill` and `/god` need no argument from a key.
* **Answer every call.** Say what happened, or why not. A silent command looks
  broken from a key, where the player never saw a line typed.
* **Use the conventions above** for players, teams and durations, so one client
  prompt works for every server.
* **Document it.** List it in `/help` with its arguments, in the forms above,
  so players can find it and clients can offer it.

A script that needs the client to show more than chat — a marker, a timer, a
HUD — is the point to propose an extension, see the [README](README.md#workflow).
