StreamLite

StreamLite is a C++17 real-time media and WebRTC project focused on low-latency audio/video transport, RTP packet processing, network resilience, and peer-to-peer communication.

The project explores the lower-level components involved in establishing and maintaining WebRTC sessions, including SDP negotiation, ICE/STUN connectivity, secure RTP transport, packetization, jitter buffering, retransmission, and bandwidth estimation.

Features

WebRTC Connectivity

StreamLite includes the core components required for establishing WebRTC connections:

* SDP offer generation and answer parsing
* ICE candidate discovery and negotiation
* STUN connectivity checks
* DTLS negotiation
* SRTP/SRTCP media protection
* IPv4 and IPv6 networking
* Peer connection state management

The main peer connection logic is implemented through the PeerConnection abstraction.

Real-Time Media Transport

The media pipeline handles RTP/RTCP communication for real-time audio and video.

Implemented components include:

* RTP packet generation and parsing
* RTCP sender and receiver reports
* RTP sequence tracking
* Media packetization
* Media depacketization
* Packet retransmission support
* RTP timestamp handling
* Track-level statistics

Audio and Video Codecs

The project contains packetization/depacketization support for several commonly used real-time media formats.

Video

* VP8
* VP9
* H.264
* H.265
* AV1

Audio

* Opus

Media encoding and decoding are kept separate from the transport layer. The WebRTC layer operates primarily on encoded media frames and handles their conversion to and from RTP packets.

Jitter Buffer

One of the important components of StreamLite is its receive-side jitter buffer.

Real-time networks can cause packets to:

* Arrive late
* Arrive out of order
* Arrive with inconsistent timing
* Become lost in transit

The jitter buffer temporarily holds incoming RTP packets so that media frames can be reconstructed in the correct order.

Conceptually:

Network
   |
   v
RTP Receiver
   |
   v
+------------------+
|   Jitter Buffer  |
|                  |
| Sequence 101     |
| Sequence 102     |
| Sequence 104     |
+------------------+
   |
   v
Depacketizer
   |
   v
Encoded Media Frame
   |
   v
Application

This introduces an important real-time systems trade-off:

larger buffer → greater tolerance to jitter but higher latency

smaller buffer → lower latency but greater sensitivity to network variation

The implementation therefore balances playback stability against real-time responsiveness.

Packet Loss and Retransmission

StreamLite contains mechanisms for handling packet loss during real-time communication.

The receive pipeline can identify missing RTP packets and use RTCP feedback mechanisms such as NACK to request retransmission.

RTX can be used when negotiated between peers.

The send side maintains RTP packet history so packets can be retransmitted when required.

Bandwidth Estimation

The project includes Transport-Wide Congestion Control (TWCC) components for estimating network conditions.

TWCC feedback can be used to observe packet delivery behavior and adapt sending decisions based on available network capacity.

Related components include:

twcc_publish
twcc_subscribe
rtp_responder_twcc
send_pacer

Simulcast

StreamLite contains support for multiple video layers through simulcast.

Different versions of a video stream can be transmitted at different resolutions or bitrates, allowing the receiver or media infrastructure to select an appropriate layer according to network conditions.

Data Channels

In addition to audio and video transport, the project contains SCTP and WebRTC data-channel functionality.

This allows arbitrary application data to be exchanged alongside media.

Architecture

At a high level:

                    Application
                         |
                         v
                  PeerConnection
                         |
          +--------------+--------------+
          |                             |
          v                             v
     SDP / ICE                     Media Tracks
          |                             |
          v                    +--------+--------+
      ICE Agent                |                 |
          |                    v                 v
          |               Packetizer       Depacketizer
          |                    |                 ^
          |                    v                 |
          |                   RTP          Jitter Buffer
          |                    |                 ^
          +--------------------+-----------------+
                               |
                               v
                         DTLS / SRTP
                               |
                               v
                         UDP Network

The PeerConnection coordinates negotiation and connectivity, while the media pipeline handles packetization, transport, buffering, and reconstruction of encoded media.

Project Structure

StreamLite/
├── include/srtc/
│   ├── peer_connection.h
│   ├── ice_agent.h
│   ├── jitter_buffer.h
│   ├── rtp_packet.h
│   ├── rtcp_packet.h
│   ├── packetizer*.h
│   ├── depacketizer*.h
│   ├── srtp*.h
│   ├── sdp_offer.h
│   ├── sdp_answer.h
│   ├── twcc*.h
│   └── ...
│
├── src/
│   ├── peer_connection.cpp
│   ├── ice_agent.cpp
│   ├── jitter_buffer.cpp
│   ├── rtp_packet.cpp
│   ├── rtcp_packet.cpp
│   ├── packetizer*.cpp
│   ├── depacketizer*.cpp
│   ├── srtp*.cpp
│   ├── sdp_offer.cpp
│   ├── sdp_answer.cpp
│   ├── twcc*.cpp
│   ├── sctp/
│   └── stun/
│
├── tools/
├── test/
├── pion-webrtc-examples-whip-whep/
├── CMakeLists.txt
└── LICENSE.txt

Building

StreamLite uses CMake.

Configure the project:

cmake -S . -B build

Build:

cmake --build build

The resulting binaries and tools are generated inside the build directory.

Testing Network Conditions

Real-time communication systems should be tested under imperfect network conditions.

On Linux, tc/netem can simulate latency, jitter, and packet loss.

Example:

sudo tc qdisc add dev lo root netem delay 50ms 20ms loss 2%

Remove the configuration with:

sudo tc qdisc del dev lo root

This is useful for evaluating jitter-buffer behavior, retransmission, and media stability under degraded network conditions.

Key Engineering Areas

StreamLite is useful for exploring several real-time systems concepts:

* WebRTC connection establishment
* SDP negotiation
* ICE/STUN
* RTP and RTCP
* SRTP/SRTCP
* Audio/video packetization
* Jitter buffering
* Packet-loss recovery
* NACK and RTX
* Bandwidth estimation
* TWCC
* Simulcast
* Data channels
* Network programming
* Event-driven C++
* Low-latency media transport

License

See LICENSE.txt for the licensing terms and retain any notices required for code derived from third-party/open-source components.
