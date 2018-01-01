<template>
  <span>
    <input type="file" ref="input"  @change="handleFileChange" id="file" class="inputfile" accept="image/png, image/jpeg, image/gif, image/jpg"/>
    <label for="file" :style="{padding:pad}">上传图片</label>
  </span>
</template>

<script>
  export default {
    data () {
      return {
        dataUrl: ''
      }
    },
    methods: {
      imgPreview (file) {
        let self = this
        // 看支持不支持FileReader
        if (!file || !window.FileReader) return
        // 创建一个reader
        let reader = new FileReader()
        // 将图片将转成 base64 格式
        reader.readAsDataURL(file)
        // 读取成功后的回调
        reader.onloadend = function () {
          self.dataUrl = this.result
          self.mes = 'gggg'
        }
      },
      handleFileChange (e) {
        let file = this.$refs.input.files[0]
        this.$emit('fileChange', file)
      }
    },
    props: {
      pad: String
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .inputfile{
    width: 0.1px;
    height: 0.1px;
    opacity: 0;
    overflow: hidden;
  }
   label {
    font-weight: 700;
    color: white;
    background-color: rgba(3, 160, 77, 0.68);
    cursor: pointer;
    border-radius: 5px;
  }

  .inputfile:focus + label,
  .inputfile + label:hover {
    background-color: #0aceff;
  }

</style>
