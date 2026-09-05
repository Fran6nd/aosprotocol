# Kick Reason

Repurposes the chat to send a disconnect reason text before a player is kicked.

| ------------: | ------------ |
| Extension ID: | 194          |
| Version:      | 1            |
| Type:         | `PACKETLESS` |

The server sends a [Chat Message](../protocol075.html#chat-message) with type 2
(`CHAT_SYSTEM`) and player id 255, before kicking a player out of the server.

The vanilla [disconnect reasons](../protocol075.html#disconnect-reasons) only
carry a single number, so this extension is the way to hand the player a
human-readable explanation.

See [Extensions](../protocol075.html#extensions) for how the extension is
negotiated.
