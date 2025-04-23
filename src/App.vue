<template>
  <div class="container">
    <h1>颜色转换器</h1>
    <div class="input-group">
      <input 
        v-model="colorInput" 
        type="text" 
        placeholder="输入16进制或RGBA颜色值"
        @keyup.enter="convertColor"
      />
      <button @click="convertColor">转换</button>
    </div>
    <div v-if="convertedColor" class="result">
      转换结果：{{ convertedColor }}
    </div>
    <div 
      v-if="convertedColor" 
      class="color-display"
      :style="{ backgroundColor: convertedColor }"
    ></div>

    <!-- 添加明度调整部分 -->
    <div class="input-group">
      <input
        v-model="brightnessValue"
        type="number"
        min="0"
        max="100"
        placeholder="输入明度 (0-100)"
      />
      <button @click="adjustBrightness">调整明度</button>
    </div>
    <div v-if="brightnessColor" class="result">
      明度调整结果：{{ brightnessColor }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      colorInput: '#000000',
      convertedColor: 'rgba(0, 0, 0, 1)',
      brightnessValue: 50,
      brightnessColor: '',
      colorDisplay: ''  // 添加 colorDisplay 定义
    }
  },
  mounted() {
    this.colorDisplay = this.convertedColor
  },
  methods: {
    convertColor() {
      const input = this.colorInput.trim()
      
      // 16进制转RGBA
      if (/^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/.test(input)) {
        this.convertedColor = this.hexToRgba(input)
      }
      // RGBA转16进制
      else if (/^rgba?\((\d{1,3}%?),\s*(\d{1,3}%?),\s*(\d{1,3}%?)(?:,\s*(\d*(?:\.\d+)?))?\)$/.test(input)) {
        this.convertedColor = this.rgbaToHex(input)
      }
      else {
        this.convertedColor = '无效的颜色格式'
      }
    },
    hexToRgba(hex) {
      let r = 0, g = 0, b = 0, a = 1;
      
      // 3位hex
      if (hex.length === 4) {
        r = parseInt(hex[1] + hex[1], 16)
        g = parseInt(hex[2] + hex[2], 16)
        b = parseInt(hex[3] + hex[3], 16)
      }
      // 6位hex
      else if (hex.length === 7) {
        r = parseInt(hex.slice(1, 3), 16)
        g = parseInt(hex.slice(3, 5), 16)
        b = parseInt(hex.slice(5, 7), 16)
      }
      
      return `rgba(${r}, ${g}, ${b}, ${a})`
    },
    rgbaToHex(rgba) {
      const [r, g, b] = rgba.match(/\d+/g)
      return `#${((1 << 24) + (Number(r) << 16) + (Number(g) << 8) + Number(b)).toString(16).slice(1)}`
    },
    adjustBrightness() {  // 在前面添加逗号
      const brightness = parseFloat(this.brightnessValue) / 100
      if (isNaN(brightness) || brightness < 0 || brightness > 1) {
        this.brightnessColor = '无效的明度值'
        return
      }

      const color = this.convertedColor.startsWith('#') ? 
        this.convertedColor : 
        this.rgbaToHex(this.convertedColor)

      const rgb = this.hexToRgb(color)
      const adjusted = rgb.map(c => Math.round(c * brightness))
      this.brightnessColor = `#${((1 << 24) + (adjusted[0] << 16) + (adjusted[1] << 8) + adjusted[2]).toString(16).slice(1)}`
    },
    hexToRgb(hex) {
      const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex)
      return result ? [
        parseInt(result[1], 16),
        parseInt(result[2], 16),
        parseInt(result[3], 16)
      ] : [0, 0, 0]
    }
  }
}
</script>

<style>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.input-group {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 8px 16px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

.result {
  padding: 10px;
  background-color: #f5f5f5;
  border-radius: 4px;
  margin-bottom: 20px;
}

/* 添加颜色显示样式 */
.color-display {
  width: 100%;
  height: 100px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
</style>
