<p align="center">
  <img src="https://raw.githubusercontent.com/Lartu/mitsubachi/master/images/mitsubachi-logo.png">
  <br>
  <img src="https://img.shields.io/badge/license-MIT-gold?style=flat-square">
  <img src="https://img.shields.io/badge/version-1.0-green.svg?style=flat-square">
  <img src="https://img.shields.io/badge/miba-!mitsubachi-orange.svg?style=flat-square">
</p>

**Mitsubachi** is a tiny and open chat protocol, designed to be minimal, easy to implement,
easy to use and easy to understand. Mitsubachi supports nicknames, nickname changing,
user-to-user messaging and channel / group (known in Mitsubachi as "_distribution lists_") chat.

I designed the Mitsubachi protocol because I found that most common, open chat protocols
(IRC, XMPP, etc.) are too complex to implement properly and completely. Writing a very
bare-bones IRC client is easy, but implementing the whole protocol is not. XMPP messages
are way to heavy to be reliable on high-latency, low-bandwidth networks. The protocol is
also very complex for an amateur, casual client / server writer to implement. Mitsubachi
aims to address all these issues.

## Protocol tl;dr

```
Connection is done via plain sockets on TCP port 7107.
Messages between client and server are divided in 5 sections
separated by a single space. Messages end with a \n character.

(Message format)
+---------+------------+------------+-----------+----------+
| command |   sender   | recipient  | extradata | content  |
+---------+------------+------------+-----------+----------+
| 4 chars | 1-32 chars | 1-32 chars | >1 chars  | >1 chars |
+---------+------------+------------+-----------+----------+

All clients and lists have a nick that identifies them. Nicks
are unique. List nicks begin with the character !.

Clients may join and leave lists. Messages sent to a client
are delivered to that client. Messages sent to lists are sent
to every client on that list.

Valid commands:
> 'NICK <nick> # # #\n' used to change your nick
> 'JOIN # <list_nick> # #\n' used to join a list
> 'LEAV # <list_nick> # #\n' used to leave a list
> 'MESG <sender_nick> <recipient_nick> # <content>\n' used to
  send an actual message (for example <content> = Hi there!)
> 'EXIT # # # #\n' used to close the connection (sent by clients)
> 'OOPS # # <error_code> #\n' used to tell a client the result of
  an operation (sent by servers).
> 'INFO # # # <content>\n' used by servers to send text messages
  to clients.
  
Error codes:
A server should reply then 000 ("ok") when a client changes success-
fully their nick. Otherwise it should reply 001 ("nick already in
use") or 002 ("invalid nick", for nicks beginning in !). 003 should
be sent when trying to join an invalid list (using a client nick).
005 should be sent when a client tries to send a message to a list
they are not members of. 006 should be sent when an invalid command
is received by the server. 007 should be sent to clients that have
not chosen a nick when the server has received a command from them
other than NICK.
```

## Protocol Details

By default and standard, Mitsubachi servers listen for connections on **TCP port 7107**. Clients
(users) connect to a server by plain socket connection. 

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

The minimum length of any message is 13: `XXXX # # # #\n".

**Nicks** identify users. Each user has a nick (or nickname) and no two users can have the same
nickname. When a user disconnects from the server, their nickname becomes available to be used
by other user. Nicknames have a minimum length of **1 character** and a maximum length of
**32 characters**. Nicknames that start with the character `!` identify **lists**. As such,
user nicknames cannot contain this character in the first position. Nicks can contain any
character except spaces.

**Lists** are lists of users. They have a nickname just like any other user, but messages sent to
a list are sent to every member of the list. In order to be able to send a message to a list, a
client must be part of said list (otherwise error message 005 ("you are not a member of this list") should be sent to the client). When a message is sent to a list, it is broadcast to every member
of the list _except_ the one who sent the message. Users may join and leave lists at any time.
When a client tries to join a non-existant list, it is created.

All this said, the following messages are valid Mitsubachi messages:
 - **NICK**: sent by a client to a server to change their nick. If the nick is in use by another client, the server should
 reply `OOPS # # 001 #` ("nick already in use"). It the nick is available to be used, the server should reply `OOPS # # 000 #` ("ok") and assign the nick to the client.
   - Command section value: `NICK`
   - Sender section value: _the desired nick_
   - Recipient section value: `#`
   - Extradata section value: `#`
   - Content section value: `#`
   - Example: `NICK myNick # # #`
 - **JOIN**: sent by a client to a server to join a list. If the nick of the list to be joined is not a valid list nick, the server should
 reply `OOPS # # 003 #` ("invalid distribution list"). If the list is valid, the client should be joined to the list and the server should reply `OOPS # # 000 #` ("ok").
   - Command section value: `JOIN`
   - Sender section value: `#`
   - Recipient section value: _nick of the list to be joined_
   - Extradata section value: `#`
   - Content section value: `#`
   - Example: `JOIN # !mitsubachi # #`
 - **LEAV**: sent by a client to a server to leave a list. If the nick of the list to be left is not a valid list nick, the server should
 reply `OOPS # # 003 #` ("invalid distribution list"). If the list is valid, the client should be removed from the list and the server should reply `OOPS # # 000 #` ("ok").
   - Command section value: `LEAV`
   - Sender section value: `#`
   - Recipient section value: _nick of the list to be left_
   - Extradata section value: `#`
   - Content section value: `#`
   - Example: `LEAV # !lamelist # #`
- **MESG**: sent by a client to a server to deliver a message to another client or list. When this
message is sent by the server to a client it means that said client has received a message.
   - Command section value: `MESG`
   - Sender section value: _nick of the sender of the message_
     - When a client is the one that sends this message to a server, it may omit the contents 
     of this section and just send a `#` in it instead. When this message is sent by a server
     to a client, the content of this section is always the nick of the user who sent the message
     originally.
   - Recipient section value: _nick of the recipient of the message (user or list)_
   - Extradata section value: `#`
   - Content section value: _actual contents of the message_
   - Example: `MESG user1 coolUser # Hello there, coolUser, how are you?`
 - **EXIT**: sent by a client to a server to close the connection.
   - Command section value: `EXIT`
   - Sender section value: `#`
   - Recipient section value: `#`
   - Extradata section value: `#`
   - Content section value: `#`
   - Example: `EXIT # # # #`
 - **OOPS**: sent by a server to a client to inform the result of a requested operation. This message
 shouldn't be sent by clients.
   - Command section value: `OOPS`
   - Sender section value: `#`
   - Recipient section value: `#`
   - Extradata section value: _error code that represents the result of the requested operation_
   - Content section value: `#`
   - Example: `OOPS # # 000 #`
 - **INFO**: an information message from the server with a text message. This is supposed to be read
 by the user. It may contain a MOTD or any valuable information the server administrator desires to
 send clients.
   - Command section value: `INFO`
   - Sender section value: `#`
   - Recipient section value: `#`
   - Extradata section value: #
   - Content section value: _the contents of a message_
   - Example: `INFO # # # Welcome to Mitsubachi!`

In order to request any command to a server, clients must first have chosen a nick (by issuing a `NICK` message). Should
they try to send any other message to the server other than a `NICK` command, the server should reply `OOPS # # 007 #` ("please log in").

Should a message not in the list above be received by a server, it should reply `OOPS # # 006 #` ("unknown command").

## Mitsubachi Server

A Mitsubachi Server written in [LDPL](https://github.com/lartu/ldpl) is uploaded to this repository. You are allowed to
use and modify it in any way you see fit. You are encouraged to host your own Mitsubachi servers.

In order to build the server included in this repository you must run the following commands:

```
lpm install ldpl_net_server
lpm install std-text
ldpl mitsubachi-server.ldpl -o=mitsubachi-server
```

## License

The Mitsubachi protocol is released under the MIT license and may freely reproduced and translated. The server included
in this repository is released under the MIT license. The Mitsubachi bee logo is released under the MIT license and may
be freely reproduced on any medium.
