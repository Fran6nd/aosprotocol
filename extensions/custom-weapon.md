# Custom Weapon

The server sets what each player carries: how many tools they have, in which
order, and how each firearm behaves. A loadout can be a spade and a block, three
grenades and an empty hand, or a rifle, a shotgun and a spade.

| ------------: | ------------- |
| Extension ID: | `0x39`        |
| Packet ID:    | `0x79`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

### Sub Packets:

| Sub ID | Name    | Direction        | Size |
|--------|---------|------------------|------|
| 0      | Loadout | Server -> Client | 3+   |

Server to client only; a server that receives one drops it.

## Sub ID 0: Loadout

One player's tools. The server sends it to every client, for every player, so
that each client can draw what others hold. A new Loadout replaces the player's
old one at once, whenever the server sends it.

| Field Name    | Field Type | Example | Notes                          |
|---------------|------------|---------|--------------------------------|
| Packet ID     | UByte      | `0x79`  | Always `0x79`.                 |
| Sub Packet ID | UByte      | `0`     | Always `0` for this sub-packet. |
| Player ID     | UByte      | `7`     | Whose loadout this is.         |
| Slot Count    | UByte      | `3`     | Slots that follow.             |
| Slots         | Slot[]     |         | See [Slot](#slot).             |

### Slot

| Field Name | Field Type | Example | Notes                                  |
|------------|------------|---------|----------------------------------------|
| Category   | UByte      | `2`     | See [Categories](#categories).         |
| Item       | UByte      | `0`     | Depends on the Category.               |
| Firearm    | Firearm    |         | Only when Category is `FIREARM`, see [Firearm](#firearm). |

Slots are in the order the player cycles through them. While a loadout is set,
the Tool of [Set Tool](../protocol075.md#set-tool) and the Held item of
[Existing Player](../protocol075.md#existing-player) are a slot index, not a
base tool.

### Categories

| Value | Name             | Items                              |
|-------|------------------|------------------------------------|
| `0`   | `CLOSE_QUARTERS` | `0` spade, `1` empty hand          |
| `1`   | `BUILDING`       | `0` block                          |
| `2`   | `FIREARM`        | `0`; the rest is in the Firearm    |
| `3`   | `THROWABLE`      | `0` grenade                        |

A client treats an unknown Category or Item as an empty hand.

### Firearm

| Field Name     | Field Type | Example | Notes                                                    |
|----------------|------------|---------|----------------------------------------------------------|
| Model          | UByte      | `0`     | What it looks like, see [Models](#models).               |
| Action         | UByte      | `1`     | `0` manual (bolt or pump), `1` semi-automatic, `2` automatic. |
| Fire Type      | UByte      | `0`     | `0` raycast, `1` launcher.                               |
| Reload         | UByte      | `0`     | `0` whole magazine, `1` one round at a time.             |
| Magazine       | UByte      | `10`    | Rounds in a full magazine.                               |
| Reserve        | UByte      | `50`    | Rounds carried beyond it.                                |
| Pellets        | UByte      | `1`     | Shots per trigger pull.                                  |
| Fire Interval  | LE Float   | `0.5`   | Seconds between shots.                                   |
| Reload Time    | LE Float   | `2.5`   | Seconds for a full reload, or per round.                 |
| Spread         | LE Float   | `0.006` | As Voxlap's guns.                                        |
| Recoil Up      | LE Float   | `0.05`  | As Voxlap's guns.                                        |
| Recoil Side    | LE Float   | `0.0001`| As Voxlap's guns.                                        |
| Muzzle Speed   | LE Float   | `0.0`   | Launcher only: the projectile's starting speed.          |

The three vanilla guns, in these terms:

| Gun     | Model | Action | Reload | Magazine | Reserve | Pellets | Interval | Reload Time | Spread | Recoil Up | Recoil Side |
|---------|-------|--------|--------|----------|---------|---------|----------|-------------|--------|-----------|-------------|
| Rifle   | `0`   | `1`    | `0`    | `10`     | `50`    | `1`     | `0.5`    | `2.5`       | `0.006`| `0.05`    | `0.0001`    |
| SMG     | `1`   | `2`    | `0`    | `30`     | `120`   | `1`     | `0.1`    | `2.5`       | `0.012`| `0.0125`  | `0.00005`   |
| Shotgun | `2`   | `0`    | `1`    | `6`      | `48`    | `8`     | `1.0`    | `0.5`       | `0.024`| `0.1`     | `0.0002`    |

A **raycast** firearm hits as the vanilla guns do, and the client reports hits
with the [Hit Packet](../protocol075.md#hit-packet). A **launcher** fires a
projectile that flies like a grenade from the muzzle at Muzzle Speed; the client
sends a [Grenade Packet](../protocol075.md#grenade-packet) for it. Damage stays
the server's business.

### Models

| Value | Model          |
|-------|----------------|
| `0`   | Rifle          |
| `1`   | SMG            |
| `2`   | Shotgun        |
| `3`   | Bolt-action    |
| `4`   | Pistol         |
| `5`   | Machine gun    |

A Model is only what the firearm looks and sounds like. A client without one
draws the vanilla model nearest to it: a pistol or a bolt-action as the rifle, a
machine gun as the SMG.

## Clients without the extension

They get the base protocol only: a server keeps them on the vanilla rifle, SMG
or shotgun, and their Set Tool values stay base tools.

## Connection state

A player's loadout lasts until another replaces it or the player leaves.
[Map Start](../protocol075.md#map-start-075) does not clear it.

See [Extensions](extension.md) for how the extension is negotiated.
