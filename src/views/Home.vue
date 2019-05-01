<template>
  <div class="home">
    <video-player ref="player" source="//player.alicdn.com/video/aliyunmedia.mp4"></video-player>
    <ul>
      <li v-for="(msg,index) in msgs" :key="index">msg:{{msg}}</li>
    </ul>
  </div>
</template>

<script>
// @ is an alias to /src
import VideoPlayer from "@/components/VideoPlayer.vue";

export default {
  name: "home",
  components: {
    VideoPlayer
  },
  data() {
    return {
      msgs: []
    };
  },
  created() {
    // 添加对document 的 visibilitychange事件的监听
    document.addEventListener("visibilitychange", this.onVisibilityChange);
  },
  methods: {
    onVisibilityChange(e) {
      console.log("event", e);
      this.msgs.push(
        `${Date.now()}:visibilityState:${document.visibilityState}`
      );
      if (e.target.visibilityState === "visible") {
        // this.$refs.player.play();
      } else if (e.target.visibilityState === "hidden") {
        this.$refs.player.pause();
      }
    }
  },
  destroyed() {
    // 移除对document 的 visibilitychange事件的监听
    console.log("onVisibilityChange", this.onVisibilityChange);
    document.removeEventListener("visibilitychange", this.onVisibilityChange);
  }
};
</script>
