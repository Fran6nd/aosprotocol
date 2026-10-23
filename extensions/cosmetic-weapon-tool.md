# Cosmetic Weapon and Tool

The server sets what a player's tools look like: a bolt-action instead of the
rifle, a stick grenade instead of the vanilla one. A look never changes how a
tool behaves: the gun fires, the grenade flies and the spade digs as their base
tools do.

| ------------: | ------------- |
| Extension ID: | `0x3A`        |
| Packet ID:    | `0x7A`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

### Sub Packets:

| Sub ID | Name | Direction        | Size |
|--------|------|------------------|------|
| 0      | Look | Server -> Client | 4+   |

## Sub ID 0: Look

One player's looks. The server sends it to every client, for every player, after
their [Create Player](../protocol075.md#create-player) or
[Existing Player](../protocol075.md#existing-player) and again whenever it
changes them. A server receiving one drops it.

| Field Name    | Field Type | Example | Notes                           |
|---------------|------------|---------|---------------------------------|
| Packet ID     | UByte      | `0x7A`  | Always `0x7A`.                  |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet. |
| Player ID     | UByte      | `7`     | Whose looks these are.          |
| Entry Count   | UByte      | `2`     | Entries that follow.            |
| Entries       | Entry[]    |         | See below.                      |

| Entry Field | Field Type | Example | Notes                                        |
|-------------|------------|---------|----------------------------------------------|
| Slot        | UByte      | `3`     | Which slot, see below.                       |
| Model       | UByte      | `2`     | See [Models](#models).                       |
| Flags       | UByte      | `0b1`   | Bit `0` `TINTED`: draw the tool in Colour. Bits `1`-`7` must be `0` and are ignored. |
| Colour      | UByte[3]   |         | Blue, green, red, as in [Set Colour](../protocol075.md#set-colour). Used only with `TINTED`. |

Slots are numbered as in
[Weapon and Tool State](weapon-tool-state.md#slots): without a
[Loadout](loadout.md) the four base tools, so the gun is slot `2` and the grenade
slot `3`; with one, its slots. A slot with no entry keeps its vanilla look, and
a Look replaces the player's previous one whole.

## Models

Model `0` is always the vanilla look of the slot's tool. The other values depend
on the tool, and a client draws one it lacks, or one meant for another tool, as
`0`.

### Guns

| Value     | Model       | Looks and animates like                           |
|-----------|-------------|---------------------------------------------------|
| `1`       | Bolt-action | A long rifle whose bolt is worked between rounds. |
| `2`       | Pistol      | A handgun, held in one hand.                      |
| `3`       | Machine gun | A heavy belt-fed gun.                             |
| `4`       | SMG         | The vanilla SMG.                                  |
| `5`       | Shotgun     | The vanilla shotgun.                              |
| `6`       | DMR         | A scoped semi-automatic marksman rifle.           |
| `7`       | Launcher    | A tube that fires rockets, see below.             |
| `8`-`255` | reserved    |                                                   |

`SMG` and `Shotgun` let any gun look like those vanilla guns, whatever it is
underneath.

An animation between rounds, such as working a bolt, lasts the time between
rounds of the gun underneath.

### Grenades

| Value     | Model          | Looks like                               |
|-----------|----------------|------------------------------------------|
| `1`       | Stick grenade  | A grenade on a wooden handle.            |
| `2`       | Smoke canister | A cylinder, as a smoke grenade.          |
| `3`       | Rocket         | A finned rocket.                         |
| `4`-`255` | reserved       |                                          |

A thrown grenade keeps its thrower's look in flight, colour included, so a server
can make any grenade it wants: a red smoke canister, a white flash, a green gas.
Its explosion is the base grenade's whatever the look.

A [Grenade Packet](../protocol075.md#grenade-packet) the server sends for a
player holding a gun slot whose model is `Launcher` is drawn as a `Rocket`,
tinted as that slot. This lets a server fire rockets from a launcher; they still
fly as grenades do.

### Spades

| Value     | Model   | Looks like                     |
|-----------|---------|--------------------------------|
| `1`       | Pickaxe | A pick, swung like the spade.  |
| `2`-`255` | reserved |                               |

A model is the tool's look, animation, sounds and kill icon. Clients without
the extension draw the vanilla tools.

## Connection state

A player's looks last until another Look replaces them or
[Player Left](../protocol075.md#player-left). A respawn and
[Map Start](../protocol075.md#map-start-075) keep them.

See [Extensions](extension.md) for how the extension is negotiated.
