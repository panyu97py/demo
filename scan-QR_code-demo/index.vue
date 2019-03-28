<template>
  <div class="views_app_scan">
    <detailsViewHead/>
    <div class="views_app_scan_mask"></div>
    <video ref="video"  class="views_app_scan_video" x5-video-player-fullscreen="true"></video>
  <canvas ref="canvas"  id="qr-canvas" style="display:none"></canvas>

  </div>
</template>
<script>
import detailsViewHead from '@/components/viewHead'
// import qrcode from './qrcode.js'
export default {
  data () {
    return {
      QR_code: null,
      value: ''
    }
  },
  beforeDestroy () {
    this.close()
  },
  components: {
    detailsViewHead
  },
  mounted () {
    if (
      navigator.mediaDevices.getUserMedia ||
      navigator.getUserMedia ||
      navigator.webkitGetUserMedia ||
      navigator.mozGetUserMedia
    ) {
      // 调用用户媒体设备, 访问摄像头
      this.getUserMedia(
        {
          video: {
            width: this.$refs.video.offsetWidth / 2,
            height: this.$refs.video.offsetHeight / 2
          },
          audio: false
        },
        this.success,
        this.error
      )
    } else {
      alert('不支持访问用户媒体')
    }
    this.initCanvas()
    this.captureToCanvas()
  },
  methods: {
    initCanvas () {
      this.$refs.canvas.style.width = this.$refs.video.offsetWidth / 2 + 'px'
      this.$refs.canvas.style.height = this.$refs.video.offsetHeight / 2 + 'px'
      this.$refs.canvas.width = this.$refs.video.offsetWidth / 2
      this.$refs.canvas.height = this.$refs.video.offsetHeight / 2
      this.QR_code = this.$refs.canvas.getContext('2d')
      this.QR_code.clearRect(0, 0, this.$refs.video.offsetWidth / 2, this.$refs.video.offsetHeight / 2)
    },
    captureToCanvas () {
      try {
        this.QR_code.drawImage(this.$refs.video, 0, 0)// 在canvas元素中绘出video的某一帧
        try {
          window.qrcode.decode()// 扫描二维码
          this.value = window.qrcode.decode()
          this.$router.push({name: 'home', query: { value: this.value }})
        } catch (e) {
          // console.log(e)// 未扫描出二维码，输出错误信息
          setTimeout(this.captureToCanvas, 500)// 500ms之后再重绘canvas
          // document.getElementById('loading').style.display = 'block'
        }
      } catch (e) {
        console.log(e)// 若失败，输出错误信息
        setTimeout(this.captureToCanvas, 500)// 500ms再重绘canvas
      }
    },
    // 访问用户媒体设备的兼容方法
    getUserMedia (constraints, success, error) {
      if (navigator.mediaDevices.getUserMedia) {
        // 最新的标准API
        navigator.mediaDevices
          .getUserMedia(constraints)
          .then(success)
          .catch(error)
      } else if (navigator.webkitGetUserMedia) {
        // webkit核心浏览器
        navigator.webkitGetUserMedia(constraints, success, error)
      } else if (navigator.mozGetUserMedia) {
        // firfox浏览器
        navigator.mozGetUserMedia(constraints, success, error)
      } else if (navigator.getUserMedia) {
        // 旧版API
        navigator.getUserMedia(constraints, success, error)
      }
    },
    success (stream) {
      // 兼容webkit核心浏览器
      // let CompatibleURL = window.URL || window.webkitURL
      // 将视频流设置为video元素的源
      // console.log(stream)
      // this.$refs.video.src = CompatibleURL.createObjectURL(stream)
      this.$refs.video.srcObject = stream
      this.$refs.video.play()
    },
    error (error) {
      console.log(`访问用户媒体设备失败${error.name}, ${error.message}`)
    },
    close () {
      this.$refs.video.srcObject.stop()
    }
  }
}
</script>
<style lang="scss" scoped>
.views_app_scan {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  &_mask {
    width: 200px;
    height: 200px;
    position: absolute;
    z-index: 3;
    left: 0;
    right: 0;
    bottom: 0;
    top: 79px;
    margin: auto;
    background: transparent;
    border-left: calc( 50vw - 100px ) solid
      rgba($color: #000000, $alpha: 0.5);
    border-right: calc(50vw - 100px) solid
      rgba($color: #000000, $alpha: 0.5);
    border-top: calc(50vh - 100px - 38.5px - 40px) solid
      rgba($color: #000000, $alpha: 0.5);
    border-bottom:  calc(50vh - 100px - 38.5px + 40px) solid
      rgba($color: #000000, $alpha: 0.5);
  }
  &_mask:after{
    content:"请将二维码放入框内，即可自动扫描";
    position: absolute;
    color:rgba($color: #ffffff, $alpha: 0.3);
    bottom: -40px;
    left: -30px;
    right: -30px;
    text-align: center;
    font-size: 15px;
  }
  &_video {
    position: relative;
    z-index: 2;
    width: 100%;
    height: calc(100% - 79px);
  }
}
</style>
