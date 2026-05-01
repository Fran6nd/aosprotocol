## Extension Negotiation Packet

### Overview

Each extension is given an unique id that is decided when it is first
registered. We differentiate between two types of packets:

*Those that:*

| Type          | Purpose                               | Extension id range |
| ------------- | ------------------------------------- | ------------------ |
| `HAS_PACKETS` | introduce new packets to the protocol | 0-191              |
| `PACKETLESS`  | don't use and need any packets        | 192-255            |

Each extension is given one legacy packet id equal to `64+extension_id`.
For `PACKETLESS` extensions this would mean that their packet ids are out
of spec `>255`, thus they don't have any.
An example for a packetless extension would be *OpenSpades'* UnicodeExt.

Each extension packet will contain 1 additional byte in its data, which is a
subpacket id, used to have multiple packets available for each extension. This
is always the case, even if an extension only needs 1 packet in total.

General extension packet structure:

| Field name    | Type      | Notes          |
| ------------- | --------- | -------------- |
| Packet id     | UByte     | 64-255         |
| Sub packet id | UByte     | 0-255          |
| Data          | UByte[]   | extension data |

### ExtInfo Packet

* Packet ID: 60
* Total size: `2+2*length`

| Field name | Field type   | Notes                        |
| ---------- | ------------ | ---------------------------- |
| length     | UByte        | `length` entries will follow |
| entries    | ExtInfoEntry | see below                    |

**ExtInfoEntry**

| Field name | Field type | Notes               |
| ---------- | ---------- | ------------------- |
| ext. ID    | UByte      | see #Overview       |
| version    | UByte      | Usually starts at 1 |

## Protocol Flow

The server should send an `ExtInfo` packet (optimally) after the Version Info response has been received to compatible clients
(OpenSpades versions > 0.1.3, see https://github.com/piqueserver/piqueserver/issues/504),
assuming it supports any. The client can store the list of extensions for later use and should
reply with an `ExtInfo` packet that lists the extensions it supports (if it does actually support any).

The client can omit any extensions that the server does not support from its
reply, but this is not necessary as the server can simply ignore them itself.


# HAS_PACKETS Packets
* [Player Properties](#player-properties)
* [Ed25519 Authentication](#ed25519-authentication)

## Player Properties

Sends additional player attributes from the server to the client.

| ---------: |-----|
| Packet ID: | 64  |
| Version:   | 1   |

| Sub ID | Name              | Direction        | Size |
|--------|-------------------|------------------|------|
| 0      | Player Properties | Server -> Client | 12   |

## Ed25519 Authentication

Authenticates a connecting client using an Ed25519 key pair. The server requests
authentication and sends a nonce, the client replies with its public key and a
signature over the nonce, and the server ends authentication once the signature
has been verified.

| ---------: |-----|
| Packet ID: | 65  |
| Version:   | 1   |

#### Sub Packets:

| Sub ID | Name                   | Direction        | Size |
|--------|------------------------|------------------|------|
| 0      | Request Authentication | Server -> Client | 2    |
| 1      | End Authentication     | Server -> Client | 4+   |
| 2      | Send Public Key        | Client -> Server | 34   |
| 3      | Send Nonce             | Server -> Client | 3+   |
| 4      | Send Signature         | Client -> Server | 66   |


# Packetless Packets
* [Player Limit](#player-limit)
* [Message Types](#message-types)
* [Kick Reason](#kick-reason)

## Player Limit

Tells client server supports up to 256 players.

| ---------: |-----|
| Packet ID: | 192 |
| Version:   | 1   |

## Message Types

This packet is an extension to the [Chat Message](protocol075.html#chat-message), it adds new chat types.
So clients can handle it how they want, in most clients it will display the message in different area/size/color/sound in
player's screen.

| ---------: |-----|
| Packet ID: | 193 |
| Version:   | 1   |

#### New Types:

| Value | Type         | Notes                                 |
|-------|--------------|---------------------------------------|
| 3     | CHAT_BIG     | Displayed on the center of the screen |
| 4     | CHAT_INFO    | Displays a notice                     |
| 5     | CHAT_WARNING | Displays a warning                    |
| 6     | CHAT_ERROR   | Displays a error                      |

## Kick Reason

Send a [Chat Message](protocol075.html#chat-message) with type 2 (CHAT_SYSTEM) and player id 255, before
kicking a player out of the server.

| ---------: |-----|
| Packet ID: | 194 |
| Version:   | 1   |

# Other Extensions
* [UTF-8 Chat](#utf-8-chat)

## UTF-8 Chat

Has no registered extension id. A [Chat Message](protocol075.html#chat-message)
whose data starts with a `0xff` byte is encoded in UTF-8 instead of Code Page
437. Messages without the prefix are interpreted as Code Page 437 as in the
base protocol.