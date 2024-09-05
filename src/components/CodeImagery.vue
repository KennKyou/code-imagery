<script setup>
import { ref, onMounted, watch } from 'vue';
import { ElMessage } from 'element-plus';
import hljs from 'highlight.js';
import 'highlight.js/styles/github.css';
import 'highlight.js/styles/github-dark.css';
import html2canvas from 'html2canvas';

const code = ref('');
const highlightedCode = ref('');
const codeContainer = ref(null);
const language = ref('auto');
const theme = ref('github-dark');

const highlightCode = () => {
  if (language.value === 'auto') {
    highlightedCode.value = hljs.highlightAuto(code.value).value;
  } else {
    highlightedCode.value = hljs.highlight(code.value, { language: language.value }).value;
  }
};

const generateImage = async () => {
  if (!codeContainer.value || !code.value.trim()) return;
  
  try {
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');

    const { width, height } = codeContainer.value.getBoundingClientRect();

    canvas.width = width * 2;
    canvas.height = height * 2;

    // 設置圓角
    ctx.beginPath();
    ctx.moveTo(0, 8);
    ctx.arcTo(0, 0, 8, 0, 8);
    ctx.lineTo(canvas.width - 8, 0);
    ctx.arcTo(canvas.width, 0, canvas.width, 8, 8);
    ctx.lineTo(canvas.width, canvas.height - 8);
    ctx.arcTo(canvas.width, canvas.height, canvas.width - 8, canvas.height, 8);
    ctx.lineTo(8, canvas.height);
    ctx.arcTo(0, canvas.height, 0, canvas.height - 8, 8);
    ctx.closePath();
    ctx.clip();

    const tempCanvas = await html2canvas(codeContainer.value, {
      scale: 2,
      backgroundColor: null,
    });

    // 將臨時 canvas 繪製到主 canvas 上
    ctx.drawImage(tempCanvas, 0, 0);

    // 將 canvas 轉換為圖片
    const image = canvas.toDataURL('image/png');
    const link = document.createElement('a');
    link.href = image;
    link.download = `code-image-${Date.now()}.png`;
    link.click();

    ElMessage({
      message: '圖片已成功生成並下載',
      type: 'success',
      duration: 3000
    });
  } catch (error) {
    console.error('生成圖片時發生錯誤：', error);
    
    ElMessage({
      message: '生成圖片時發生錯誤',
      type: 'error',
      duration: 3000
    });
  }
};

const clearCode = () => {
  code.value = '';
};

watch([code, language, theme], () => {
  highlightCode();
});

onMounted(() => {
  highlightCode();
});
</script>

<template>
  <div class="code-imagery">
    <div class="controls">
      <el-select v-model="language" placeholder="選擇語言">
        <el-option label="自動檢測" value="auto"></el-option>
        <el-option label="JavaScript" value="javascript"></el-option>
        <el-option label="Python" value="python"></el-option>
        <el-option label="Java" value="java"></el-option>
        <!-- 可以添加更多語言選項 -->
      </el-select>
      <el-select v-model="theme" placeholder="選擇主題">
        <el-option label="深色主題" value="github-dark"></el-option>
        <!-- <el-option label="淺色主題" value="github"></el-option> -->
        <!-- 可以添加更多主題選項 -->
      </el-select>
    </div>
    <el-input
      v-model="code"
      type="textarea"
      :rows="10"
      placeholder="請輸入您的程式碼"
    />
    <div class="action-buttons">
      <el-button type="primary" @click="generateImage" :disabled="!code.trim()" class="generate-btn">
        生成圖片
      </el-button>
      <el-button type="warning" @click="clearCode" :disabled="!code.trim()" class="clear-btn">
        清除程式碼
      </el-button>
    </div>
    <div v-if="code.trim()" ref="codeContainer" class="code-container" :class="theme">
      <div class="code-header">
        <span></span>
        <span></span>
        <span></span>
      </div>
      <pre><code v-html="highlightedCode"></code></pre>
    </div>
  </div>
</template>

<style scoped>
.code-imagery {
  padding: 20px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  max-width: 800px;
  width: 800px;
  margin: 0 auto;
}

.page-title {
  margin-bottom: 20px;
  color: #409EFF;
  font-size: 1.5rem;
}

.controls {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
  gap: 10px;
}

.el-select {
  flex: 1;
}

.el-input {
  margin-bottom: 20px;
}

.action-buttons {
  display: flex;
  margin-bottom: 20px;
  margin-top: 20px;
  gap: 10px;
}

.generate-btn,
.clear-btn {
  flex-grow: 1;
  transition: transform 0.2s, box-shadow 0.2s;
}

.generate-btn:hover,
.clear-btn:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.generate-btn:active,
.clear-btn:active {
  transform: translateY(0);
  box-shadow: none;
}

.code-container {
  padding: 0 20px 20px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  overflow-x: auto;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.code-header {
  display: flex;
  align-items: center;
  height: 35px;
  margin-bottom: 10px;
}

.code-header span {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  margin-right: 8px;
}

.code-header span:nth-child(1) {
  background-color: #f95e57;
}

.code-header span:nth-child(2) {
  background-color: #fbbc2f;
}

.code-header span:nth-child(3) {
  background-color: #30c83f;
}

.code-container pre {
  margin: 0;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.github-dark {
  background-color: #282c34;
  color: #abb2bf;
}

.github {
  background-color: #f6f8fa;
  color: #24292e;
}

@media (max-width: 768px) {
  .code-imagery {
    padding: 10px;
    width: 100%;
  }

  .page-title {
    font-size: 1.2rem;
  }

  .controls {
    flex-direction: column;
  }

  .el-select {
    width: 100%;
  }

  .generate-btn {
    width: 100%;
  }

  .code-container {
    font-size: 12px;
  }
}
</style>