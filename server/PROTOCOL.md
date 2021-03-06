# Client-server protocol

This document lists the various messages that can be sent to and by the server. Client-server connection is made via the WebSocket API. **As game development is still unstable, this protocol is liable to change without warning.**

## General form

Messages are of the form

`messagename|parameter1|parameter2|...etc...|lastparameter`.

If the message has no parameters, the message is simply

`messagename`.

## Messages sent to the server

### username

This message does not follow the general message form. The client must send a `username` message as the very first message of the connection. The content of the message is simply the client's username. For example, if the client's username were `foo`, the message should be

`foo`.

### move

Parameters (1): `move_str`.

This message tells the server to move the player in the direction specified by `move_str`. `move_str` must be zero or more of the following letters: `l`, `r`, `u`, and `d`, standing for left, right, up, and down. The order of the letters is not important. Duplicate letters are ignored. See [movedto](#movedto) for the server's response.

### fastmove

Parameters (1): `move_str`.

This message is similar to `move`, except that it is used for fast-walking. See [move](#move) for an explanation of the `move_str` parameter.

### interact

No parameters.

This message tells the server that the player wishes to interact, e.g. with a sign or a non-player character.

### dialoguechoose

Parameters (1): `uuid`, `choice`.

This message should be sent after a [dialoguechoice](#dialoguechoice) message. It indicates which option the client has chosen to say. The parameter `uuid` is the UUID of the entity with which the player is talking. The parameter `choice` is a zero-indexed number (i.e. to pick the first option, the client should send `0`) telling the server which option the client has picked, out of the options given by the [dialoguechoice](#dialoguechoice) message.

### getupdates

No parameters.

This message retrieves the usernames of other players and their locations, as well as the state of the entities in the game. See [players](#players) and [entities](#entities) for details on the response.

### battlemove

Parameters (1): `move_num`

This message tells the server what move to use after the client has received a [battlemoverequest](#battlemoverequest) message. The parameter `move_num` is a zero-indexed number (i.e. to pick the first move listed, the client should send `0`).

## Messages sent by the server

### world

Parameters (1): `world_str`.

This message is sent when the player first joins (in response to the [username](#username) message) or when the player changes worlds, e.g. through a portal. The `world_str` parameter is a JSON object governed by the transmission to client format detailed in WORLDSTRUCTURE.md.

### movedto

Parameters (2): `x_pos`, `y_pos`.

This message is sent in response to the client's [move](#move) and [fastmove](#fastmove), although the server may send this message at any time. The parameters `x_pos` and `y_pos` are numbers denoting the player's new position.

### signtext

Parameters (1): `sign_text`.

This message is sent when the player reads a sign (possibly in response to the [interact](#interact) message). The `sign_text` parameter gives the text of the sign. Note that `sign_text` may contain `|` characters.

### players

Parameters (Variable number, given by number of other players * 3): `username1`, `x_pos1`, `y_pos1`, `username2`, `x_pos2`, `y_pos2`, etc.

This message is sent in response to the [getplayers](#getplayers) message. For each player in the same world as the client, three parameters are given: the player's username, the player's x position, and the player's y position. The list of players returned excludes the client.

### entities

Parameters (Variable number, given by number of entities): `entity1`, `entity2`, etc.

This message is sent in response to the [getplayers](#getplayers) message. For each non-player entity in the same world as the client, one parameter is given: a JSON representation of the entity.

### dialogue

Parameters (1): `uuid`, `dialogue_text`

This is one of three possible messages sent when the player interacts with an NPC with dialogue lines (possibly in response to the [interact](#interact) message). It indicates that the NPC with UUID `uuid` has spoke the text given in `dialogue_text`. The `dialogue_text` parameter gives a line of the NPC's dialogue. Note that `dialogue_text` may contain `|` characters.

### dialoguechoice

Parameters (Variable number, given by number of choices plus one): `uuid`, `choice1`, `choice2`, etc.

This is one of three possible messages sent when the player interacts with an NPC with dialogue lines (possibly in response to the [interact](#interact) message). It indicates that the player can respond to the NPC with UUID `uuid` with either `choice1`, `choice2`, or etc. None of the parameters will contain `|` characters. The client chooses an option using the [dialoguechoose](#dialoguechoose) message.

### dialogueend

Parameters (1): `uuid`

This is one of three possible messages sent when the player interacts with an NPC with dialogue lines (possibly in response to the [interact](#interact) message). It indicates that the previous message the NPC with UUID `uuid` sent was its last line.

### tag

Parameters (2): `tagging_player`, `tagged_player`

This message is sent when a player with username `tagging_player` sends an `interact` message while touching the player with username `tagged_player`. The message is sent both to the tagging player and to the tagged player.

### battlestart

No parameters.

This message is sent when the player is engaged in a battle, e.g. in a wild-grass encounter.

### battlemovereq

Parameters (Variable number, given by number of moves): `move1`, `move2`, etc.

This message is sent when the server is waiting for the player to choose a move. A [battlemove](#battlemove) message should be sent back.

### battlestatus

Parameters (2): `player_hp`, `enemy_hp`

This message is sent after a player makes a move to indicate the HPs of the combatants. THIS MESSAGE IS VERY LIKELY TO CHANGE AS NEW FEATURES GET ADDED.

### battleend

No parameters.

This message is sent when the battle that the player is engaged in ends.

### death

No parameters.

This message is sent when the player dies.
