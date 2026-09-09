# StreamLite

StreamLite is an experimental real-time audio and video communication
project built to explore WebRTC, peer-to-peer communication, signaling,
network jitter, packet loss, and low-latency media delivery.

The primary goal of the project is to understand the engineering
challenges behind real-time communication systems rather than relying
entirely on managed communication platforms.

## Architecture

StreamLite separates signaling from media communication.

                     ┌─────────────────────┐
                     │  Signaling Server   │
                     │ Node.js/TypeScript  │
                     │     WebSocket       │
                     └──────────┬──────────┘
                                │
                    SDP / ICE signaling
                                │
                 ┌──────────────┴──────────────┐
                 │                             │
          ┌──────▼──────┐               ┌──────▼──────┐
          │   Client A  │◄──────────────►│   Client B  │
          │             │    WebRTC      │             │
          │ C++ media   │   Audio/Video  │ C++ media   │
          │ processing  │                │ processing  │
          └─────────────┘                └─────────────┘

The signaling server helps peers discover each other and exchange
connection information.

After negotiation completes, media is transmitted through the WebRTC
peer connection rather than through the signaling server.

## Core Components

### WebRTC Communication

StreamLite uses WebRTC for low-latency audio and video communication
between peers.

The connection process includes:

1. Peer discovery through the signaling server
2. SDP offer/answer exchange
3. ICE candidate exchange
4. NAT traversal using ICE/STUN
5. Peer connection establishment
6. Real-time audio/video streaming

TURN can be used as a relay when a direct peer-to-peer connection
cannot be established.

## Signaling Server

The signaling layer is implemented using Node.js, TypeScript and
WebSockets.

Its responsibilities include:

- Registering connected peers
- Coordinating peer discovery
- Forwarding SDP offers
- Forwarding SDP answers
- Forwarding ICE candidates
- Handling peer disconnects

The signaling server does not need to process the actual audio/video
stream once the WebRTC connection has been established.

## C++ Media Processing

StreamLite contains a C++ component for experimenting with lower-level
real-time media processing.

The main areas explored include:

- Packet sequencing
- Packet buffering
- Jitter handling
- Packet-loss behavior
- Concurrent packet processing
- Playback timing

Keeping this processing close to the receiving client allows buffering
decisions to be based on the network conditions experienced by that
specific peer.

## Jitter Buffer

Real-time packets do not necessarily arrive at perfectly consistent
intervals.

Network conditions can cause:

- Variable packet delay
- Out-of-order packets
- Packet loss
- Temporary latency spikes

StreamLite experiments with a small client-side jitter buffer.

Conceptually:

Network
   |
   v
Packet Receiver
   |
   v
+----------------+
| Jitter Buffer  |
|                |
| seq: 101       |
| seq: 102       |
| seq: 104       |
|      ...       |
+----------------+
   |
   v
Ordered Media
   |
   v
Playback

Packets are temporarily buffered and ordered using packet sequence
information.

If an expected packet does not arrive within the allowed buffering
window, the receiver can treat that packet as lost and continue
processing subsequent packets rather than waiting indefinitely.

## Latency vs Stability

One of the main experiments in StreamLite is understanding the
trade-off between buffering and latency.

A larger jitter buffer:

- Handles larger variations in packet arrival time
- Produces more stable playback
- Adds additional latency

A smaller jitter buffer:

- Reduces latency
- Improves responsiveness
- Is more sensitive to network jitter

The objective is therefore not simply to minimize buffering but to find
a reasonable balance between responsiveness and playback stability.

## Network Testing

Network degradation can be simulated during development using Linux
traffic-control tools.

For example:

```bash
sudo tc qdisc add dev lo root netem delay 50ms 20ms loss 2%
