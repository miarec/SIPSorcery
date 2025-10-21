# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**SIPSorcery** is a comprehensive, fully managed C# .NET library for building real-time communications applications. It provides SIP, WebRTC, and VoIP capabilities without requiring native library wrappers or platform-specific code.

**Key Capabilities:**
- Full Session Initiation Protocol (SIP) implementation (RFC 3261)
- WebRTC peer-to-peer real-time communications
- Real-time Transport Protocol (RTP) for media streaming
- Interactive Connectivity Establishment (ICE) for NAT traversal
- STUN/TURN server support
- Audio codecs: G711 (PCMU/PCMA), G722, G729 (pure C#)
- DTLS-SRTP for secure media
- SCTP for WebRTC data channels
- RTSP client support

**Version:** 6.1.1-pre
**License:** BSD 3-Clause
**NuGet:** https://www.nuget.org/packages/SIPSorcery/

## Technology Stack

**Target Frameworks** (multi-targeting):
- .NET Standard 2.0 / 2.1
- .NET Core 3.1
- .NET 5.0 / 6.0

**Main Dependencies:**
- **Portable.BouncyCastle** (v1.9.0) - Cryptography for DTLS/SRTP
- **DnsClient** (v1.7.0) - DNS SRV record resolution
- **SIPSorcery.WebSocketSharp** (v0.0.1) - WebSocket support
- **SIPSorceryMedia.Abstractions** (v1.2.0) - Media endpoint interfaces

**Platform:** Cross-platform (Windows, Linux, macOS) - pure C# implementation

## Build and Development Commands

### Building

Build the entire solution (includes tests):
```bash
dotnet build src/SIPSorcery.sln
```

Build in Release mode:
```bash
dotnet build src/SIPSorcery.sln -c Release
```

Build only the core library:
```bash
dotnet build src/SIPSorcery.csproj -c Release
```

### Testing

Run unit tests:
```bash
dotnet test test/unit/SIPSorcery.UnitTests.csproj
```

Run integration tests:
```bash
dotnet test test/integration/SIPSorcery.IntegrationTests.csproj
```

Run all tests:
```bash
dotnet test src/SIPSorcery.sln
```

**Test Framework:** xUnit 2.4.1
**Test Data:** RFC 4475 SIP message torture tests, TLS certificates for DTLS testing

### Publishing

Create NuGet package (auto-generated on build):
```bash
dotnet pack src/SIPSorcery.csproj -c Release
```

## Architecture

### High-Level Component Layers

```
Application Layer (app/)
  ├── SIPUserAgent - High-level VoIP API
  ├── VoIPMediaSession - Media session management
  └── Codecs - G711, G722, G729 implementations

Core SIP Layer (core/)
  ├── SIPTransport - Multi-protocol transport manager
  ├── SIP Channels - UDP, TCP, TLS, WebSocket
  ├── SIPTransactionEngine - RFC 3261 transaction state machine
  └── DNS SRV Resolution

Network/Media Layer (net/)
  ├── RTP/RTCP - Real-time media transport
  ├── WebRTC - RTCPeerConnection, data channels
  ├── ICE/STUN - NAT traversal
  ├── SDP - Session Description Protocol
  ├── SCTP - Stream Control Transmission Protocol
  ├── DtlsSrtp - Secure media encryption
  └── RTSP - Streaming protocol client

System Utilities (sys/)
  └── Logging, crypto, network helpers
```

### Core Components

**SIPTransport** (`src/core/SIP/SIPTransport.cs`)
- Central manager for all SIP communication
- Manages multiple transport channels simultaneously (UDP, TCP, TLS, WebSocket)
- Handles DNS SRV resolution for service discovery
- Transaction engine for RFC 3261 compliance
- Async/await support with CancellationToken

**SIPUserAgent** (`src/app/SIPUserAgents/SIPUserAgent.cs`)
- High-level API for VoIP applications
- Manages both client and server roles
- Call lifecycle: setup, hold, transfer, hangup
- Media session integration
- Call transfer (REFER) support per RFC 3515

**VoIPMediaSession** (`src/app/Media/VoIPMediaSession.cs`)
- Audio/video stream management during calls
- Codec negotiation (G711, G722, G729)
- RTP session management
- Integrates with SIPSorceryMedia.Abstractions for device I/O

**RTCPeerConnection** (`src/net/WebRTC/RTCPeerConnection.cs`)
- WebRTC peer connection implementation
- ICE candidate management
- SDP offer/answer negotiation
- Data channel support via SCTP

**RTPChannel** (`src/net/RTP/RTPChannel.cs`)
- UDP socket management for media
- RTP packet send/receive
- RTCP reporting (RFC 3550)
- DTMF event handling

### Transport Channels

Each transport has a dedicated channel implementation:
- `SIPUDPChannel` - Connectionless UDP (most common)
- `SIPTCPChannel` - Stream-based TCP with message framing
- `SIPTLSChannel` - Encrypted TLS with certificate validation
- `SIPWebSocketChannel` - WebSocket for firewall/browser traversal

## Project Structure

```
src/
├── SIPSorcery.csproj          # Main library project
├── core/                       # Core SIP protocol
│   ├── SIP/                   # SIP messages, headers, URIs
│   │   └── Channels/          # UDP, TCP, TLS, WebSocket transports
│   ├── SIPTransactions/       # Transaction state machines
│   ├── SIPEvents/             # Event subscriptions (presence, dialog)
│   └── DNS/                   # DNS SRV resolution
├── app/                        # Application layer (high-level APIs)
│   ├── SIPUserAgents/         # User agent implementations
│   ├── Media/                 # Media sessions and codecs
│   │   ├── Codecs/            # G711, G722, G729
│   │   └── Sources/           # Test signal generators
│   └── SIPRequestAuthoriser/  # Authentication
├── net/                        # Network protocols and media
│   ├── RTP/                   # Real-time Transport Protocol
│   ├── RTCP/                  # RTP Control Protocol
│   ├── WebRTC/                # WebRTC implementation
│   ├── ICE/                   # Interactive Connectivity Establishment
│   ├── STUN/                  # NAT traversal
│   ├── SDP/                   # Session Description Protocol
│   ├── SCTP/                  # Stream Control Transmission Protocol
│   ├── DtlsSrtp/              # Secure media encryption
│   └── RTSP/                  # Real Time Streaming Protocol
├── sys/                        # System utilities
│   ├── Crypto/                # Cryptographic helpers
│   └── Net/                   # Network utilities
└── media/                      # Embedded test media files

test/
├── unit/                       # Unit tests (xUnit)
│   └── core/                  # RFC 4475 SIP torture tests
└── integration/                # Integration tests
```

## Important Implementation Details

### Media Endpoint Integration

SIPSorcery provides only basic audio codecs (G711, G722, G729). For production applications, additional platform-specific libraries are required:

- **Windows Audio I/O:** `SIPSorceryMedia.Windows`
- **Video Codecs:** `SIPSorceryMedia.Encoders` (VP8 via libvpx)
- **Cross-Platform:** `SIPSorceryMedia.FFmpeg`

The library uses `SIPSorceryMedia.Abstractions` interfaces (`IAudioSource`, `IAudioSink`, `IVideoSource`, `IVideoSink`) for media device abstraction.

### Transaction Timeouts

Recent enhancement (commit 601471b9) allows customization of T1/T2/T6 timeouts on individual transactions:
- T1: Initial retransmission interval (default 500ms)
- T2: Maximum retransmission interval (default 4s)
- T6: Transaction timeout (default 64*T1)

Can also disable retransmissions via `DisableRetransmitSending` flag.

### DNS SRV Resolution

When `SipRecPort` is 0, the library performs DNS SRV lookups to discover SIP servers:
- Query format: `_sip._tcp.domain.com` or `_sips._tcp.domain.com`
- Falls back to A/AAAA records if SRV not found
- Supports multiple targets with priority/weight

### Logging Configuration

Uses `Log.Logger` static property (compatible with Microsoft.Extensions.Logging):
- Integrates with Serilog, NLog, and other ILogger implementations
- SIP request/response logging at Information level (changed from Debug in commit 4e800f13)
- Tests use Serilog with xUnit sink

### Thread Pool Configuration

For optimal performance in production:
- SIPSorcery uses async/await extensively
- Separate message processing thread for queued SIP messages
- Ensure adequate thread pool size for high-volume scenarios

### Security (DTLS-SRTP)

For secure WebRTC/media:
- DTLS handshake establishes encryption keys
- SRTP encrypts RTP media streams
- SRTCP encrypts RTCP control messages
- BouncyCastle provides cryptographic primitives
- Custom certificate validation callbacks supported

## RFC Compliance

The library implements the following RFCs:
- **RFC 3261** - SIP (Session Initiation Protocol)
- **RFC 3550** - RTP (Real-time Transport Protocol)
- **RFC 8445** - ICE (Interactive Connectivity Establishment)
- **RFC 3515** - REFER (SIP call transfer)
- **RFC 3262** - PRACK (Provisional response acknowledgment)
- **RFC 4475** - SIP message parser torture tests

## Development Notes

- Pure C# implementation - no native dependencies in core library
- Cross-platform compatible via .NET Standard 2.0+
- Multi-targets 5 framework versions for broad compatibility
- Auto-generates NuGet package on build
- SourceLink support for debugging into library code
- Test projects target net461, netcoreapp3.1, net5.0, net6.0
- .NET 5.6.1 removed from TargetFrameworks (commit dd1fab25)
- BouncyCastle v2 upgrade reverted (commit 69713b06) - using v1.9.0
