# Message Types

This packet is an extension to the [Chat Message](../protocol075.html#chat-message), it adds new chat types.

| ------------: | ------------ |
| Extension ID: | 193          |
| Version:      | 1            |
| Type:         | `PACKETLESS` |

So clients can handle it how they want, in most clients it will display the message in different area/size/color/sound in
player's screen.

## New Types:

| Value | Type         | Notes                                 |
|-------|--------------|---------------------------------------|
| 3     | CHAT_BIG     | Displayed on the center of the screen |
| 4     | CHAT_INFO    | Displays a notice                     |
| 5     | CHAT_WARNING | Displays a warning                    |
| 6     | CHAT_ERROR   | Displays a error                      |

See [Extensions](extension.html) for how the extension is negotiated.
