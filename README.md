<p align="center">
  <img src="https://raw.githubusercontent.com/Lartu/mitsubachi/master/images/mitsubachi-logo.png">
  <br>
  <img src="https://img.shields.io/badge/license-MIT-gold?style=flat-square">
  <img src="https://img.shields.io/badge/version-1.0-green.svg?style=flat-square">
  <img src="https://img.shields.io/badge/miba-!mitsubachi-orange.svg?style=flat-square">
</p>

**Mitsubashi** is a tiny and open chat protocol, designed to be minimal, easy to implement,
easy to use and easy to understand. Mitsubashi supports nicknames, nickname changing,
user-to-user messaging and channel / group (known in Mitsubashi as "_message lists_") chat.

I designed the Mitsubashi protocol because I found that most common, open chat protocols
(IRC, XMPP, etc.) are to complex to implement properly and completely. Writing a very
bare-bones IRC client is easy, but implementing the whole protocol is not. XMPP messages
are way to heavy to be reliable on low-latency, low-bandwidth networks. The protocol is
also very complex for an amateur, casual client / server writer to implement. Mitsubashi
aims to address all these issues.

## The Protocol

By default and standard, Mitsubashi servers listen for connections on **port 7107**. Clients
connect to a server by plain socket connection.

Exchange between server and client is handled via **messages**. A message is a string sent
from a client to the server or from the server to a client. Messages have a maximum length
of **1024 bytes** (1024 characters), are divided in **5 sections** and follow the format
```
command sender recipient extradata content\n
```
Note the `\n` at the end. This character is used to delimit the end of the message and thus
cannot be used anywhere else in the message.

Each section of a message represents a different type of information:
- The `command` part tells
the recipent of the message what we intend to do with this message (for example, change our nick or join a list).
  - This section has a fixed length of **4 characters**.
  - This section is always in **UPPERCASE**.
- The `sender` part tells the recipient **who** is this message **from**.
  - This section has a minimum length of **1 character**.
  - This section has a maximum length of **32 characters**.
  - This section is always in **lowercase**.
  - This section may not contain any spaces.
- The `recipient` part tells **who** was this message sent **to**.
  - This section has a minimum length of **1 character**.
  - This section has a maximum length of **32 characters**.
  - This section is always in **lowercase**.
  - This section may not contain any spaces.
- The `extradata` section is a reserved part of the message used for sending error codes or other
data.
  - This section has a minimum length of **1 character**.
  - This section has no maximum length.
  - This section is always in **lowercase**.
  - Multiple words in this section must be delimited by the character `|`.
  - This section may not contain any spaces.
- The `content` part of the message is used for sending _messages_ per sé, as in the content of
a message (_"hi there!"_, _"how are you?"_).
  - This section has a minimum length of **1 character**.
  - This section has no maximum length.
  - This section may contain spaces.
  
When a section is supposed to be empty (for example, when sending a message to the server that
has no recipient) it must be filled with a single `#` character.

**Nicks** identify users. Each user has a nick (or nickname) and no two users can have the same
nickname. When a user disconnects from the server, their nickname becomes available to be used
by other user. Nicknames have a minimum length of **1 character** and a maximum length of
**32 characters**. Nicknames that start with the character `!` identify **lists**. As such,
user nicknames cannot contain this character in the first position.

**Lists** are lists of users. They have a nickname just like any other user, but messages sent to
a list are sent to every member of the list. In order to be able to send a message to a list, a
user must be part of said list. When a message is sent to a list, it is broadcast to every member
of the list _except_ the one who sent the message. Users may join and leave lists at any time.
When a user tries to join a non-existant list, it is created.

