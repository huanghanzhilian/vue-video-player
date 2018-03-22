<template>
  <div class="video-player">
    <video class="video-js">
      <!-- <track kind="subtitles" src="http://test.huanghanlian.com/example-captions.vtt" srclang="en" label="Chinese"> -->
    </video>
  </div>
</template>
<script>
window.videojs = require('video.js')
window.libjass = require('libjass')
window.URLSafeBase64 = require('urlsafe-base64')
//import URLSafeBase64 from 'urlsafe-base64'
//require('videojs5-hlsjs-source-handler')
videojs = videojs.default || videojs
export default {
  name: 'video-player',
  data() {
    return {
      'img': require("./img/logo-w.png"),
      isSubtitles: true,
      player: null,
      textTracksmy: null
    }
  },
  props: {
    options: {
      type: Object,
      required: true
    },
    start: {
      type: Number,
      default: function() {
        return 0
      }
    },
    playsinline: {
      type: Boolean,
      default: function() {
        return false
      }
    },
    customEventName: {
      type: String,
      default: function() {
        return 'statechanged'
      }
    },
    subtitle:{
      type: String,
      default: function() {
        return ''
      }
    }
  },
  mounted: function() {
    //console.log(this.player)
    /*if (!this.player) {
      this.initialize()
    }*/
  },
  beforeDestroy: function() {
    if (this.player) {
      this.dispose()
    }
  },
  methods: {
    initialize: function() {

      // init
      var self = this
      this.player = null
      // videojs options  默认参数
      var videoOptions = Object.assign({
        autoplay: false, // 自动播放
        controls: true, // 是否显示控制栏
        // preload: 'auto',
        // fluid: false,
        // muted: false,
        width: '100%',
        height: '100%',
        language: 'en',
        // "html5": {
        //   hls: {
        //     withCredentials: true
        //   }
        // },
        // html5: {
        //     hlsjsConfig: {
        //         debug: true
        //     }
        // },
        controlBar: {
          // remainingTimeDisplay: false,
          // subtitlesButton: true, //字幕
          // captionsButton: true, //字幕带设置
          // chaptersButton: true,
          // liveDisplay: false,
          // playbackRateMenuButton: false,
          // playToggle: {},
          // progressControl: {},
          // fullscreenToggle: {},
          // subtitlesButton: {},
          // volumeMenuButton: {
          //   inline: false,
          //   vertical: true
          // }
          'playToggle': true, //播放暂停按钮
          'volumeMenuButton': false, //音量控制
          'currentTimeDisplay': true, //当前播放时间
          'timeDivider': true, //  '/'
          'durationDisplay': true, //总时间
          'remainingTimeDisplay': false, //当前播放时间
          'progressControl': true, //点播流时，播放进度条，seek控制
          'chaptersButton': false,
          'liveDisplay': false, //直播流时，显示LIVE
          'fullscreenToggle': true, //全屏控制
          'descriptionsButton': false,
          'subsCapsButton': false,
          'audioMenuButton':false,//耳机按钮
          'playbackRate':false,
        },
        // techOrder: ['html5'],
        plugins: {
          /*ass: {
            src: 'http://test.huanghanlian.com/%5Bzmk.tw%5DBatman.v.Superman.Dawn.of.Justice.2016.Ultimate.Edition.WEB-DL.Chs.&amp;.Eng.ass',
            label: "engsub",
            videoWidth: 640,
            videoHeight: 360,
          }*/
        }
      }, this.options)



      // check sources
      /*
      if (!videoOptions.sources || !videoOptions.sources.length) {
        console.warn('Missing required option: "sources".')
        return false
      }
      */

      // ios fullscreen  苹果 全屏
      var playsinline = this.playsinline
      if (playsinline) {
        this.$el.children[0].setAttribute('playsinline', playsinline)
        this.$el.children[0].setAttribute('webkit-playsinline', playsinline)
      }

      // emit event
      var emitPlayerState = function(event, value) {

        if (event) {
          self.$emit(event, self.player)
        }
        if (value) {
          var values = {}
          values[event] = value
          self.$emit(self.customEventName, values)
        }
      }

      // videoOptions
      // console.log(videoOptions)

      // avoid error "VIDEOJS: ERROR: Unable to find plugin: __ob__"
      if (videoOptions.plugins) {
        delete videoOptions.plugins.__ob__
      }





      //开始实例化
      this.player = videojs(this.$el.children[0], videoOptions, function() {



        var playerss = this;
        window.playerss = playerss
        //console.log(videoOptions.fbl)
        playerss.updateSrc(videoOptions.fbl)

        //监听切换视频 未登入时候点击通知vue父组件
        this.on('authority', function() {
          self.$emit('switchUrl', self.player)
          self.player.pause()
          //console.info('Source changed to %s', this.src())
        })
        this.on('resolutionchange', function() {
          //console.info('Source changed to %s', this.src())
        })
        /*this.on('play', function() {
          console.info('Source changed to %s', this.src())
        })*/

        // this.play();
        // //this.pause();
        // console.log(this.pause())



        // player readied
        var _this = this
        self.$emit('ready', self.player)

        // events
        var events = ['loadeddata',
          'canplay',
          'canplaythrough',
          'play',
          'pause',
          'waiting',
          'playing',
          'ended',
          //'error'
        ]
        for (var i = 0; i < events.length; i++) {
          (function(event) {
            _this.on(event, function() {
              emitPlayerState(event, true)
            })
          })(events[i])
        }

        this.on('timeupdate', function() {
          emitPlayerState('timeupdate', this.currentTime())
        })
        /*this.on('error', function(error) {
          //self.$emit('onError', error.errMsg)
          console.log(error)
        })*/







      })

      //this.player.qualityPickerPlugin();





      //获取咱们组件的基类，所有组件都要继承自这个类。
      var Component = videojs.getComponent("Component");








      if (!videoOptions.logothas) {
        //获取咱们组件的基类，所有组件都要继承自这个类。
        var Component = videojs.getComponent("Component");
        //这个组件元素在后面将被添加到播放容器中
        var RefreshComponent = videojs.extend(Component, {
          createEl: function() {
            var img = document.createElement("img");
            img.src = self.img;
            img.className = "vjs-title-buttonhuang";
            return img;
          }
        });

        //实例化这个组件
        var refreshComponent = new RefreshComponent();
        //组件基类实现了事件绑定相关的接口，可以使用on绑定事件，off解绑事件
        //这里给组件绑定了click事件，当点击组件元素的时候会触发
        refreshComponent.on("click", function() {
          //console.log(self.player);
          self.$emit('shuaxin', self.player)
          //myPlayer.refresh();
        });
        this.player.controlBar.addChild(refreshComponent, {});
      }













      //头部提示
      //var titleBars = videojs.getComponent("Component");
      var TitleBar = videojs.extend(Component, {
        constructor: function(player, options) {
          Component.apply(this, arguments);

          if (options.text) {
            this.updateTextContent(options.text);
          }
        },

        createEl: function() {
          return videojs.createEl('div', {
            className: 'vjs-title-bar'
          });
        },
        updateTextContent: function(text) {
          if (typeof text !== 'string') {
            text = 'Title Unknown';
          }
          videojs.emptyEl(this.el());
          videojs.appendContent(this.el(), text);
        }
      });

      videojs.registerComponent('TitleBar', TitleBar);
      this.player.addChild('TitleBar', { text: videoOptions.ctitle });
      //头部提示


      //错误组件使用的元素
      var errorElement = document.createElement("div");
      //添加class   my-error-display
      videojs.addClass(errorElement, "my-error-display");
      //继承组件
      //var Component = videojs.getComponent("Component");
      var ErrorComponent = videojs.extend(Component, {});
      //和上面的刷新按钮那个栗子比较，这里提供了第二种声明组件元素的时机，即在实例化组件的时候传入组件元素
      var errorComponent = new ErrorComponent(null, { el: errorElement });
      this.player.addChild(errorComponent, {});
      this.player.on('error', function(error) {
        //videojs.log.error()
        var errorStatys = self.player.error();
        //console.log(self.player.error())
        if (errorStatys.code == 4) {
          errorElement.innerHTML = "服务器或网络失败或因为格式不受支持，请按F5，刷新页面试试！";
        } else if (errorStatys.code == -3) {
          errorElement.innerHTML = '请按F5，刷新页面试试！';
        } else {
          errorElement.innerHTML = '请按F5，刷新页面试试！';
        }
        //errorElement.innerHTML = '请按F5，刷新页面试试！';
        self.$emit('onError', errorStatys)
      })

      //错误组件使用的元素

      if (videoOptions.ass) {
        //console.log(videoOptions.ass)
        var vjs_ass = this.player.ass(videoOptions.ass,self.subtitle);
      }
      /*var tecksmy = this.player.textTracks()
      for (var i = 0; i < tecksmy.length; i++) {
        alert(1)
        if (tecksmy[i].kind == "subtitles" && tecksmy[i].label == "engsub") {
          textTracksmy = tecksmy[i];
        }
      }
      console.log(tecksmy)*/



    },




    //选择字幕按钮
    sopostah() {
      //字幕
      var self = this;
      //获取咱们组件的基类，所有组件都要继承自这个类。
      var Component = videojs.getComponent("Component");
      //这个组件元素在后面将被添加到播放容器中
      var MySubtitle = videojs.extend(Component, {
        createEl: function() {
          var button = document.createElement("button");
          button.className = 'vjs-hSubtitle open';
          button.innerHTML = '<svg height="100%" version="1.1" viewBox="0 0 36 36" width="100%"><use class="ytp-svg-shadow" xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#ytp-id-17"></use><path d="M11,11 C9.9,11 9,11.9 9,13 L9,23 C9,24.1 9.9,25 11,25 L25,25 C26.1,25 27,24.1 27,23 L27,13 C27,11.9 26.1,11 25,11 L11,11 Z M11,17 L14,17 L14,19 L11,19 L11,17 L11,17 Z M20,23 L11,23 L11,21 L20,21 L20,23 L20,23 Z M25,23 L22,23 L22,21 L25,21 L25,23 L25,23 Z M25,19 L16,19 L16,17 L25,17 L25,19 L25,19 Z" fill="#fff" id="ytp-id-17"></path></svg>';
          //button.innerHTML = 'sss';
          return button;
        }
      });

      //实例化这个组件
      var mySubtitle = new MySubtitle();
      //组件基类实现了事件绑定相关的接口，可以使用on绑定事件，off解绑事件
      //这里给组件绑定了click事件，当点击组件元素的时候会触发
      mySubtitle.on("click", function() {

        //var tracks = self.player.textTracks()[0];

        if (self.isSubtitles) {
          videojs.removeClass(this.el_, 'open');
          self.textTracksmy.mode = 'disabled';
          self.isSubtitles = false;
        } else {
          videojs.addClass(this.el_, 'open');
          self.textTracksmy.mode = 'showing';
          self.isSubtitles = true;
        }

      });
      this.player.controlBar.addChild(mySubtitle, {});

      //字幕
    },

    //离开清除
    dispose: function() {
      if (this.player && videojs) {
        if (this.player.techName_ !== 'Flash') {
          this.player.pause && this.player.pause()
        }
        videojs(this.$el.children[0]).dispose()
        if (!this.$el.children.length) {
          var video = document.createElement('video')
          video.className = 'video-js'
          this.$el.appendChild(video)
        }
        this.player = null
      }
    }
  },
  watch: {
    options(vio, vie) {
      //console.log(vio)
      this.dispose()
      this.initialize();

    },
    comImoveTyoewe(vio, vie) {
      if(vio){
        for (var i = 0; i < vio.length; i++) {
          if (vio[i].kind == "subtitles" && vio[i].label == "engsub") {
            this.textTracksmy = vio[i];
          }
        }
      }
        
    },
    textTracksmy(vio, vie) {
      this.sopostah();
    }
    /*options: {
      deep: true,
      handler: function(options, oldOptions) {
        this.dispose()
        if (options && options.sources && options.sources.length) {
          this.initialize()
        }
      }
    }*/
  },
  //计算
  computed: {
    comImoveTyoewe() {
      if (this.player) {
        if (this.player.textTracks().length) {
          return this.player.textTracks()
        } else {
          return null
        }
      } else {
        return null
      }

    },
  }
}

</script>
