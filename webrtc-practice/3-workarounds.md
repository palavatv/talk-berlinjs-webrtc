!SLIDE

# Workarounds

2 Examples


!SLIDE

## Problem

Firefox and Chrome don't want to talk with each other


!SLIDE small-code

## Workaround

Hardcode Crypto Headers

    @@@ coffeescript
    sdpHandler = (sd) =>
      if window.mozRTCPeerConnection
        sd.sdp += (
          'a=crypto:1 AES_CM_128_HMAC_SHA1_80 inline:' +
          'BAADBAADBAADBAADBAADBAADBAADBAADBAADBAAD\r\n'
        )
      # ...


!SLIDE

## Problem

When you mute one media stream in Chrome,

it will mute all of the other media streams!


!SLIDE

## Workaround

Manage all video "muted" states yourself!

When a stream gets (un)muted, always mute all streams

Then manually turn on all streams, where the mute state is "unmuted"
