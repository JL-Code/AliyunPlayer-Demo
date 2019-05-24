<template>
  <div class="home">
    <!-- 音频播放 -->
    <video-player
      ref="player"
      source="https://s128.xiami.net/214/214/905/10915_2865875_l.mp3?ccode=xiami_web_web&expire=86400&duration=278&psid=db47a1e212e0bc4acbf36943d92364a5&ups_client_netip=14.111.60.252&ups_ts=1556728390&ups_userid=0&utid=c3bYFAbxIioCAQ5vPNBBsZQW&vid=10915&fn=10915_2865875_l.mp3&vkey=B1c56ed76c2b07fb2f18132bce2f7e3ae"
      height="50px"
    ></video-player>
    <ul>
      <li v-for="(msg,index) in msgs" :key="index">msg:{{msg}}</li>
    </ul>
    <!-- 视频播放 -->
    <div>
      <video-player ref="player2" source="//player.alicdn.com/video/aliyunmedia.mp4"></video-player>
    </div>
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
      msgs: [],
      skinLayout: [
        {
          name: "controlBar",
          align: "blabs",
          x: 0,
          y: 0,
          children: [
            {
              name: "progress",
              align: "blabs",
              x: 0,
              y: 44
            },
            {
              name: "playButton",
              align: "tl",
              x: 15,
              y: 12
            },
            {
              name: "timeDisplay",
              align: "tl",
              x: 10,
              y: 7
            },
            {
              name: "volume",
              align: "tr",
              x: 10,
              y: 10
            }
          ]
        }
      ]
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
