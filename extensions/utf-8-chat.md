# UTF-8 Chat

Allows [Chat Message](../protocol075.html#chat-message) packets to carry UTF-8
text instead of Code Page 437. Known as UnicodeExt in *OpenSpades*.

| ------------: | ---------------- |
| Extension ID: | none (see below) |
| Version:      | n/a              |
| Type:         | unregistered     |

A [Chat Message](../protocol075.html#chat-message) whose data starts with a
`0xff` byte is encoded in UTF-8 instead of Code Page 437. Messages without the
prefix are interpreted as Code Page 437 as in the base protocol.

This extension has never been given an extension id, so it is not announced
through the [ExtInfo packet](extension.html#extinfo-packet) and support for it
has to be assumed or detected some other way.
