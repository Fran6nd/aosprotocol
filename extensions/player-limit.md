# Player Limit

Tells the client that the server supports up to 256 players.

| ------------: | ------------ |
| Extension ID: | 192          |
| Version:      | 1            |
| Type:         | `PACKETLESS` |

Vanilla 0.75 reserves player id 255 as a special value and clients traditionally
assume a much smaller player limit. A server announcing this extension tells the
client that it may receive player ids covering the whole `UByte` range, and that
it should size its player table accordingly.

This extension does not change the wire format of any packet, it only signals
that the full id range is in use.

See [Extensions](../protocol075.html#extensions) for how the extension is
negotiated.
