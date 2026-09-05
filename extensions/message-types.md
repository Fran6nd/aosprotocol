# Message Types

Adds new chat types to the [Chat Message](../protocol075.html#chat-message)
packet.

| ------------: | ------------ |
| Extension ID: | 193          |
| Version:      | 1            |
| Type:         | `PACKETLESS` |

Clients can handle the new types however they want. In most clients the message
will be displayed in a different area, size, colour or with a different sound in
the player's screen.

## New Types

| Value | Type         | Notes                                 |
|-------|--------------|---------------------------------------|
| 3     | CHAT_BIG     | Displayed on the center of the screen |
| 4     | CHAT_INFO    | Displays a notice                     |
| 5     | CHAT_WARNING | Displays a warning                    |
| 6     | CHAT_ERROR   | Displays a error                      |

See [Extensions](../protocol075.html#extensions) for how the extension is
negotiated.
