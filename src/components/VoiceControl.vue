
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
        <button id="btn_control02">开始录音</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, nextTick, onBeforeUnmount } from 'vue';

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
    const messages = ref<Message[]>([]);
    const inputMessage = ref('');
    const chatMessages = ref(null);
    const recordingButtonText = ref('语音');
    let iatRecorder: any = null;

    const APPID = "e744e845";
    const API_SECRET = "NzNmYzZkNDE0ZjJmNTZjNjQyMGM2ZWI5";
    const API_KEY = "472fb85982b85765712ea7cefc0b7f95";
    const YOUR_SSE_ENDPOINT = "https://zhiyuejiayuan.id5.cn:10443/prod-api/applet/beida/chat";

    onMounted(() => {
      const scripts = [
        '../../iat-js-demo/example/crypto-js.js',
        '../../iat-js-demo/dist/index.umd.js',
        '../../iat-js-demo/example/iat/index.js',
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
      };

      loadScriptsSequentially().catch(error => {
        console.error('Failed to load scripts:', error);
      });

      // 设置全局变量
      window.APPID = APPID;
      window.API_SECRET = API_SECRET;
      window.API_KEY = API_KEY;

      addMessage('欢迎来到聊天室！', 'bot');

      // 监听 id="result" 的变化
      const resultElement = document.getElementById('result');
      if (resultElement) {
        const observer = new MutationObserver((mutationsList) => {
          mutationsList.forEach((mutation) => {
            if (mutation.type === 'childList' || mutation.type === 'characterData') {
              inputMessage.value = resultElement.textContent || '';
            }
          });
        });

        observer.observe(resultElement, { childList: true, subtree: true, characterData: true });
      }

      // 初始化 SSE 连接
      const eventSource = new EventSource(YOUR_SSE_ENDPOINT); // 替换为你的 SSE 接口地址
      eventSource.addEventListener('message', (event) => {
        try {
          const data = JSON.parse(event.data);
          handleSSEData(data);
        } catch (error) {
          console.error('Failed to parse SSE data:', error);
        }
      });

      onBeforeUnmount(() => {
        eventSource.close();
      });
    });

    // 使用 Fetch 发送消息到服务器
    function sendMessageToServer(message: string) {

      var myHeaders = new Headers();
      // myHeaders.append("User-Agent", "Apifox/1.0.0 (https://apifox.com)");
      myHeaders.append("Content-Type", "application/json");
      myHeaders.append("Accept", "*/*");
      // myHeaders.append("Host", "210.12.135.100:10443");
      myHeaders.append("Connection", "keep-alive");

      var raw = JSON.stringify({
        "content": message
      });

      var requestOptions = {
        method: 'POST',
        headers: myHeaders,
        body: raw,
        redirect: 'follow'
      };

      fetch(YOUR_SSE_ENDPOINT, requestOptions)
        .then(response => response.text())
        .then(result => {

          // 解析每一行数据
          const data = result.trim().split('\n').filter(line => line.startsWith('data:')).map(line => JSON.parse(line.replace('data: ', '')));
          //data.forEach(item => handleSSEData(item)); // 处理每条数据
          // 找到最后一条数据并处理
          const lastItem = data[data.length - 1]; // 取最后一条
          handleSSEData(lastItem)
          //console.log(result);

        })
        .catch(error => console.log('error', error));
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

        //发送服务器
        sendMessageToServer(inputMessage.value);

        // setTimeout(() => {
        //   addMessage('这是一个自动回复: ' + inputMessage.value, 'bot');
        // }, 1000);
        inputMessage.value = '';
      }
    };

    const addMessage = (text: string, type: 'user' | 'bot') => {
      messages.value.push({ text, type });
      nextTick(() => {
        if (chatMessages.value) {
          chatMessages.value.scrollTop = chatMessages.value.scrollHeight;
        }
      });
    };

    const handleSSEData = (response: any) => {
      // 打印原始响应，查看类型和内容
      console.log("handleSSEData - response:", response);

      // 如果 `response` 是字符串，则尝试解析 JSON
      let data;
      if (typeof response === 'string') {
        try {
          data = JSON.parse(response); // 解析 JSON 字符串
        } catch (error) {
          console.error('Failed to parse JSON:', error);
          return;
        }
      } else {
        // 如果 `response` 已经是对象，直接使用
        data = response;
      }

      switch (data.event) {
        case 'request_accepted':
          addMessage(`请求已接受：${data.time}`, 'bot');
          break;
        case 'generate_search_queries':
          addMessage(`生成的搜索查询: ${data.data.join(', ')}`, 'bot');
          break;
        case 'on_text':
          addMessage(data.data, 'bot');
          break;
        case 'refer_docs':
          const docs = data.data.map((doc: any) => `标题: ${doc.title}, 来源: ${doc.source}`).join('\n');
          addMessage(`参考文档:\n${docs}`, 'bot');
          break;
        case 'chat_reply':
          handleChatReply(data.data);
          break;
        case 'full_chat_reply':
          addMessage(data.data, 'bot');
          break;
        default:
          console.log('未知事件类型:', data.event);
      }
    };

    let chatReplyBuffer = '';

    const handleChatReply = (text: string) => {
      chatReplyBuffer += text;
      addMessage(chatReplyBuffer, 'bot');
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