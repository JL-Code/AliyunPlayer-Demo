<template>
  <div :id="playerId"></div>
</template>

<script>
export default {
  name: "Player",
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

    // 视频封面
    cover: String,
    // 阿里云Web播放器SDK
    aliplayerSdkPath: {
      type: String,
      default: "https://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-h5-min.js"
    },
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
    /**
     * 播放器是否自动播放，在移动端autoplay属性会失效。Safari11不会自动开启自动播放
     */
    autoplay: {
      type: Boolean,
      default: false
    }
    // skinLayout: [Boolean, Array]
  },
  data() {
    return {
      playerId:
        "aliplayer_" +
        Math.random()
          .toString(36)
          .substr(2),
      scriptTagStatus: 0,
      isReload: false,
      initialized: false,
      playing: false,
      isAudio: undefined,
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
    if (window.Aliplayer !== undefined) {
      // 如果全局对象存在，说明编辑器代码已经初始化完成，直接加载编辑器
      this.scriptTagStatus = 2;
      this.initPlayer();
    } else {
      // 如果全局对象不存在，说明编辑器代码还没有加载完成，需要加载编辑器代码
      this.insertScriptTag();
    }
  },
  created() {
    console.log("created");
  },
  methods: {
    initPlayer() {
      console.log("start initPlayer");
      const vm = this;
      // scriptTagStatus 为 2 的时候，说明两个必需引入的 js 文件都已经被引入，且加载完成
      if (
        vm.scriptTagStatus === 2 &&
        (vm.$player === null || vm.reloadPlayer)
      ) {
      }
      vm.$player && vm.$player.dispose();
      // Vue 异步执行 DOM 更新，这样一来代码执行到这里的时候可能 template 里面的 script 标签还没真正创建
      // 所以，我们只能在 nextTick 里面初始化 Aliplayer
      vm.$nextTick(() => {
        if (typeof Aliplayer === "undefined") {
          console.warn("missing Aliplayer");
          return;
        }
        let defaults = {
          // player 容器ID
          id: vm.playerId,
          // 播放器宽度
          width: "100%",
          // 播放器是否自动播放，在移动端autoplay属性会失效。
          autoplay: false,
          preload: false,
          controlBarVisibility: "always",
          skinLayout: [
            {
              name: "bigPlayButton",
              align: "blabs",
              x: 30,
              y: 80
            },
            {
              name: "H5Loading",
              align: "cc"
            },
            {
              name: "errorDisplay",
              align: "tlabs",
              x: 0,
              y: 0
            },
            {
              name: "infoDisplay"
            },
            {
              name: "tooltip",
              align: "blabs",
              x: 0,
              y: 56
            },
            {
              name: "thumbnail"
            },
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
                  name: "fullScreenButton",
                  align: "tr",
                  x: 10,
                  y: 12
                },
                {
                  name: "volume",
                  align: "tr",
                  x: 5,
                  y: 10
                }
              ]
            }
          ]
        };
        const opts = Object.assign({}, defaults, {
          source: vm.source,
          width: vm.width,
          height: vm.height,
          autoplay: vm.autoplay,
          language: "zh-cn"
        });
        // 创建一个player实例
        console.log("Aliplayer opts: ", opts);
        new Aliplayer(opts, function(player) {
          vm.$player = player;
          console.log("player", player);
          console.log("player _isAudio", player._isAudio);
          vm.isAudio = player._isAudio;
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
      });
    },
    insertScriptTag() {
      const _this = this;
      let playerScriptTag = document.getElementById("playerScriptTag");
      // 如果这个tag不存在，则生成相关代码tag以加载代码
      if (playerScriptTag === null) {
        playerScriptTag = document.createElement("script");
        playerScriptTag.type = "text/javascript";
        playerScriptTag.src = this.aliplayerSdkPath;
        playerScriptTag.id = "playerScriptTag";
        let s = document.getElementsByTagName("head")[0];
        s.appendChild(playerScriptTag);
      }
      if (playerScriptTag.loaded) {
        _this.scriptTagStatus++;
      } else {
        playerScriptTag.addEventListener("load", () => {
          _this.scriptTagStatus++;
          playerScriptTag.loaded = true;
          _this.initPlayer();
        });
      }
      _this.initPlayer();
    },
    reloadPlayer: function() {
      this.isReload = true;
      this.initPlayer();
      this.isReload = false;
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
    },
    // 获取当前的播放时刻，返回的单位为秒。
    getCurrentTime() {
      this.$player.getCurrentTime();
    },
    // 跳转到某个时刻进行播放，time的单位为秒。
    seek(time) {
      this.$player.seek(time);
    },
    // 获取播放器状态，包含的值：
    // ‘init’
    // ‘ready’
    // ‘loading’
    // ‘play’
    // ‘pause’
    // ‘playing’
    // ‘waiting’
    // ‘error’
    // ‘ended’
    getStatus() {
      return this.$player.getStatus();
    }
  },
  beforeDestroy() {
    // 销毁player
    this.$player.dispose();
    this.$player = null;
  }
};
</script>

<style>
@import url(//g.alicdn.com/de/prismplayer/2.8.1/skins/default/aliplayer-min.css);
</style>

