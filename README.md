# StreamLite

StreamLite is a C++17 project for exploring the internals of real-time audio and video communication using WebRTC.

The project focuses on the lower-level systems involved in establishing and maintaining real-time media sessions, including SDP negotiation, ICE/STUN connectivity, RTP/RTCP transport, secure media communication, jitter buffering, packet-loss recovery, and bandwidth estimation.

The goal of StreamLite is to explore how real-time communication systems behave under practical network conditions such as latency, jitter, packet loss, and changing bandwidth.

---

## Features

### WebRTC Connectivity

StreamLite contains the core components required to establish and maintain WebRTC connections, including:

- SDP offer generation and answer parsing
- ICE negotiation
- STUN connectivity checks
- DTLS negotiation
- SRTP and SRTCP
- IPv4 and IPv6 networking
- Peer connection state management
- Multiple media tracks
- WebRTC data channels

The `PeerConnection` abstraction coordinates connection establishment and media transport.

---

## Real-Time Media Transport

StreamLite handles real-time media using RTP and RTCP.

The media pipeline includes:

- RTP packet generation and parsing
- RTCP sender reports
- RTCP receiver reports
- RTP sequence tracking
- Audio/video packetization
- Audio/video depacketization
- Packet retransmission
- RTP timestamp handling
- Media-track statistics

At a high level:

```text
Encoded Media
     |
     v
 Packetizer
     |
     v
 RTP Packets
     |
     v
 DTLS / SRTP
     |
     v
 UDP Network
     |
     v
 SRTP / RTP
     |
     v
 Jitter Buffer
     |
     v
 Depacketizer
     |
     v
Encoded Media Frame
```

---

## Supported Media Formats

The project contains RTP packetization and depacketization support for several commonly used media codecs.

### Video

- VP8
- VP9
- H.264
- H.265
- AV1

### Audio

- Opus

Media encoding and decoding are kept outside the core transport layer. StreamLite primarily works with encoded media frames and handles their transport through RTP.

---

## Jitter Buffer

One of the important components of StreamLite is the receive-side jitter buffer.

Packets transmitted over a real network do not necessarily arrive at perfectly consistent intervals.

They may:

- Arrive late
- Arrive out of order
- Be temporarily delayed
- Be lost completely

The jitter buffer temporarily stores incoming RTP packets before reconstructing the corresponding media frame.

```text
             Network
                |
                v
          RTP Receiver
                |
                v
      +-------------------+
      |   Jitter Buffer   |
      |-------------------|
      | Sequence 101      |
      | Sequence 102      |
      | Sequence 104      |
      +-------------------+
                |
                v
          Depacketizer
                |
                v
       Encoded Media Frame
                |
                v
           Application
```

This introduces an important trade-off in real-time systems.

A larger buffer can tolerate more network variation but introduces additional latency.

A smaller buffer reduces latency but makes playback more sensitive to jitter.

The objective is therefore to balance **responsiveness and media stability**.

---

## Packet Loss and Retransmission

Packet loss is unavoidable on some real-world networks.

StreamLite contains mechanisms for detecting and recovering from missing RTP packets.

The receive side can identify gaps in RTP sequence numbers and generate feedback requesting retransmission.

The project includes support for:

- NACK feedback
- RTX retransmission
- Packet history
- Packet-loss detection
- Key-frame requests
- RTCP feedback

When retransmission is appropriate, previously transmitted RTP packets can be resent to the receiver.

For real-time communication, recovery also needs to consider latency: waiting too long for a missing packet can be worse than continuing playback.

---

## ICE and STUN

Before media can flow between peers, StreamLite must determine how the peers can communicate over the network.

ICE coordinates candidate discovery and connectivity checks, while STUN assists with discovering network addressing information when peers are behind NAT.

Conceptually:

```text
Peer A
   |
   | ICE Candidates
   v
ICE / STUN Negotiation
   ^
   | ICE Candidates
   |
Peer B
```

After connectivity has been established, the selected transport path can be used for real-time media communication.

---

## SDP Negotiation

SDP is used to describe the capabilities of each peer.

StreamLite contains components for:

- Generating SDP offers
- Parsing SDP answers
- Negotiating codecs
- Negotiating media tracks
- Negotiating RTP extensions
- Configuring transport parameters

This allows two WebRTC endpoints to agree on how audio, video, and data will be exchanged.

---

## Secure Media Transport

Real-time media is protected using WebRTC's secure transport mechanisms.

The connection pipeline includes:

```text
ICE
 |
 v
DTLS
 |
 v
SRTP / SRTCP
 |
 v
RTP / RTCP Media
```

DTLS establishes the cryptographic context used for secure RTP communication.

SRTP protects the media packets while SRTCP protects RTCP control traffic.

---

## Bandwidth Estimation

Real-time applications need to react when network capacity changes.

StreamLite contains Transport-Wide Congestion Control components for observing packet-delivery behavior and estimating available bandwidth.

Relevant components include:

- TWCC publishing
- TWCC receiving
- Transport-wide sequence tracking
- Send pacing
- Bandwidth estimation
- Network feedback processing

This information can be used to make better media-sending decisions when network conditions change.

---

## Simulcast

StreamLite contains support for simulcast.

Simulcast allows multiple representations of the same video stream to be transmitted at different qualities.

For example:

```text
Camera
  |
  +----> High Resolution
  |
  +----> Medium Resolution
  |
  +----> Low Resolution
```

A receiver or media infrastructure can then select an appropriate stream according to available bandwidth and device capabilities.

---

## Data Channels

In addition to audio and video, the project contains WebRTC data-channel functionality.

Data channels allow arbitrary application data to be transported alongside real-time media.

The implementation includes SCTP-related components used for reliable data-channel communication.

---

## Architecture

The high-level architecture is:

```text
                    Application
                         |
                         v
                  PeerConnection
                         |
              +----------+----------+
              |                     |
              v                     v
          SDP / ICE             Media Tracks
              |                     |
              v             +-------+-------+
          ICE Agent         |               |
              |             v               v
              |        Packetizer      Depacketizer
              |             |               ^
              |             v               |
              |            RTP        Jitter Buffer
              |             |               ^
              +-------------+---------------+
                            |
                            v
                       DTLS / SRTP
                            |
                            v
                       UDP Network
```

`PeerConnection` acts as the main coordination layer.

It brings together:

- SDP negotiation
- ICE connectivity
- Transport security
- Media tracks
- RTP/RTCP
- Network feedback
- Connection state

---

## Project Structure

```text
StreamLite/
├── include/
│   └── srtc/
│       ├── peer_connection.h
│       ├── ice_agent.h
│       ├── jitter_buffer.h
│       ├── rtp_packet.h
│       ├── rtcp_packet.h
│       ├── sdp_offer.h
│       ├── sdp_answer.h
│       ├── packetizer*.h
│       ├── depacketizer*.h
│       ├── srtp*.h
│       ├── twcc*.h
│       └── ...
│
├── src/
│   ├── peer_connection.cpp
│   ├── ice_agent.cpp
│   ├── jitter_buffer.cpp
│   ├── rtp_packet.cpp
│   ├── rtcp_packet.cpp
│   ├── sdp_offer.cpp
│   ├── sdp_answer.cpp
│   ├── packetizer*.cpp
│   ├── depacketizer*.cpp
│   ├── srtp*.cpp
│   ├── twcc*.cpp
│   ├── sctp/
│   └── stun/
│
├── test/
├── tools/
├── pion-webrtc-examples-whip-whep/
├── CMakeLists.txt
└── LICENSE.txt
```

---

## Building

StreamLite uses CMake.

### Configure

```bash
cmake -S . -B build
```

### Compile

```bash
cmake --build build
```

---

## Testing Network Conditions

Real-time communication should be tested under imperfect network conditions rather than only on a stable local network.

On Linux, `tc/netem` can be used to introduce artificial network degradation.

For example:

```bash
sudo tc qdisc add dev lo root netem delay 50ms 20ms loss 2%
```

This introduces:

- Base network delay
- Variable packet delay
- Packet loss

The configuration can be removed with:

```bash
sudo tc qdisc del dev lo root
```

This type of testing is useful for studying:

- Jitter-buffer behavior
- Packet-loss recovery
- Retransmission
- Latency
- Media stability

---

## Engineering Concepts Explored

StreamLite provides a practical environment for exploring:

- C++ network programming
- WebRTC internals
- SDP negotiation
- ICE
- STUN
- DTLS
- SRTP/SRTCP
- RTP/RTCP
- Audio/video packetization
- Jitter buffering
- Packet ordering
- Packet loss
- NACK
- RTX
- Bandwidth estimation
- TWCC
- Simulcast
- Data channels
- Concurrency
- Low-latency systems

---

## Motivation

Real-time communication behaves differently from traditional request-response applications.

A normal backend request can often tolerate hundreds of milliseconds of additional processing without significantly affecting the user experience.

In a real-time media system, however, relatively small delays can directly affect conversation quality.

StreamLite is intended as a hands-on environment for understanding these trade-offs and exploring how WebRTC systems maintain responsive communication despite imperfect network conditions.

---

## License and Attribution

StreamLite contains or is derived from open-source components. See `LICENSE.txt` and the applicable source-file notices for licensing and attribution requirements.

Where code is derived from another project, the corresponding copyright and license notices should be retained.
