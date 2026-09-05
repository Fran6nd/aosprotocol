# UTF-8 Chat

| ------------: | ------------ |
| Extension ID: | none         |
| Type:         | unregistered |

Has no registered extension id. A [Chat Message](../protocol075.md#chat-message)
whose data starts with a `0xff` byte is encoded in UTF-8 instead of Code Page
437. Messages without the prefix are interpreted as Code Page 437 as in the
base protocol.
