# SSL-Messager 

A basic cyphered tool for messaging.

## System

main.py file is the server, and client.py file is obviously the client.
user.csv file is to check for username and password. After users log in,
they choose a target to send messages to. If the user is online, messages
are sent. If not, a harmless error message is printed.

There is no peer to peer connection. Everybody is connected to the server,
and sends messages to the server. Server then relays these messages to
intended targets. One can only send message to one person at this version,
but can receive from multiple users.

Self-signed certificates are required.

A thread is created for each user in the server. This thread handles both
receiving from this user, and sending messages to targets. Also another thread
is created in the server to check if the connections are still alive. If not,
closes server-sided sockets and as a resulting exception, related threads return.

In the client side 2 threads exist. One is for receiving and printing out messages,
the other is for getting input from the user and sending messages.

## Protocol

Without any user online, server just sits silently. When a user is trying to log in,
following sentence is sent:

`AUTH username password \r\n`

In response, if accepted, server answers with the following reply (client receives its token):

`ACCEPT username token \r\n`

If not accepted:

`END <incorrect username or password> \r\n`

To log in, proper syntax is below:

`PUT username password \r\n`

If username already taken, servers response:

`TRY <username already taken> \r\n`

If not, server just sends the _ACCEPT_ query and logs in the user.

To send messages, client sends the following query with its session token:

`MSG target username message token \r\n`

When group chat is enabled:

`MSGG username message token \r\n`

Server redirects this message to the target(s) in the exact same order, except the first word:

`RELAY target username message \r\n`

When group chat is enabled by the sender:

`RELAYG username message \r\n`

To check if a client socket is online, server tries to send the following message (no response needed):

`CHECK \r\n`

If any sent query is not in the form stated above, server sends the following query and closes down
the socket unilaterally:

`END * <incorrect protocol> \r\n`

IF user inputs the command ":quit:", client sends the END command:

`END <user command> token \r\n`

Server replies with:

`END <end accepted> \r\n`

### Important notes on the protocol

- MSG, PUT and AUTH flagged messages can only be sent from the client.
- END can be sent by everyone
- All other messages are sent by the server.
- Every query ends with "\r\n" but this does not serve a purpose for now.
- Server messages about errors are written inside <> sings
- Server sends the token once whilst authorizing
- Client sends its token at each query for validation


## Commands

### :quit:

Closes down the connection by sending END query then quits the program

### :online:

Shows top 100 online users. This limit is for practical reasons.

### :new_target:

Allows user to change targets. Client thread does not get killed. Just
the target changes.

### :toggle:

Toggles the group chat. At log in, group chat is not enabled.

## Notes

### Major security issue solved (22 June 2023)

When trying to sign up, if already taken username inputted, server used to put
the user in *object_list* temporarily by their ip. If another user, logged in,
typed :online: whilst this process, they could see this ip. 

Ip is now switched with a 64-bit random number given as a temporary name.