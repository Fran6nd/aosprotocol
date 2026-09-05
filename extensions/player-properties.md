# Player Properties

Sends additional player attributes from the server to the client.

| ------------: | ------------- |
| Extension ID: | 0             |
| Packet ID:    | 64            |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

| Sub ID | Name              | Direction        | Size |
|--------|-------------------|------------------|------|
| 0      | Player Properties | Server -> Client | 12   |

## Sub ID 0: Player Properties

The server sends this packet to inform the client of authoritative stats for
a single player. There is no client-to-server form. The 12-byte total
includes the Packet ID and Sub Packet ID as described in the
[general extension packet structure](extension.html#extension-packets).

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

See [Extensions](extension.html) for how the extension is negotiated.
