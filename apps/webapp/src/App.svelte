<script lang="ts">
  import { Peer } from "peerjs";
  import { onMount } from "svelte";

  let peer: Peer;
  let myPeerId: string;
  let peerId: string;
  let myStream: MediaStream;
  let myVideo: HTMLVideoElement;
  let remotePeerVideo: HTMLVideoElement;

  async function getMyVideoStream() {
    return navigator.mediaDevices
      .getUserMedia({
        audio: false,
        video: {
          facingMode: "facingMode",
        },
      })
      .then((stream) => {
        myStream = stream;
        myVideo.srcObject = stream;
      })
      .catch((err) => {
        console.log("Failed to get local stream", err);
      });
  }

  function callPeer(remotePeerId: string) {
    const call = peer.call(remotePeerId, myStream);
    call.on("stream", function (remoteStream) {
      console.log("Got remote stream from the called person:\n", remoteStream);
      remotePeerVideo.srcObject = remoteStream;
    });
  }

  onMount(() => {
    peer = new Peer();
    console.log(peer);
    peer.on("open", function (id) {
      myPeerId = id;
      console.log("My peer ID is: " + myPeerId);
    });

    // Receiving a call
    peer.on("call", function (call) {
      getMyVideoStream().then(() => {
        call.answer(myStream); // Answer the call with an A/V stream.
        call.on("stream", function (remoteStream) {
          console.log("Got remote stream from caller:\n", remoteStream);
          remotePeerVideo.srcObject = remoteStream;
        });
      });
    });

    getMyVideoStream();
  });
</script>

<main>
  <form on:submit|preventDefault={() => callPeer(peerId)}>
    <label for="peerId">
      <p>Enter peer id to call</p>
      <input type="text" bind:value={peerId} />
    </label>
    <button type="submit">Submit</button>
  </form>
  <div id="videos">
    <video bind:this={myVideo} muted autoplay playsinline />
    <video bind:this={remotePeerVideo} muted autoplay playsinline />
  </div>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  #videos {
    display: grid;
    grid-template-columns: 1fr 1fr;
    width: 100%;
  }
</style>
