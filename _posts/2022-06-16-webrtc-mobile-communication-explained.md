---
layout: post
title:  "WebRTC & Mobile Integration: Explained"
date:   2022-06-16
excerpt:
tag:
- WebRTC
- Mobile Apps
- Mobile webRTC integration 
- Distributed Systems
comments: true
---
With WebRTC, you can add real-time communication capabilities to your application that works on top of an open standard. It supports video, voice, and generic data to be sent between peers, allowing developers to build powerful voice- and video-communication solutions.

Sequence Diagram (Simple)

![image](https://user-images.githubusercontent.com/25557899/174005462-2b8c335e-23de-4e6d-84ed-b30501777a96.png)


WebRTC requires two data payloads to be transferred between parties, it's called SDP (sesssion description protocol). One is called offer  and the second is answer. 

You can either create an offer and send it to other party or wait for an offer to be delivered to you. Usually SDP handshakes are done by special signaling server, but in this case we are not using any, so you'll need to pass SDPs manually by e.g. e-mail. 

If it's running IPv4, it's very unlikely that both parties will have a public IP address or it will be on the same network. SDP requires that you'll need to pass external IP address there, this is done automatically by process called ICE gathering. It uses two types of external servers - STUN and TURN. We are using only STUN in our App, but it should work with TURN as well (and even better) .(Not anymore, they have their own media server)  It can even punch through some NAT mechanisms. 

Sequence Diagram (p2p)

![image](https://user-images.githubusercontent.com/25557899/174005513-94615de6-f196-4e3b-908f-09da769dd38a.png)


**Peer Connections**

Peer connections is the part of the WebRTC specifications that deals with connecting two applications on different computers to communicate using a peer-to-peer protocol. The communication between peers can be video, audio or arbitrary binary data (for clients supporting the RTCDataChannel API). In order to discover how two peers can connect, both clients need to provide an ICE Server configuration. This is either a STUN or a TURN-server, and their role is to provide ICE candidates to each client which is then transferred to the remote peer. This transferring of ICE candidates is commonly called signaling.

Each peer connection is handled by a RTCPeerConnection object. The constructor for this class takes a single RTCConfiguration object as its parameter. This object defines how the peer connection is set up and should contain information about the ICE servers to use.

Once the RTCPeerConnection is created we need to create an SDP offer or answer, depending on if we are the calling peer or receiving peer. Once the SDP offer or answer is created, it must be sent to the remote peer through a different channel. Passing SDP objects to remote peers is called signaling and is not covered by the WebRTC specification.

To initiate the peer connection setup from the calling side, we create a RTCPeerConnection object and then call createOffer() to create a RTCSessionDescription object. This session description is set as the local description using setLocalDescription() and is then sent over our signaling channel to the receiving side. We also set up a listener to our signaling channel for when an answer to our offered session description is received from the receiving side.

On the receiving side, we wait for an incoming offer before we create our RTCPeerConnection instance. Once that is done we set the received offer using setRemoteDescription(). Next, we call createAnswer() to create an answer to the received offer. This answer is set as the local description using setLocalDescription() and then sent to the calling side over our signaling server.

Once the two peers have set both the local and remote session descriptions they know the capabilities of the remote peer. This doesn't mean that the connection between the peers is ready. For this to work we need to collect the ICE candidates at each peer and transfer (over the signaling channel) to the other peer.

Once a RTCPeerConnection object is created, the underlying framework uses the provided ICE servers to gather candidates for connectivity establishment (ICE candidates). The event icegatheringstatechange on RTCPeerConnection signals in what state the ICE gathering is (new, gathering or complete).

Once ICE candidates are being received, we should expect the state for our peer connection will eventually change to a connected state. To detect this, we add a listener to our RTCPeerConnection where we listen for connectionstatechange events.

**Use of Kurento Media Server **

Sequence Diagram(Kurento)

![image](https://user-images.githubusercontent.com/25557899/174005604-c3cec95b-26fb-4d7f-aeee-b6baeaea87d4.png)


1. User A is registered in the server with his name

2. User B is registered in the server with her name

3. User A wants to call to User B

4. User B accepts the incoming call

5. The communication is established and media is flowing between User A and User B

6. One of the users finishes the video communication

As you can see in the diagram, SDP and ICE candidates need to be interchanged between client and server to establish the WebRTC connection between the Kurento client and server. Specifically, the SDP negotiation connects the WebRtcPeer in the browser with the WebRtcEndpoint in the server.

**Connection Observer methods (callbacks)**


- **addStream**: if this method was called, the Javascript code has added a MediaStream to the peerconnection. You can see the id of the stream as well as the audio and video tracks. onAddStream shows a remote stream being added, including the audio and video track ids, it is called between the setRemoteDescription call and the setRemoteDescriptionOnSuccess callback

- **createOffer** shows any calls to this API including the options such as offerToReceiveAudio, offerToReceiveVideo or iceRestart. createOfferOnSuccess shows the results of the createOffer call, including the type (which should be ‘offer’ obviously) and the SDP resulting from it. createOfferOnFailure could also be called indicating an error but that is quite rare

- **createAnswer** and createAnswerOnSuccess and createAnswerOnFailure are similar but with no additional options

- **setLocalDescription** shows you the type and SDP used in the setLocalDescription call. If you do any SDP munging between createOffer and setLocaldescription you will see this here. This results in either a setLocalDescriptionOnSuccess or setLocalDescriptionOnFailure callback which shows any errors. The same applies to the setRemoteDescription and its callbacks, setRemoteDescriptionOnSuccess and setRemoteDescriptionOnFailure

- **onRenegotiationNeeded** is the old chrome-internal name for the onnegotiationneeded event. If your app uses this you might want to look for it

- **onSignalingStateChange** shows the changes in the signaling state as a result of calls to setLocalDescription and setRemoteDescription. See the wonderful diagram in the specification for the gory details. At the end of the day, you will want to be in the stable state most of the time

- **iceGatheringStateChange** is the little brother of the ice connection state. It will show you the state of the ice gatherer. It will change to gathering after setLocalDescription if there are ICE candidates to gather

- **onnicecandidate** events show all candidates gathered, with information for which m-line and MID. Likewise, the addIceCandidate method shows that information from the other side. Typically you should see both event types. See below for a more detailed discussion of these events

- **oniceconnectionstate** is one of the most important event handlers. It tells you whether a peer-to-peer connection succeeded or not. From here, you can start searching for the active candidate as explained.
Timeline of the aboves:

**caller:**

- (addStream if the local side wants to send media)

- createOffer

- createOfferOnSuccess

- setLocalDescription

- setLocalDescriptionOnSuccess

- setRemoteDescription

- (onaddstream if the remote end signalled media streams in the SDP)

- setRemoteDescriptionOnSuccess

**answerer:**

- setRemoteDescription

- (onaddstream if the remote end signalled media streams in the SDP)

- createAnswer

- createAnswerOnSuccess

- setLocalDescription

- setLocalDescriptionOnSuccess

**Steps to create Video Tracks Captures from p2p connection**

1. Create and initialize PeerConnectionFactory 

2. Create a VideoCapturer instance which uses the camera of the device 

3. Create a VideoSource from the Capturer Create a VideoTrack from the source 

4. Create a video renderer using a SurfaceViewRenderer view and add it to the VideoTrack instance 

5. Initialize Peer connections 

6. Start streaming Video 

**Sources: ( study material for webRTC )**

https://www.w3.org/TR/webrtc/    

https://temasys.io/webrtc-ice-sorcery/

https://andrewjprokop.wordpress.com/2014/07/21/understanding-webrtc-media-connections-ice-stun-and-turn/

https://www.html5rocks.com/en/tutorials/webrtc/infrastructure/

https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling

https://hackernoon.com/swiftywebrtc-789936b0e39b

https://bloggeek.me/how-webrtc-works/ 

https://temasys.io/webrtc-ice-sorcery/ 

https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling 

https://webrtc.ventures/2018/01/webrtc-on-android-tutorial-how-to-build-a-chat-roulette-clone-using-kotlin-and-typescript/ 

http://www.hehuo168.com/misc/WebRTC%20Getting%20Started.pdf 

https://media.prod.mdn.mozit.cloud/attachments/2018/08/22/16137/06b5fa4f9b25f5613dae3ce17b0185c5/WebRTC_-_Signaling_Diagram.svg 

https://amryousef.me/android-webrtc (ΚΑΛΟ) 

https://github.com/ant-media/Ant-Media-Server/wiki/Step-by-Step-Guide-to-Build-WebRTC-Native-Android-App 

https://libs.garden/kotlin/search?q=webrtc 

https://testrtc.com/webrtc-api-trace/

https://medium.com/@SeoJaeDuk/
