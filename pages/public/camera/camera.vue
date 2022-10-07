<template>
  <fui-page>
    <view class="preview-box">
      <text>效果预览</text>
      <view class="preview-avatar-box">
        <view class="preview-avatar">
          <image :src="previewUrl" class="preview-avatar-img" style="width: 150px;height: 150px;"></image>
          <image :src="frameUrl"
                 class="preview-frame"></image>
          <!--      <canvas id="preview" canvas-id="preview"-->
          <!--              style="height: 150px;width: 150px;margin: 0 auto"></canvas>-->
        </view>
      </view>
    </view>

    <view class="camera-box">
      <image-cropper id="image-cropper"
                     :limit_move="true"
                     :disable_rotate="false"
                     :disable_ratio="true"
                     :img_width="imgWidth"
                     :img_height="imgHeight"
                     :width="width"
                     :height="height"
                     :scale="scale"
                     :imgSrc="coverImage"
                     :max_width="200"
                     :max_height="200"
                     @load="cropperload"
                     @imageload="loadimage"
                     @tapcut="clickcut"
                     @move_stop="moveStop"
                     @canvasDraw="canvasDraw"
      ></image-cropper>
    </view>

    <view class="tools fui-flex-between fui-p25">
      <button type="default" @tap="cancel">取消</button>
      <button type="default" @tap="cutImage">确认</button>
    </view>
  </fui-page>
</template>

<script>
export default {
  data() {
    return {
      imgWidth: 0,
      imgHeight: 0,
      width: 0,
      height: 0,
      scale: 1,
      coverImage: '',
      ctx: null,
      previewUrl: '',
      frameUrl: 'https://vkceyugu.cdn.bspapp.com/VKCEYUGU-bf4cef3b-71c8-46f4-92c5-7aaf14fc015b/ccc7577a-fc00-4d3d-8982-c413d02c9de8.png'
    }
  },
  onLoad({url}) {
    this.$data.coverImage = url
    // #ifdef APP-NVUE
    const eventChannel = this.$scope.eventChannel; // 兼容APP-NVUE
    // #endif
    // #ifndef APP-NVUE
    const eventChannel = this.getOpenerEventChannel();
    // #endif
    this.$data.eventChannel = eventChannel


  },
  onReady() {
    // this.$refs.preview = wx.createCanvasContext("preview", this);
  },
  methods: {
    GCD(w, h) {
      if (w % h) {
        return this.GCD(h, w % h)
      } else {
        return h
      }
    },
    loadimage(data) {
      console.log('图片加载完成', data)
      uni.hideLoading()
      let info = uni.getSystemInfoSync()
      let w = data.detail.width
      let h = data.detail.height
      let gcd = this.GCD(w / 2, h / 2)
      this.$data.width = info.windowWidth * 0.6
      this.$data.height = info.windowWidth * 0.6
      this.$data.imgWidth = info.windowWidth * 0.6
      console.log(this.$data.width)
      this.$data.imgHeight = info.windowWidth * 0.6 / (w / gcd) * h / gcd
      console.log(this.$data.imgHeight)
      this.$nextTick(() => {
        this.cropper.setCutCenter();
        this.$nextTick(() => {
          this.cropper.imgReset();
        })
      })
    },
    cropperload() {
      this.cropper = this.selectComponent("#image-cropper")
    },
    clickcut() {
    },
    cutImage() {
      this.cropper.getImg(({url}) => {
        console.log(url)
        this.$data.eventChannel.emit('save', url)
        uni.navigateBack({
          delta: 1
        });
      })
    },
    cancel() {
      console.log(' 取消')
      if (this.$data.eventChannel.emit) {
        this.$data.eventChannel.emit('cancel')
      }
      this.$back()
    },
    canvasDraw(e) {
      const detail = e.detail
      // console.log(detail)
      const ctx = uni.createCanvasContext("preview", this)
      ctx.translate(detail.xpos * detail.export_scale, detail.ypos * detail.export_scale);
      ctx.drawImage(detail.img,
          -detail.img_width / 2, -detail.img_height / 2, detail.img_width, detail.img_height,
          0, 0, 150, 150);
      ctx.draw();
    },
    moveStop() {
      this.cropper.getImg((res) => {
        console.log(res.url)
        this.$data.previewUrl = res.url
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.preview-box {
  position: absolute;
  top: 0;
  width: 100%;
  padding: 50px 0 20px 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  vertical-align: center;
  align-items: center;
  background-color: #f4f5f6;
  box-shadow: 0 1px 10px #888888;
  z-index: 50;
}

.preview-box text {
  font-size: 14px;
  font-weight: 400;
  color: #555555;
}

.preview-avatar-box {
  width: 150px;
  height: 150px;
  padding: 5px;
  overflow: hidden;
}

.preview-avatar {
  position: relative;
  width: 100%;
  height: 100%;
  border-radius: 10px;
  border: #cccccc 1px solid;
  overflow: hidden;
}

.preview-avatar image {
  display: block;
  width: 100%;
  height: 100%;
}

.preview-frame {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}

.camera-box {
  /* position: relative; */
  width: 80vw;
  height: 80vh;
  margin: auto;
}

.tools {
  position: fixed;
  bottom: 10px;
  width: 100%;
  z-index: 99;
  box-sizing: border-box;

  button {
    width: 50px;
    height: 50px;
    line-height: 50px;
    text-align: center;
    border-radius: 50%;
    font-size: 14px;
    padding: 0;
  }
}
</style>
