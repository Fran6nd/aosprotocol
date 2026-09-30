# Damage Markers

Tells the client how much damage a hit did and who took it, so the client can
float the number over that player.

| ------------: | ------------- |
| Extension ID: | `0x20`        |
| Packet ID:    | `0x60`        |
| Version:      | 1             |
| Type:         | `HAS_PACKETS` |

Sent by the server to the player who dealt the damage, one packet per hit. There
is no client-to-server form.

| Field Name | Field Type | Example | Notes                               |
|------------|------------|---------|-------------------------------------|
| Packet ID  | UByte      | `0x60`  | Always `0x60`.                      |
| Player ID  | UByte      | `7`     | The player who took the damage.     |
| Hit Amount | see below  | `45`    | Damage the server applied.          |

The packet length selects the type of Hit Amount:

| Packet length | Hit Amount type    |
|--------------:|--------------------|
| 3 bytes       | UByte              |
| 4 bytes       | LE Short (signed)  |
| 6 bytes       | LE Int (signed)    |

The client drops a packet of any other length. A negative Hit Amount is a heal.

How the number is drawn is the client's call; TigerSpades floats it above the
player for about three seconds.

Nothing is retained between packets, so there is no state to clear on
[Player Left](../protocol075.md#player-left) or
[Map Start](../protocol075.md#map-start-075).

See [Extensions](extension.md) for how the extension is negotiated.
