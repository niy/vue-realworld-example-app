<template>
  <div class="video-player-container">
    <p v-if="videoStatus === 'uploaded'">Video will be ready within few minutes...</p>
    <p v-if="videoStatus === 'failed'" class="text-danger">Video failed to convert. Please try another one.</p>
    <video v-if="videoStatus === 'converted'" class="video-player"
           ref="videoPlayer" controls="controls"
           preload="metadata" v-bind:poster="video.thumbnail_url"></video>
  </div>
</template>
<script>
  import Hls from 'hls.js'

  export default {
    name: 'RwvVideoPlayer',
    props: {
      video: { type: Object, required: true }
    },
    mounted () {
      this.init()
    },
    beforeDestroy () {
      clearInterval(this.checkVideoInterval)
    },
    computed: {
      videoStatus: function () {
        return this.video.upload_status
      }
    },
    watch: {
      videoStatus: function (val) {
        if (val === 'converted') {
          this.loadVideo()
        }
      }
    },
    methods: {
      init () {
        this.loadVideo()

        // @TODO: remove this, this is only for testing
        this.video.upload_status = 'converted'

        // @TODO: periodically check the backend to see if the file is converted then loadVideo

        // const self = this
        // VideosService.checkVideoStatus(this.video)
        //   .then(function (video) {
        //     if (video.status === 'uploaded') {
        //       // check every 30 seconds for video
        //       // @TODO: only set interval if response is 404
        //       self.checkVideoInterval = setInterval(self.init, 30 * 1000)
        //     } else if (video.status === 'completed') {
        //       if (self.checkVideoInterval) {
        //         clearInterval(self.checkVideoInterval)
        //       }
        //       self.loadVideo()
        //     }
        //   })
        //   .catch(function (response) {
        //   })
      },
      loadVideo () {
        let videoPlayerDom = this.$refs.videoPlayer
        if (Hls.isSupported()) {
          let hls = new Hls()
          hls.loadSource(this.video.url)
          hls.attachMedia(videoPlayerDom)
        } else if (videoPlayerDom.canPlayType('application/vnd.apple.mpegurl')) {
          videoPlayerDom.src = this.video.url
        }
      }
    }
  }
</script>
