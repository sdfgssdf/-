<template>
  <div class="container" @mouseup="draggable = false">
    <div class="canvas-wrap" ref="imgwrap" @click="selected = false">
      <canvas id="backgroud" ref="bg" @mousemove="drag"></canvas>
      <UploadButton @fileChange="imgfile = arguments[0]" pad="20px" v-if="!imgfile" style="position: absolute;"></UploadButton>
      <input v-if="imgfile" v-for="(item,index)  in textArray" @click.stop="select(index)" @input="handleTextInput(item,$event)" type="text" :style="{ top: item.top, left: item.left }" placeholder="单击输入拖拽" @mousedown="dragBegin(index , $event)" onfocus="this.style.color='transparent'" >
    </div>

    <div class="card-wrap">
      <div class="row row-center">
        <button @click="addText" :disabled="(!imgfile)">增加文本</button>
        <button @click="removeText" :disabled="(!selected)&&(selected!==0)">删除文本</button>
        <label>文字颜色<input type="color" @input="handleInput('color',$event.target.value)"></label>
        <button @click="clearImg" :disabled="(!imgfile)">全部清除</button>
        <button @click="drawImg" :disabled="(!imgfile)">生成图片</button>
      </div>
      <div class="row row-center">
        <label><input type="radio" name="font" @change="handleInput('fontfamily','Microsoft YaHei,微软雅黑, Hiragino Sans GB ,冬青黑体')" checked>默认字体</label>
        <label><input type="radio" name="font" @change="handleInput('fontfamily','KaiTi,楷体,STKaiti,华文楷体')">楷体</label>
        <label><input type="radio" name="font" @change="handleInput('fontfamily','SimSun,宋体,STSong,华文宋体')">宋体</label>
        <label>文字阴影<input type="checkbox" name="shadow" @change="handleInput('shadow',!shadow)"></label>
        <label>空心字<input type="checkbox" name="stroke" @change="handleInput('stroke',!stroke)"></label>
      </div>
      <div class="row"><label><span>字体大小</span><input type="range" @input="handleInput('fontsize',$event.target.value)"></label></div>
      <div class="row"><label><span>文本透明度</span><input  type="range" min="0" step="0.1" max="1" @input="handleInput('alpha',$event.target.value)"></label></div>
      <div class="row"><label><span>文本粗细</span><input  type="range" min="100" step="100" max="900" @input="handleInput('fontweight',$event.target.value)"></label></div>
      <div class="row"><a href="#">GITHUB 地址</a></div>
    </div>
  </div>
</template>

<script>
import UploadButton from './UploadBt.vue'
export default {
  components: {
    UploadButton
  },
  name: 'HelloWorld',
  data () {
    return {
      mes: 'cdc',
      initX: 0, // 表单上的偏移量
      initY: 0,
      initTextX: 0, // 初始文本位置
      initTextY: 0,
      scaleRatio: 1, // 图片的缩放比
      draggable: false, // 被拖拽的index
      selected: false, // 被选中的index
      imgfile: undefined,
      img: '',
      textArray: [  // 文字数组
        {
          fontsize: 80,
          alpha: 1,
          stroke: false,
          fontweight: 400,
          fontfamily: 'Microsoft YaHei,微软雅黑, Hiragino Sans GB ,冬青黑体',
          content: '',
          color: '#00DC48',
          shadow: false,
          left: 'auto', // 表单位置
          top: 'auto',
          x: 300, // 文字位置
          y: 300
        }],
      fontsize: 80, // 下面为表单双向绑定值
      alpha: 1,
      stroke: false,
      fontweight: 400,
      fontfamily: 'Microsoft YaHei,微软雅黑, Hiragino Sans GB ,冬青黑体',
      color: '#00DC48',
      shadow: false
    }
  },
  watch: {
    // 如果 `imgflle` 发生改变，这个函数就会运行
    imgfile: function (imgfile) {
      let self = this
      // 看支持不支持FileReader
      if (!imgfile || !window.FileReader) return
      // 创建一个reader
      let reader = new FileReader()
      // 将图片将转成 base64 格式
      reader.readAsDataURL(imgfile)

      // 读取成功后的回调
      reader.onloadend = function () {
        let img = new Image()  // 创建img元素
        img.src = this.result // 设置图片源地址
        img.onload = function () {
          // 缩放图片
          let wrapRatio = self.$refs.imgwrap.offsetWidth / self.$refs.imgwrap.offsetHeight
          let imgRatio = img.width / img.height
          self.$refs.bg.width = img.width
          self.$refs.bg.height = img.height
          self.img = img// 缓存img
          if (wrapRatio > imgRatio) {
            self.$refs.bg.style.height = '100%'
            self.scaleRatio = img.height / self.$refs.imgwrap.offsetHeight// 获得缩放比例
            self.initTextX = (self.$refs.bg.offsetWidth - 258) / 2
            self.initTextY = (self.$refs.bg.offsetHeight - 43) / 2 + 20
            self.textArray[0].x = (self.$refs.bg.offsetWidth - 258) / 2
            self.textArray[0].y = (self.$refs.bg.offsetHeight - 43) / 2 + 20
          } else {
            self.$refs.bg.style.width = '100%'
            self.scaleRatio = img.width / self.$refs.imgwrap.offsetWidth
            self.initTextX = (self.$refs.bg.offsetWidth - 258) / 2
            self.initTextY = (self.$refs.bg.offsetHeight - 43) / 2 + 20
            self.textArray[0].x = (self.$refs.bg.offsetWidth - 258) / 2
            self.textArray[0].y = (self.$refs.bg.offsetHeight - 43) / 2 + 20
          }
          self.$refs.bg.style.background = 'url(' + img.src + ')'
        }
      }
    }
  },
  methods: {
    addText: function () {
      this.textArray.push({
        fontsize: this.fontsize,
        alpha: this.alpha,
        stroke: this.stroke,
        fontweight: this.fontweight,
        fontfamily: this.fontfamily,
        content: '',
        color: this.color,
        shadow: this.shadow,
        left: 'auto',
        top: 'auto',
        x: this.initTextX,
        y: this.initTextY
      })
    }, // 增加文字
    handleInput: function (type, value) {
      this[type] = value
      if (this.selected || this.selected === 0) {
        this.textArray[this.selected][type] = value
        this.mes = value
        this.draw()
      }
    }, // 处理表单input事件
    removeText: function () {
      if (this.selected || this.selected === 0) {
        this.textArray.splice(this.selected, 1)
        this.selected = false
        this.draw()
      }
    }, // 移除文字
    clearImg: function () {
      this.textArray = []
      this.draw()
    },  // 清除画布
    drawImg: function () {
      let ctx = this.$refs.bg.getContext('2d')
      let ratio = this.scaleRatio
      let width = this.$refs.bg.width
      let height = this.$refs.bg.height
      ctx.clearRect(0, 0, width, height)
      ctx.drawImage(this.img, 0, 0)
      function drawtext (item) {
        ctx.globalAlpha = item.alpha
        ctx.font = item.fontweight + ' ' + item.fontsize + 'px ' + item.fontfamily
        ctx.textBaseline = 'middle'
        ctx.shadowColor = 'rgba(0, 0, 0, 0)'
        ctx.shadowOffsetX = 0
        ctx.shadowOffsetY = 0
        if (item.stroke) {
          if (item.shadow) {
            ctx.shadowColor = 'rgba(0, 0, 0, 0.8)'
            ctx.shadowOffsetX = 10
            ctx.shadowOffsetY = 10
          }
          ctx.strokeStyle = item.color
          ctx.strokeText(item.content, item.x * ratio, item.y * ratio)
        } else {
          if (item.shadow) {
            ctx.shadowColor = 'rgba(0, 0, 0, 0.2)'
            ctx.shadowOffsetX = 10
            ctx.shadowOffsetY = 10
          }
          ctx.fillStyle = item.color
          ctx.fillText(item.content, item.x * ratio, item.y * ratio)
        }
      }
      this.textArray.forEach(drawtext)
      alert('右击图片保存')
    }, // 生成最终图片
    select: function (index) {
      this.selected = index
    }, // 选中文本框
    dragBegin: function (index, e) {
      this.draggable = index
      this.initX = e.offsetX
      this.initY = e.offsetY
    }, // 开始拖动
    drag: function (e) {
      if (this.draggable || this.draggable === 0) {
        let item = this.textArray[this.draggable]
        item.left = e.clientX - this.initX + 'px'
        item.top = e.clientY - this.initY + 'px'
        item.x = e.offsetX - this.initX
        item.y = e.offsetY - this.initY + 20
        this.draw()
      }
    }, // 拖动文本框
    handleTextInput: function (item, e) {
      item.content = e.target.value
      this.draw()
    }, // 处理文本框input事件
    draw: function () {
      let ctx = this.$refs.bg.getContext('2d')
      let ratio = this.scaleRatio
      let width = this.$refs.bg.width
      let height = this.$refs.bg.height
      ctx.clearRect(0, 0, width, height)
      function drawtext (item) {
        ctx.globalAlpha = item.alpha
        ctx.font = item.fontweight + ' ' + item.fontsize + 'px ' + item.fontfamily
        ctx.textBaseline = 'middle'
        ctx.shadowColor = 'rgba(0, 0, 0, 0)'
        ctx.shadowOffsetX = 0
        ctx.shadowOffsetY = 0
        if (item.stroke) {
          if (item.shadow) {
            ctx.shadowColor = 'rgba(0, 0, 0, 0.8)'
            ctx.shadowOffsetX = 10
            ctx.shadowOffsetY = 10
          }
          ctx.strokeStyle = item.color
          ctx.strokeText(item.content, item.x * ratio, item.y * ratio)
        } else {
          if (item.shadow) {
            ctx.shadowColor = 'rgba(0, 0, 0, 0.2)'
            ctx.shadowOffsetX = 10
            ctx.shadowOffsetY = 10
          }
          ctx.fillStyle = item.color
          ctx.fillText(item.content, item.x * ratio, item.y * ratio)
        }
      }
      this.textArray.forEach(drawtext)
    } //  生成图片
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.container{
  width:500px;
  margin: auto;
  overflow: hidden;
  background-color: #f7fff3;
}
  .canvas-wrap{
    display: flex;
    justify-content:center;
    height: 60vh;
    align-items:center;
    overflow: hidden;
  }
.card-wrap{
  display: flex;
  flex-direction: column;
  justify-content:space-around;
  height:40vh;
  color: #03a04d;
}
.card-wrap button{
  border-radius: 10%;
  background-color: #d0ffde;
}
.row{
  width: 100%;
}
.row-center {
  display: flex;
  justify-content:space-around;;
}
canvas{
  background-size: 100% auto!important;
  }
input[type= 'color']{
  border-radius: 50%;
  background-color: aqua;
  border-color: #cd1215;
  outline: seagreen;
}
input[type= 'text']{
  caret-color: #cd615a;
  position: absolute;
  padding-left: 3px;
  border: none;
  font-size: 36px;
  background-color:transparent;
  width:260px;
  user-select: none;
}
:-moz-placeholder { /* Mozilla Firefox 4 to 18 */
  color: #00dc48; opacity:.8;
}

::-moz-placeholder { /* Mozilla Firefox 19+ */
  color: #00dc48;opacity:.8;
}

input:-ms-input-placeholder{
  color: #00dc48;opacity:.8;
}

input::-webkit-input-placeholder{
  color: #00dc48;opacity:.8;
}
div canvas {
    padding: 0;
    margin: 0;
    border: none;
  }
/*  以下滑动条样式COPY自网上 */
input[type=range] {
  -webkit-appearance: none;
  margin: 0 10px;
  width: 80%;
}
input[type=range]:focus {
  outline: none;
}
input[type=range]::-webkit-slider-runnable-track {
  width: 100%;
  height: 8.4px;
  cursor: pointer;
  animate: 0.2s;
  background: #02dc77;
  border-radius: 1.3px;
  border: 0.2px solid #0f60dc;
}
input[type=range]::-webkit-slider-thumb {
  border: 1px solid #0f60dc;
  height: 36px;
  width: 16px;
  border-radius: 3px;
  background: #ffffff;
  cursor: pointer;
  -webkit-appearance: none;
  margin-top: -14px;
}
input[type=range]:focus::-webkit-slider-runnable-track {
  background: #03a04d;
}
input[type=range]::-moz-range-track {
  width: 100%;
  height: 8.4px;
  cursor: pointer;
  animate: 0.2s;
  background: #03a04d;
  border-radius: 1.3px;
  border: 0.2px solid #0f60dc;
}
input[type=range]::-moz-range-thumb {
  border: 1px solid #0f60dc;
  height: 36px;
  width: 16px;
  border-radius: 3px;
  background: #ffffff;
  cursor: pointer;
}
input[type=range]::-ms-track {
  width: 100%;
  height: 8.4px;
  cursor: pointer;
  animate: 0.2s;
  background: transparent;
  border-color: transparent;
  border-width: 16px 0;
  color: transparent;
}
input[type=range]::-ms-fill-lower {
  background: #03a04d;
  border: 0.2px solid #0f60dc;
  border-radius: 2.6px;
}
input[type=range]::-ms-fill-upper {
  background: #03a04d;
  border: 0.2px solid #0f60dc;
  border-radius: 2.6px;
}
input[type=range]::-ms-thumb {
  border: 1px solid #0f60dc;
  height: 36px;
  width: 16px;
  border-radius: 3px;
  background: #ffffff;
  cursor: pointer;
}
input[type=range]:focus::-ms-fill-lower {
  background: #03a04d;
}
input[type=range]:focus::-ms-fill-upper {
  background: #03a04d;
}

</style>
