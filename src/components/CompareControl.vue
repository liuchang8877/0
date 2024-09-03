<template>
  <div>
    <br />
    <div>
      浏览器录音听写：
      <button @click="startRecording" id="btn_control02">开始录音</button>
    </div>
    <br />
    <div id="result">{{ result }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const APPID = 'e744e845';
const API_SECRET = 'NzNmYzZkNDE0ZjJmNTZjNjQyMGM2ZWI5';
const API_KEY = '472fb85982b85765712ea7cefc0b7f95';

const result = ref('');

// 定义需要引入的脚本路径
const scripts = [
  '../../iat-js-demo/example/crypto-js.js',
  '../../iat-js-demo/dist/index.umd.js',
  '../../iat-js-demo/example/iat/index.js',
];

// 动态加载脚本的函数
const loadScripts = async () => {
  for (const src of scripts) {
    await new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.src = src;
      script.onload = resolve;
      script.onerror = reject;
      document.body.appendChild(script);
    }).catch((error) => {
      console.error(`加载脚本失败: ${src}`, error);
    });
  }
};

onMounted(() => {
  // 在组件挂载时加载所有脚本
  loadScripts().then(() => {
    console.log('所有脚本加载完成');
    // 这里可以执行依赖这些脚本的初始化逻辑
  });
  // 设置全局变量
  window.APPID = APPID;
  window.API_SECRET = API_SECRET;
  window.API_KEY = API_KEY;
});

const startRecording = () => {
  // 实现开始录音的逻辑
  console.log('开始录音');
  // 在此处添加录音相关的代码
};
</script>

<style scoped>
/* 你可以在这里添加组件的样式 */
</style>