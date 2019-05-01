<template>
  <div ref="J_AliyunPlayer_Element" id="J_AliyunPlayer_Element">视频播放器</div>
</template>

<script>
export default {
  name: "VideoPlayer",
  props: {
    /**
     * @description 视频播放方式
     * @see https://help.aliyun.com/document_detail/62941.html?spm=a2c4g.11186623.2.24.16831c4cnOui7G
     * 1. 支持播放地址播放,此播放优先级最高
     * 2. 点播用户推荐
     * ```json
     * {
     *    vid : '1e067a2831b641db90d570b6480fbc40', // 视频ID
          playauth : 'ddd', // 播放授权
          cover: 'http://liveroom-img.oss-cn-qingdao.aliyuncs.com/logo.png',  //视频封面
     * }
     * ```
     * 3. 仅MPS用户使用
     */
    playmode: {
      type: String,
      default: "default"
    }, // default : 播放地址播放；
    // 播放地址
    source: String,
    // 播放器宽度
    width: {
      type: String,
      default: "100%"
    },
    // 播放器高度
    height: {
      type: String,
      default: "300px"
    },
    // 播放器是否自动播放，在移动端autoplay属性会失效。
    autoplay: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      initialized: false,
      playing: false,
      $player: null
    };
  },
  computed: {
    // 播放模式
    // playerOptions() {
    //   switch (this.playmode) {
    //     case "default":
    //       break;
    //     default:
    //       break;
    //   }
    // }
  },
  mounted() {
    console.log("mounted");
  },
  created() {
    console.log("created");
    this.$nextTick(() => {
      this.initPlayer();
    });
  },
  methods: {
    initPlayer() {
      console.log("start initPlayer");
      var vm = this;
      if (typeof Aliplayer === "undefined") {
        console.warn("missing Aliplayer");
        return;
      }
      var defaults = {
        // player 容器ID
        id: "J_AliyunPlayer_Element",
        // 播放器宽度
        width: "100%",
        // 播放器是否自动播放，在移动端autoplay属性会失效。
        autoplay: false
      };
      var opts = Object.assign({}, defaults, {
        source: vm.source,
        width: vm.width,
        height: vm.height,
        autoplay: vm.autoplay
      });
      // 创建一个player实例
      new Aliplayer(opts, function(player) {
        vm.$player = player;
        console.log("player", player);
        console.log("播放器初始完成");
        vm.initialized = true;
        vm.$emit("initialized");
        // 监听player事件
        vm.$player.on("ready", function(e) {
          vm.$emit("ready", e);
        });
        vm.$player.on("play", function(e) {
          vm.$emit("play", e);
          vm.playing = true;
        });
        vm.$player.on("pause", function(e) {
          vm.$emit("pause", e);
          vm.playing = false;
        });
        vm.$player.on("ended", function(e) {
          vm.$emit("ended", e);
          vm.playing = false;
        });
        vm.$player.on("playing", function(e) {
          vm.$emit("playing", e);
        });
        vm.$player.on("error", function(e) {
          vm.$emit("error", e);
        });
      });
    },
    /**
     * @description 播放视频
     */
    play() {
      this.$player.play();
    },
    /**
     * @description 暂停播放
     */
    pause() {
      this.$player.pause();
    },
    /**
     * @description 重播视频
     */
    replay() {
      this.$player.replay();
    }
  },
  beforeDestroy() {
    // 移除事件监听
    // 移除额外创建的dom元素
    this.$player = null;
  }
};
</script>

