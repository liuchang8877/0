// 主应用组件 App.vue
<template>
  <div class="chat-app">
    <ChatInterface />
     <!-- <CompareControl/> -->
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import ChatInterface from './components/VoiceControl.vue';
import CompareControl from './components/CompareControl.vue'
export default defineComponent({
  name: 'App',
  components: {
    ChatInterface
    // CompareControl
  }
});
</script>