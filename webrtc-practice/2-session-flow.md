!SLIDE

# Session Flow
## Obtain Local Media
## Setup PeerConnection / Handle Streams
## Connect to Signaling Server / Join Room
## Exchange SDP
## Exchange Ice Candidates


!SLIDE

## Session Flow
# Obtain Local Media

!SLIDE small-code
    @@@ javascript
    var onLocalStreamReady = function(localStream) {
        ...
    };

    navigator.webkitGetUserMedia(
        {video: true, audio: false},
        function(stream) {
            console.log("Got localstream");
            onLocalStreamReady(stream);
        }
    );

!SLIDE

## Session Flow
# Setup PeerConnection / Handle Streams

!SLIDE small-code

    @@@ javascript
    var onLocalStreamReady = function(localStream) {
        var pc = new webkitRTCPeerConnection(
            {iceServers: [{url: "stun:93.186.193.18"}]},
            {optional: [{DtlsSrtpKeyAgreement: true}]}
        );
        pc.addStream(localStream);
        pc.onaddstream = function(event) {
            console.log("Add stream");
            $('#example2')[0].src =
                webkitURL.createObjectURL(event.stream);
        };

        // continues in next code example...

!SLIDE

## Session Flow
# Connect to Signaling Server / Join Room

!SLIDE smaller-code
    @@@ javascript
        // ... continued from last code example

        var peerId;

        var server = new WebSocket('wss:palava.tv:4233');
        server.onopen = function() {
            server.onmessage = function(msg) {
              ... // react on joined_room, answer & ice_candidate
            };

            ... // send own ice_candidates

            console.log("Send message: 'join_room'");
            server.send(JSON.stringify({
                event: 'join_room',
                room_id: 'berlinjs'
            }));
        };
    };

!SLIDE

## Session Flow
# Exchange SDP

!SLIDE smaller-code
    @@@ javascript
    server.onmessage = function(msg) {
          msg = JSON.parse(msg.data);
          console.log("Got message: " + msg.event);

          if(msg.event === 'joined_room') {
              peerId = msg.peer_ids[0];
              pc.createOffer(function(sdp) {
                  console.log("Created offer for peer" + peerId);
                  pc.setLocalDescription(sdp);
                  server.send(JSON.stringify({
                      event: 'send_to_peer',
                      peer_id: peerId,
                      data: {
                          event: 'offer',
                          sdp: sdp
                      }
                  }));
              });
          ...
          }
    };

!SLIDE small-code
    @@@javascript
    server.onmessage = function(msg) {
          msg = JSON.parse(msg.data);
          console.log("Got message: " + msg.event);
          ...
          if(msg.event === 'answer') {
              pc.setRemoteDescription(
                  new RTCSessionDescription(msg.sdp)
              );
          }
          ...
    };

!SLIDE

## Session Flow
# Exchange Ice Candidates

!SLIDE smaller-code
    @@@javascript
    server.onmessage = function(msg) {
        msg = JSON.parse(msg.data);
        console.log("Got message: " + msg.event);
        ...
        if(msg.event === 'ice_candidate') {
            var candidate = new RTCIceCandidate({candidate: msg.candidate...});
            pc.addIceCandidate(candidate);
        }
    };

    pc.onicecandidate = function(event) {
        if(event.candidate) {
            console.log("Send ice candidate");
            server.send(JSON.stringify({
                event: 'send_to_peer',
                peer_id: peerId,
                data: {event:'ice_candidate', candidate:event.candidate.candidate...}
            }));
        }
    };


!SLIDE example2-slide small

# PeerConnection Demo

<div style="text-align: center; display: none" id="warning">Nobody is in <a href="https://palava.tv/berlinjs">https://palava.tv/berlinjs</a>, so this demo won't work!</div>
<video id="example2" autoplay="autoplay" />

<script>
// This example code joins the berlinjs room on the palava rtc server
// It requires the other peer to already be present in the room
// Only works in Chrome
$(".example2-slide").bind("showoff:show", function() {
  var onLocalStreamReady = function(localStream) {
      var pc = new webkitRTCPeerConnection({iceServers: [{url: "stun:93.186.193.18"}]}, {"optional": [{"DtlsSrtpKeyAgreement": true}]});
      var server = new WebSocket('wss:palava.tv:4233');

      pc.addStream(localStream);
      pc.onaddstream = function(event) {
          console.log("Add stream");
          $('#example2')[0].src = webkitURL.createObjectURL(event.stream);
      };
      var peerId;

      server.onopen = function() {
          server.onmessage = function(msg) {
              msg = JSON.parse(msg.data);
              console.log("Got message: " + msg.event);

              if(msg.event === 'joined_room') {
                  if(msg.peer_ids.length === 0) {
                    $('#warning').show();
                    server.close();
                    return null;
                  }
                  peerId = msg.peer_ids[0];
                  pc.createOffer(function(sdp) {
                      console.log("Created offer for peer" + peerId);
                      pc.setLocalDescription(sdp);
                      server.send(JSON.stringify({
                          event: 'send_to_peer',
                          peer_id: msg.peer_ids[0],
                          data: {
                              event: 'offer',
                              sdp: sdp
                          }
                      }));
                  });
              }

              if(msg.event === 'answer') {
                  pc.setRemoteDescription(new RTCSessionDescription(msg.sdp));
              }

              if(msg.event === 'ice_candidate') {
                  var candidate = new RTCIceCandidate({candidate: msg.candidate, sdpMLineIndex: msg.sdpmlineindex, sdpMid: msg.sdpmid});
                  pc.addIceCandidate(candidate);
              }
          };

          pc.onicecandidate = function(event) {
              if(event.candidate) {
                  console.log("Send ice candidate");
                  server.send(JSON.stringify({
                      event: 'send_to_peer',
                      peer_id: peerId,
                      data: {
                          event: 'ice_candidate',
                          label: event.candidate.label,
                          sdpmlineindex: event.candidate.sdpMLineIndex,
                          sdpmid: event.candidate.sdpMid,
                          candidate: event.candidate.candidate
                      }
                  }));
              }
          };

          console.log("Send message: 'join_room'");
          server.send(JSON.stringify({
              event: 'join_room',
              room_id: 'berlinjs'
          }));
      };
  };

  navigator.webkitGetUserMedia(
      {video: true, audio: false},
      function(stream) {
          console.log("Got localstream");
          onLocalStreamReady(stream);
      }
  );
});
</script>
