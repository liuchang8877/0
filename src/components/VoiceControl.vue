<template>
  <div class="chat-app">
    <header class="chat-header">
      <div class="header-left">
        <button class="menu-button">☰</button>
        <img src="../assets/robot-add.png" alt="logo" class="logo" />
        <span class="app-name">chatgpt</span>
        <div id="result">语音信息：</div>

      </div>
    </header>

    <div class="chat-interface">
      <div class="chat-messages" ref="chatMessages">
        <div v-for="(message, index) in messages" :key="index" :class="['message', message.type]">
          {{ message.text }}
        </div>
      </div>
      <div class="chat-input">
        <input v-model="inputMessage" @keyup.enter="sendMessage" placeholder="输入消息..." />
        <button @click="sendMessage">发送</button>
        <!-- <button @click="toggleRecording">{{ recordingButtonText }}</button> -->
        <button id="btn_control02">开始录音</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">

import { defineComponent, ref, onMounted, nextTick } from 'vue';

interface Message {
  text: string;
  type: 'user' | 'bot';
}

declare global {
  interface Window {
    IatRecorder: any;
    handleFileInput: (event: Event) => void;
    CryptoJS: any;
    APPID: string;
    API_SECRET: string;
    API_KEY: string;
  }
}

export default defineComponent({
  name: 'VoiceControl',
  setup() {
    const messages = ref<Message[]>([]); // Specify the type of messages
    const inputMessage = ref('');
    const chatMessages = ref(null);
    const recordingButtonText = ref('语音');
    let iatRecorder: any = null;

    const APPID = "e744e845";
    const API_SECRET = "NzNmYzZkNDE0ZjJmNTZjNjQyMGM2ZWI5";
    const API_KEY = "472fb85982b85765712ea7cefc0b7f95";

    onMounted(() => {
      const scripts = [
        '../../iat-js-demo/example/crypto-js.js',
        '../../iat-js-demo/dist/index.umd.js',
        '../../iat-js-demo/example/iat/index.js',
        // '../../iat-js-demo/example/iat/input-file.js'
      ];

      const loadScript = (src: string): Promise<void> => {
        return new Promise((resolve, reject) => {
          const script = document.createElement('script');
          script.src = src;
          script.async = false;
          script.onload = () => resolve();
          script.onerror = () => reject(new Error(`Failed to load script: ${src}`));
          document.body.appendChild(script);
        });
      };

      const loadScriptsSequentially = async () => {
        for (const src of scripts) {
          await loadScript(src);
        }
        // initializeIatRecorder();
      };

      loadScriptsSequentially().catch(error => {
        console.error('Failed to load scripts:', error);
      });

      // 设置全局变量
      window.APPID = APPID;
      window.API_SECRET = API_SECRET;
      window.API_KEY = API_KEY;

      addMessage('欢迎来到聊天室！', 'bot');
      // 获取 id="result" 的内容并赋值给 inputMessage
      // 监听 id="result" 的变化
      const resultElement = document.getElementById('result');
      if (resultElement) {
        // 创建一个 MutationObserver 实例
        const observer = new MutationObserver((mutationsList) => {
          mutationsList.forEach((mutation) => {
            if (mutation.type === 'childList' || mutation.type === 'characterData') {
              // 更新 inputMessage 值
              inputMessage.value = resultElement.textContent || '';
            }
          });
        });

        // 配置要监听的变化类型
        observer.observe(resultElement, { childList: true, subtree: true, characterData: true });
      }

    });

    function initializeIatRecorder() {
      if (typeof window.IatRecorder === 'function') {
        iatRecorder = new window.IatRecorder({
          onResult: (text: string) => {
            inputMessage.value = text;
            if (inputMessage.value.trim()) {
              addMessage(inputMessage.value, 'user');
            }
          },
          onError: (error: any) => {
            console.error('识别错误:', error);
          }
        });
      } else {
        console.error('IatRecorder is not a constructor', window.IatRecorder);
      }
    }

    const toggleRecording = () => {
      if (recordingButtonText.value === '开始录音') {
        iatRecorder?.start();
        recordingButtonText.value = '停止录音';
      } else {
        iatRecorder?.stop();
        recordingButtonText.value = '开始录音';
      }
    };

    const sendMessage = () => {
      if (inputMessage.value.trim()) {
        addMessage(inputMessage.value, 'user');
        setTimeout(() => {
          addMessage('这是一个自动回复: ' + inputMessage.value, 'bot');
        }, 1000);
        inputMessage.value = '';
      }
    };

    const addMessage = (text: string, type: 'user' | 'bot') => {
      messages.value.push({ text, type });
      nextTick(() => {
        if (chatMessages.value) {
          // chatMessages.value.scrollTop = chatMessages.value.scrollHeight;
        }
      });
    };

    return {
      messages,
      inputMessage,
      chatMessages,
      recordingButtonText,
      toggleRecording,
      sendMessage,
    };
  }
});


</script>

<style scoped>
#result {
  display: none;
}

.chat-app {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: #f6f6f6;
}

.chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 20px;
  background-color: #ffffff;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.header-left {
  display: flex;
  align-items: center;
}

.menu-button,
.settings-button {
  background: none;
  border: none;
  font-size: 18px;
  cursor: pointer;
  margin-right: 10px;
}

.logo {
  width: 30px;
  height: 30px;
  margin-right: 10px;
}

.app-name {
  font-size: 18px;
  font-weight: bold;
}

.chat-interface {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
  background-color: #fff;
  border-radius: 8px 8px 0 0;
}

.chat-title {
  padding: 10px 20px;
  font-size: 16px;
  border-bottom: 1px solid #ddd;
  background-color: #ffffff;
}

.chat-messages {
  flex-grow: 1;
  overflow-y: auto;
  padding: 20px;
}

.message {
  max-width: 70%;
  padding: 10px;
  margin-bottom: 10px;
  border-radius: 20px;
  word-wrap: break-word;
}

.user {
  background-color: #d9f7be;
  align-self: flex-end;
  margin-left: auto;
}

.bot {
  background-color: #e6f7ff;
  align-self: flex-start;
}

.chat-input {
  display: flex;
  padding: 10px;
  border-top: 1px solid #ddd;
  background-color: #f6f6f6;
}

input {
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 20px;
  margin-right: 10px;
}

button {
  padding: 10px 15px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  margin-left: 5px;
}

button:hover {
  background-color: #45a049;
}
</style>