<template>
  <div class="d-flex justify-content-center">
    <div style="width: 450px;">
      <video v-show="!isPhotoTaken" ref="camera" :width="width" :height="height" autoplay></video>
      <canvas v-show="isPhotoTaken" id="photoTaken" ref="canvas" :width="width" :height="height"></canvas>
      <br>
      <div class="d-flex justify-content-center">
        <button v-show="isOnCamera" type="button" class="btn btn-primary" @click="stopCamera">Stop Camera</button>
        <button v-show="!isOnCamera" type="button" class="btn btn-primary" @click="startCamera">Start Camera</button>
        <button v-show="!isPhotoTaken" type="button" class="btn btn-warning text-white mx-2" @click="takePhoto">Take Photo</button>
        <button v-show="isPhotoTaken" type="button" class="btn btn-success mx-2" @click="isPhotoTaken = false">Retake Photo</button>
      </div>
      <div class="d-flex justify-content-center">
        <button type="button" class="btn btn-success m-2" @click="switchCam"> Switch Camera </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isPhotoTaken : false,
      isOnCamera: true,
      width: 450,
      height: 337.5,
      cam: "environment",
    }
  },
  computed: {
    elementWidth() {
      const element = this.$refs.camera
      return element.offsetWidth
    },
    elementHeight() {
      const element = this.$refs.camera
      return element.offsetHeight
    },
  },
  mounted() {
    this.startCamera()
    window.addEventListener('resize', this.handleResize)
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.handleResize)
  },
  methods: { 
    startCamera() {
      this.isOnCamera = true
      this.isPhotoTaken = false
      const constraints = (window.constraints = {
				audio: false,
				video: {
          facingMode: this.cam,
        }
			})

			navigator.mediaDevices.getUserMedia(constraints)
				.then(stream => {
					this.$refs.camera.srcObject = stream
				})
				.catch(error => {
					alert("May the browser didn't support or there is some errors.")
				})
    },
    stopCamera() {
      this.isOnCamera = false
      const tracks = this.$refs.camera.srcObject.getTracks()

			tracks.forEach(track => {
				track.stop()
			})
    },
    takePhoto() {
      this.isPhotoTaken = true
      const context = this.$refs.canvas.getContext('2d')
      context.drawImage(this.$refs.camera, 0, 0, this.width, this.height)
      const canvas = document.getElementById("photoTaken").toDataURL("image/jpeg")
      const fileName = 'imageCapture.jpg'
      this.base64ToImage(canvas, fileName)
    },
    switchCam() {
      if(this.cam == "environment") {
        this.cam = "user"
        this.stopCamera()
        this.startCamera()
      } else {
        this.cam = "environment"
        this.stopCamera()
        this.startCamera()
      }
    },
    base64ToImage(base64Data, fileName) {
      const link = document.createElement('a')
      link.href = base64Data
      link.download = fileName
      link.click()
    },
    handleResize() {
      this.width = this.elementWidth
      this.height = this.elementHeight
      console.log(this.width)
      // console.log(this.height)
    },
  },
}
</script>
<style>
  video {
    /* width: 100%;
    height: auto;
    background-color: grey; */
  }
  canvas {
    /* width: 100%;
    height: 337.5px; */
    /* max-width: 450px !important; */
  }
</style>
