This page documents the networking Protocol of Ace of Spades 0.75, as well as
the extensions made by the community.

# Overview
[Ace of Spades](http://buildandshoot.com/) uses the [ENet networking
library](http://sauerbraten.org/enet/) for all server-client
communication. The initial source for the protocol information was the original
[pyspades](http://code.google.com/p/pyspades/) source code, for which the
source for 1.0 was not released. Nonetheless, the 1.0 alpha client has been
reverse engineered to document the protocol.

## Versions

 * [0.75 (and 0.76) documentation](protocol075.html)
 * [1.0 alpha documentation](protocol100a1.html)

## Extensions

The 0.75 protocol supports extensions, which allow adding new packets as well as
querying the support for client and server functionality.

The negotiation mechanism and the registry of extension ids are documented in
[Extensions](protocol075.html#extensions), and each extension has its own
specification under `extensions/`:

 * [Player Properties](extensions/player-properties.html) (0)
 * [Player Limit](extensions/player-limit.html) (192)
 * [Message Types](extensions/message-types.html) (193)
 * [Kick Reason](extensions/kick-reason.html) (194)
 * [UTF-8 Chat](extensions/utf-8-chat.html) (unregistered)

### Implementers
 * [OpenSpades](https://github.com/yvt/openspades)
 * [piqueserver](https://github.com/piqueserver/piqueserver)
 * [BetterSpades](https://github.com/xtreme8000/BetterSpades)

Links to the respective projects pages that detail the extensions evailable in
each version should be linked here.

## Other Protocols

 * [Master Server Protocol](protocolmaster.html)
 * [Ping Protocol](protocolping.html)

# Other Resources
* [KVX File Format Specification](https://web.archive.org/web/20100102023608/http://mystaddict.tlayeh.com/Computer%20Camp/Slab6/slab6.txt) - An archive of the mirror of the readme for Slab6 which contains the .kvx file format, the format that the AoS model format is based on
* [VXL File Format Specification](mapformat.html) - A description of the .vxl file format, the format used for AoS maps<br />([original](http://silverspaceship.com/aosmap/aos_file_format.html), [mirror](aos_file_format.html))
