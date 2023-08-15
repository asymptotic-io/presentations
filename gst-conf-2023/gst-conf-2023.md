---
title:
- Bridging WebRTC and SIP using GStreamer & SIPjs
author:
- Sanchayan Maity
theme:
- default
classoption:
- aspectratio=169
---

# Who?

- Open source consulting firm based out of Bangalore and Toronto.
- Building high-quality, low-level systems software.
- Most of our work revolves around audio/video using GStreamer and PipeWire.

# WebRTC

- `getUserMedia` - gets access to data streams such as the user's camera and phone
- `RTCPeerConnection` - enables audio or video calling with facilities for encryption and bandwidth management
    * `RTCDataChannel` - enables peer-to-peer communication of generic data
- Works in conjunction with other standards involved.
    * Session Description Protocol (SDP)
    * Interactive Connectivity Establishment (ICE)
    * Session Traversal Utilities for NAT (STUN)/Traversal Using Relays around NAT (TURN)
    * Real-time Transport Protocol (RTP)

# Session Initiation Protocol (SIP)

- RFC 3261
- Signalling Protocol
- Managing session (initiate, maintain, stop)
- Works in conjunction with other standards involved.
    * Session Description Protocol (SDP)
    * Interactive Connectivity Establishment (ICE)
    * Session Traversal Utilities for NAT (STUN)/Traversal Using Relays around NAT (TURN)
    * Real-time Transport Protocol (RTP)
- Resources on network (user agents, routers) are identified by a Uniform Resource Identifier (URI)
- Carried by transport layer protocols like TCP or UDP

# Why SIP?

- SIP is how phones connect to the Internet
- Dial in and dial out
- Conference room equipment interoperates using SIP

# SIP session setup

```{.d2}
shape: sequence_diagram
Alice's softphone -> "atlanta.com proxy": INVITE
"atlanta.com proxy" -> "biloxi.com proxy": INVITE
"atlanta.com proxy" -> Alice's softphone: 100 Trying
"biloxi.com proxy" -> Bob's SIP phone: INVITE
"biloxi.com proxy" -> "atlanta.com proxy": 100 Trying
Bob's SIP phone -> "biloxi.com proxy": 180 Ringing
"biloxi.com proxy" -> "atlanta.com proxy": 180 Ringing
"atlanta.com proxy" -> Alice's softphone: 180 Ringing
Bob's SIP phone -> "biloxi.com proxy": 200 OK
"biloxi.com proxy" -> "atlanta.com proxy": 200 OK
"atlanta.com proxy" -> Alice's softphone: 200 OK
Alice's softphone -> Bob's SIP phone: ACK
Alice's softphone <-> Bob's SIP phone: Media session
Bob's SIP phone -> Alice's softphone: BYE
Alice's softphone -> Bob's SIP phone: 200 OK
```

# SIPjs

- Library for talking to SIP servers from JavaScript
    * Uses WebSockets for SIP signalling (RFC 7118)
    * Uses `RTCPeerConnection` for media
    * Primarily used on browser

# SIPjs - User agent/Registration[^1]

```javascript
const transportOptions = {
  server: "wss://atlanta.com:8443"
};
const uri = UserAgent.makeURI("sip:alice@atlanta.com");
const userAgentOptions: UserAgentOptions = {
  authorizationPassword: 'secretPassword',
  authorizationUsername: 'authorizationUsername',
  transportOptions,
  uri
};
const userAgent = new UserAgent(userAgentOptions);
userAgent.start().then(() => {
  registerer.register();
});
```

[^1]: [SIPjs guides - user agent](https://sipjs.com/guides/user-agent/)

# SIPjs - make or receive a call[^2][^3]

```javascript
// userAgent defined elsewhere
userAgent.start().then(() => {
  const target = UserAgent.makeURI("sip:bob@biloxi.com");

  const inviter = new Inviter(userAgent, target);
  // make a call
  inviter.invite();
});
```

```javascript
// receive call
function onInvite(invitation) {
  invitation.accept();
}
```

[^2]: [SIPjs guides - make call](https://sipjs.com/guides/make-call/)
[^3]: [SIPjs guides - receive call](https://sipjs.com/guides/receive-call/)

# SIPjs - attach media[^4]

```javascript
// Assumes you have a media element on the DOM
const mediaElement = document.getElementById('mediaElement');

const remoteStream = new MediaStream(); // getUserMedia
function setupRemoteMedia(session: Session) {
  session.sessionDescriptionHandler.peerConnection.getReceivers().
    forEach((receiver) => {
      if (receiver.track) {
        remoteStream.addTrack(receiver.track);
      }
  });
  mediaElement.srcObject = remoteStream;
  mediaElement.play();
}
```
[^4]: [SIPjs guides - attach media](https://sipjs.com/guides/attach-media/)

# SIPjs - Session Description Handler Interface

```javascript
export interface SessionDescriptionHandler {
  getDescription(
    options?: SessionDescriptionHandlerOptions,
    modifiers?: Array<SessionDescriptionHandlerModifier>
  ): Promise<BodyAndContentType>;

  hasDescription(contentType: string): boolean;

  setDescription(
    sdp: string,
    options?: SessionDescriptionHandlerOptions,
    modifiers?: Array<SessionDescriptionHandlerModifier>
  ): Promise<void>;
}
```

# SIPjs - Session Description Handler

```javascript
export class SessionDescriptionHandler
  implements SessionDescriptionHandlerDefinition {
    protected _localMediaStream: MediaStream;
    protected _remoteMediaStream: MediaStream;
    protected _dataChannel: RTCDataChannel;
    protected _peerConnection: RTCPeerConnection;

    private iceGatheringCompletePromise: Promise<void>;
    private iceGatheringCompleteTimeoutId: number;
    private iceGatheringCompleteResolve: ResolveFunction;
    private iceGatheringCompleteReject: RejectFunction;
    private localMediaStreamConstraints: MediaStreamConstraints;
    private onDataChannel: ((dataChannel: RTCDataChannel) => void);
}
```

# `webrtcbin` & SIPjs

- `SIPjs` provides hooks to override `RTCPeerConnection` usage
    * GStreamer has a `webrtc` implementation with an API similar to `RTCPeerConnection`
    * Use this to call out to a GStreamer-based implementation leveraging `webrtcbin`
    * that's `SIPjs` (Node) -> `GStreamer` app (Rust)

# `webrtcbin` <-> `sipjs` - I

```{.d2}
direction: right

nodejs App: nodejs App {
  shape: rectangle
  style.border-radius: 8
  SIPjs: SIPjs {
    shape: rectangle
    style.border-radius: 8
    SessionDescriptionHandler: SessionDescriptionHandler {
      shape: rectangle
      style.border-radius: 8
      getSDP: getSDP {
        shape: rectangle
        style.border-radius: 8
      }
      getDescription: getDescription {
        shape: rectangle
        style.border-radius: 8
      }
      setDescription: setDescription {
        shape: rectangle
        style.border-radius: 8
      }
    }
  }
  GStreamer Process: GStreamer Process {
    shape: rectangle
    style.border-radius: 8
  }
}
Rust App: {
  shape: rectangle
  style.border-radius: 8
  GStreamer: GStreamer {
    shape: rectangle
    style.border-radius: 8
    webrtcbin: webrtcbin {
      shape: rectangle
      style.border-radius: 8
      create-offer-answer: create-offer-answer {
        shape: rectangle
        style.border-radius: 8
      }
      local-description: local-description {
        shape: rectangle
        style.border-radius: 8
      }
      set-remote-description: set-remote-description {
        shape: rectangle
        style.border-radius: 8
      }
    }
  }
}

nodejs App.GStreamer Process -> Rust App: IPC
nodejs App.SIPjs.SessionDescriptionHandler.getDescription -> Rust App.GStreamer.webrtcbin.local-description
nodejs App.SIPjs.SessionDescriptionHandler.setDescription -> Rust App.GStreamer.webrtcbin.set-remote-description
nodejs App.SIPjs.SessionDescriptionHandler.getSDP -> Rust App.GStreamer.webrtcbin.create-offer-answer
```

# `webrtcbin` <-> `SIPjs` - II

```javascript
class GstSessionDescriptionHandler implements SessionDescriptionHandler {
  gstProcess: ChildProcess;

  private async sendGst(msg: JsMessage): Promise<GstMessageSdp> {
    this.gstProcess.send(msg);
    return new Promise((resolve, reject) => {
      this.gstProcess.stdout.on('data', (message: string) => {
        let msg: GstMessage = JSON.parse(message);
        resolve(msg);
      });
    });
  }

  hasDescription(contentType: string): boolean {
    return contentType === 'application/sdp';
  }
```

# `webrtcbin` <-> `SIPjs` - III

```javascript
  async getDescription(): Promise<BodyAndContentType> {
    let msg =
      await this.sendGst({ code: JsMessageCode.GET_LOCAL_DESCRIPTION });
    return Promise.resolve({
      body: msg.sdpBody,
      contentType: 'application/sdp',
    });
  }

  async setDescription(
    sdp: string,
  ): Promise<void> {
    await this.sendGst({ code: JsMessageCode.SET_REMOTE_DESCRIPTION,
    sdpBody: sdp });
  }
```

# Daily back-end architecture[^5]

```{.d2}
direction: right

Daily Client: Daily Client {
  shape: rectangle
  style.multiple: true
  style.border-radius: 8
}
Selective Forwarding Unit: SFU {
  shape: rectangle
  style.border-radius: 8
}
GStreamer Backend: GStreamer Backend {
  shape: rectangle
  style.border-radius: 8
}
RTMP: RTMP {
  shape: rectangle
  style.border-radius: 8
}
AWS S3: S3 {
  shape: rectangle
  style.border-radius: 8
}
HLS: HLS {
  shape: rectangle
  style.border-radius: 8
}

Daily Client <-> Selective Forwarding Unit: Audio/Video
Daily Client <-> Selective Forwarding Unit: Audio/Video
Selective Forwarding Unit -> GStreamer Backend: Audio + Video
Selective Forwarding Unit -> GStreamer Backend: Audio + Video
GStreamer Backend -> RTMP: Audio + Video
GStreamer Backend -> AWS S3: Audio + Video
GStreamer Backend -> HLS: Audio + Video
```

[^5]: [GStreamer for your back-end services](https://www.daily.co/blog/gstreamer-for-your-backend-services-2/)

# `WebRTC -> SIP`

```{.d2}
direction: right

SFU: SFU {
  shape: rectangle
  style.border-radius: 8
}
rtpbin: rtpbin {
  shape: rectangle
  style.border-radius: 8
}
audiomixer: audiomixer {
  shape: rectangle
  style.border-radius: 8
}
opusenc: opusenc {
  style.border-radius: 8
    shape: rectangle
}
rtpopuspay: rtpopuspay {
  style.border-radius: 8
    shape: rectangle
}
webrtcbin: webrtcbin {
  shape: rectangle
  style.border-radius: 8
}

SFU -> rtpbin: Audio
SFU -> rtpbin: Audio
SFU -> rtpbin: Audio
rtpbin -> audiomixer: Audio
rtpbin -> audiomixer: Audio
rtpbin -> audiomixer: Audio
audiomixer -> opusenc
opusenc -> rtpopuspay
rtpopuspay -> webrtcbin: OPUS
```

# `SIP -> WebRTC`

```{.d2}
direction: left

SFU: SFU {
  shape: rectangle
  style.border-radius: 8
}
rtpbin: rtpbin {
  shape: rectangle
  style.border-radius: 8
}
udpsink: udpsink {
  shape: rectangle
  style.border-radius: 8
}
webrtcbin: webrtcbin {
  shape: rectangle
  style.border-radius: 8
}

SFU <- udpsink
udpsink <- rtpbin
rtpbin <- webrtcbin
```

# SIP Architecture

```{.d2}
direction: down

Daily SFU: SFU {
  shape: rectangle
  style.border-radius: 8
}
SIP Worker: SIP Worker {
  shape: rectangle
  style.border-radius: 8
  GStreamer Audio Mixer: GStreamer Audio Mixer {
    shape: rectangle
    style.border-radius: 8
  }
  Track Forwarding: Track Forwarding {
    shape: rectangle
    style.border-radius: 8
  }
}
SIP Provider: SIP Provider {
  shape: rectangle
  style.border-radius: 8
}
Softphone Client: Softphone Client {
  shape: rectangle
  style.multiple: true
  style.border-radius: 8
}
Phone: Phone {
  shape: rectangle
  style.multiple: true
  style.border-radius: 8
}

Daily SFU -> SIP Worker.GStreamer Audio Mixer
SIP Worker.GStreamer Audio Mixer -> SIP Provider: Daily Audio (Mixed)
SIP Worker.Track Forwarding <- SIP Provider: SIP Audio (Mixed, OPUS/G722/PCMA)
Daily SFU <- SIP Worker.Track Forwarding: OPUS
SIP Provider <-> Softphone Client
SIP Provider <-> Softphone Client
SIP Provider <-> Phone
SIP Provider <-> Phone
```

# Learnings

- Refactoring as bins
- NodeJS & Rust interop alternative approaches
  - Library style wrapper/FFI
  - Develop `RTCPeerConnection` like API
- Minor `webrtcbin` changes
  - Handling of non bundle media with `webrtcbin`
  - No default direction specified in SDP
- Media/Codec incompatibilities
- Latency

# What's next

- Upstream `webrtcbin` patches
- Open source `webrtcbin` and `SIPjs` demo
- Implement matrix/audio mixing ourselves
- Add video support


# Questions?

Thank You.
