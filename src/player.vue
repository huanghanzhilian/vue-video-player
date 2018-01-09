<template>
  <div class="video-player">
    <video class="video-js"></video>
  </div>
</template>
<script>
window.videojs = require('video.js')
videojs = videojs.default || videojs
export default {
  name: 'video-player',
  data() {
    return {
      'img': require("./img/logo-w.png"),
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
    }
  },
  mounted: function() {
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
        autoplay: false,
        controls: true,
        preload: 'auto',
        fluid: false,
        muted: false,
        width: '100%',
        height: '100%',
        language: 'en',
        controlBar: {
          remainingTimeDisplay: false,
          playToggle: {},
          progressControl: {},
          fullscreenToggle: {},
          volumeMenuButton: {
            inline: false,
            vertical: true
          }
        },
        techOrder: ['html5'],
        plugins: {}
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

      this.player = videojs(this.$el.children[0], videoOptions, function() {
        console.log(videoOptions.fbl)
        var playerss = this;
        window.playerss = playerss
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
      var Component = videojs.getComponent("Component");

      if(!videoOptions.logothas){
        //获取咱们组件的基类，所有组件都要继承自这个类。
        var Component = videojs.getComponent("Component");
        //这个组件元素在后面将被添加到播放容器中
        var RefreshComponent = videojs.extend(Component, {
          createEl: function() {
            /*var button = document.createElement("button");
            button.innerHTML = "刷新";
            button.className = "vjs-title-buttonhuang"
            return button;*/
            var img=document.createElement("img");
            img.src=self.img;
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
        errorElement.innerHTML = '出现故障';
        self.$emit('onError', error)
      })




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
      this.initialize()
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
  }
}

</script>
