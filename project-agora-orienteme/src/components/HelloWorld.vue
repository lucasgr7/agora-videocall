<template>
  <div>
    <button @click="joinCall">Join Call</button>
    <button @click="leaveCall">Leave Call</button>
    <div ref="localPlayerContainer"></div>
    <div ref="remotePlayerContainer"></div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import AgoraRTC from "agora-rtc-sdk-ng";

export default {
  setup() {

    let options = {
      appId: "28e2969aa1ab4f11af8e66e135b1941b",
      channel: "orienteme",
      token: "007eJxTYNC+UHqlW+MT53nBbTyPNy83asj2fZVSnGDTmHu4PHha7BoFBiOLVCNLM8vERMPEJJM0Q8PENItUM7NUQ2PTJENLE8OkEon5KQ2BjAyf+3YzMjJAIIjPyZBflJmaV5Kam8rAAAAEiyH0",
      uid: 0,
    };

    const localPlayerContainer = ref(null);
    const remotePlayerContainer = ref(null);
    const agoraEngine = ref(null);
    
    const channelParameters = ref({
      localAudioTrack: null,
      localVideoTrack: null,
      remoteAudioTrack: null,
      remoteVideoTrack: null,
      remoteUid: null,
    });

    const joinCall = async () => {
      agoraEngine.value = AgoraRTC.createClient({ mode: "rtc", codec: "vp9" });

      agoraEngine.value.on("user-published", async (user, mediaType) => {
        await agoraEngine.value.subscribe(user, mediaType);

        if (mediaType === "video") {
          channelParameters.value.remoteVideoTrack = user.videoTrack;
          channelParameters.value.remoteAudioTrack = user.audioTrack;
          channelParameters.value.remoteUid = user.uid.toString();
          remotePlayerContainer.value.id = user.uid.toString();
          remotePlayerContainer.value.textContent = "Remote user " + user.uid.toString();
          channelParameters.value.remoteVideoTrack.play(remotePlayerContainer.value);
          channelParameters.value.localVideoTrack.play(localPlayerContainer.value);
        }

        if (mediaType === "audio") {
          channelParameters.value.remoteAudioTrack = user.audioTrack;
          channelParameters.value.remoteAudioTrack.play();
        }
      });

      agoraEngine.value.on("user-unpublished", (user) => {
        console.log(user.uid + " has left the channel");
      });

      await agoraEngine.value.join(
        options.appId,
        options.channel,
        options.token,
        options.uid
      );

      channelParameters.value.localAudioTrack = await AgoraRTC.createMicrophoneAudioTrack();
      channelParameters.value.localVideoTrack = await AgoraRTC.createCameraVideoTrack();
      channelParameters.value.localVideoTrack.play(localPlayerContainer.value);

      await agoraEngine.value.publish([
        channelParameters.value.localAudioTrack,
        channelParameters.value.localVideoTrack,
      ]);

      console.log("Publish success!");
    };

    const leaveCall = async () => {
      channelParameters.value.localAudioTrack.close();
      channelParameters.value.localVideoTrack.close();
      removeVideoDiv(remotePlayerContainer.value.id);
      removeVideoDiv(localPlayerContainer.value.id);
      await agoraEngine.value.leave();
      console.log("You left the channel");
      window.location.reload();
    };

    const removeVideoDiv = (elementId) => {
      console.log("Removing " + elementId + "Div");
      let Div = document.getElementById(elementId);
      if (Div) {
        Div.remove();
      }
    };

    onMounted(() => {
      localPlayerContainer.value = document.createElement("div");
      localPlayerContainer.value.id = options.uid;
      localPlayerContainer.value.textContent = "Local user " + options.uid;
      localPlayerContainer.value.style.width = "500px";
      localPlayerContainer.value.style.height = "480px";
      localPlayerContainer.value.style.padding = "15px 5px 5px 5px";
      document.body.append(localPlayerContainer.value);

      remotePlayerContainer.value = document.createElement("div");
      remotePlayerContainer.value.style.width = "500px";
      remotePlayerContainer.value.style.height = "480px";
      remotePlayerContainer.value.style.padding = "15px 5px 5px 5px";
      document.body.append(remotePlayerContainer.value);
    });

    return {
      joinCall,
      leaveCall,
      localPlayerContainer,
      remotePlayerContainer,
    };
  },
};
</script>
