# dbot-irc design plan

## Aim

To develop the backend for a dbot module which will allow users to log into and
use a persistent IRC session through the dbot web interface.

## Design

#### Use Case

1. The delightful danharibo creates a persistent web session from DBot.
2. He then wanders off to eat some delicious fried chicken from Lip Lickin'
     Chicken
3. Returning to his flat, he decides to post a review of his chicken in an IRC
     room, so he logs into his persistent web session on DBot web.

#### Session Creation

dbot-irc will be initialised with the configuration options 
*session_manager_port* and *session_port_range*. The DBot websession module 
will keep open an socket connection to the session manager.

1. User requests a persistent session from DBot.
2. DBot sends a request for a new persistent session through the session-manager
     stream.
3. dbot-irc will create the session, and report back with data including the
     port at which this session can be found (or an error).

#### Session Persistance

dbot-irc will persist IRC connections for users, and the default state for a
connection will be 'away,' meaning the user is not currently physically using
the session. In this state, the connection will be kept open by dbot-irc keeping
a 'state' of the IRC session usable by the web client when it's interested. It
will also keep a socket open on the specified session port waiting for a
connection to indicate that it wishes to use the session.

#### Session Use

1. User logs into the web client.
2. Dbot uses the port it has recorded to make a connection to the dbot-irc
     socket concerning the user's session.
3. After indicating to dbot-irc that it wishes to use the session, dbot-irc will
     communicate a 'current state' object which will be used to populate the web
     client initially.
4. Once the state transfer is complete, and dbot indicates it is done receiving
    the state, the session connection will begin to stream the IRC protocol
    through the socket, which will be used by the web client to operate the
    connection and receive updates.

## why dnt u jst do it in dbt??/

The DBot software itself already has plenty to take care of performing its
regular botting activities - multiplexing persistent JSBot instances inside DBot
would quickly become an IO problem.

Furthermore, this design allows possibilites in terms of future scaling; if one
dbot-irc instance starts experiencing performance problems, the port-per-session
model means that one could simply run more instances of dbot-irc. It also means
that dbot-irc instances could be potentially run on other hosts.

## Future Issues

* IRC servers don't like it when you connect a lot of sessions at once. They
    will get sad and ban the host. We'll have to ask for exceptions.

## Possible Alternatives
* Perhaps it would be a good idea to only start listening on a port for a
    session when a user wants to log into it. This intent can be communicated
    through the session-manager connection.
