#  Gmaestro

*Gmaestro* is a UDP datagram protocol used by [Gamma Control] and [Gamma Board] to talk to each other over the local network.

[Gamma Control]: https://michelf.ca/gamma-control/
[Gamma Board]: https://michelf.ca/gamma-board/

- Status: **This document describes the most useful messages, but is incomplete.**

## General Message Format

Each message is of the following form:
```
"Gmaestro" + space + <message-name> + space + <JSON-payload>
```
- The `Gmaestro` header string identifies the messaging protocol.
- `<message-name>` is the name of the command. It consists of non-whitespace ASCII characters.
- `<JSON-payload>` contains message-specific data encoded using JSON.

Note that the two `space` characters must be exactly one space.

## Messages for Gamma Control

Gamma Control listens for messages on UDP port 44188.

### `state-update` message

Tells Gamma Control to update the state for a list of screens. The command has this form:
```
Gmaestro state-update {
	<screen-id>: {
		"gamma": [
			<blending-mode>,
			<master-black>, <master-middle>, <master-white>,
			<red-black>,    <red-middle>,    <red-white>,
			<green-black>,  <green-middle>,  <green-white>,
			<blue-black>,   <blue-middle>,   <blue-white>
		],
		"content": <calibration-patterns>
	},
	<screen-id>: {
		...
	}
}
```
The payload contains one entry per screen we want to update.

- `<screen-id>`: the unique identifier for each screen, as supplied by the message `state-info` sent by Gamma Control.
- `<blending-mode>`: `0` for absolute, `1` for relative to profile
- `<master-black>`/`<master-middle>`/`<master-white>`: floating point value for the master slider
- `<red-black>`/`<red-middle>`/`<red-white>`: floating point value for the red slider
- `<green-black>`/`<green-middle>`/`<green-white>`: floating point value for the green slider
- `<blue-black>`/`<blue-middle>`/`<blue-white>`: floating point value for the blue slider
- `<calibration-patterns>`: `0` to show no calibration pattern, `1` to show the color calibration patterns, `2` to show the grayscale calibration patterns

After sending a `state-update` message, a `state-info` message is sent back in response. More `state-info` messages will be sent if the state changes in response to the user dismissing the calibration patterns. This will last only for a few seconds, unless a new `state-update` message is received.

Note that you can send a `state-update` message with an empty list of screens in order to receive a `state-info` message as a response. You can also broadcast this message (send to 255.255.255.255) to get a reply from every Gamma Control instance on the local network.

## Messages sent by Gamma Control

When Gamma Control sends a response message, it sends it to the same port and address it was sent from. This is normally UDP port 48184 since this is the port Gamma Board uses to send its messages, but it can be any port.

### `state-info` message

Gamma Control sends the state of each of its screens in a `state-info` message in reply to a `state-update` message. The `state-info` message payload has the same form as the `state-update` message: 
```
Gmaestro state-info {
	<screen-id>: {
		"gamma": [
			<blending-mode>,
			<master-black>, <master-middle>, <master-white>,
			<red-black>,    <red-middle>,    <red-white>,
			<green-black>,  <green-middle>,  <green-white>,
			<blue-black>,   <blue-middle>,   <blue-white>
		],
		"content": <calibration-patterns>
	},
	<screen-id>: {
		...
	}
}
```
The payload contains one entry per screen on the computer Gamma Control is running on.

- `<screen-id>`: the unique identifier for each screen.
- `<blending-mode>`: `0` for absolute, `1` for relative to profile
- `<master-black>`/`<master-middle>`/`<master-white>`: floating point value for the master slider
- `<red-black>`/`<red-middle>`/`<red-white>`: floating point value for the red slider
- `<green-black>`/`<green-middle>`/`<green-white>`: floating point value for the green slider
- `<blue-black>`/`<blue-middle>`/`<blue-white>`: floating point value for the blue slider
- `<calibration-patterns>`: `0` to show no calibration pattern, `1` to show the color calibration patterns, `2` to show the grayscale calibration patterns
