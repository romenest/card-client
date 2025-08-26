<template>
  <div>
    <h1>WebSocket 테스트</h1>
    <input v-model="message" @keyup.enter="sendMessage" placeholder="메시지를 입력하세요" />
    <button @click="sendMessage">전송</button>
    <div v-if="receivedMessages.length">
      <h2>채팅방</h2>
      <ul>
        <li v-for="(msg, index) in receivedMessages" :key="index">
          <strong>{{ msg.sender }}</strong>: {{ msg.message }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const message = ref('');
const receivedMessages = ref([]);
const senderName = ref('User-' + Math.floor(Math.random() * 1000));
let ws = null; // 웹소켓 객체

const connectWebSocket = () => {
    if (ws && (ws.readyState === WebSocket.OPEN || ws.readyState === WebSocket.CONNECTING || ws.readyState === WebSocket.CLOSING)) {
        return;
    }
    ws = new WebSocket('ws://192.168.2.10:8080/ws/game');

    ws.onopen = () => {
        console.log('웹소켓 연결 성공!');
    };

    ws.onmessage = (event) => {
        try {
            // 서버로부터 받은 JSON 문자열을 객체로 변환합니다.
            const chatMessage = JSON.parse(event.data);
            console.log('서버로부터 메시지 수신:', chatMessage);
            receivedMessages.value.push(chatMessage);
        } catch (e) {
            console.error('메시지 파싱 오류:', e);
        }
    };

    ws.onclose = (event) => {
        console.log('웹소켓 연결 종료!', event.code, event.reason);
        if (!event.wasClean) {
            setTimeout(connectWebSocket, 3000);
        }
    };

    ws.onerror = (error) => {
        console.error('웹소켓 에러 발생:', error);
        ws.close();
    };
};

onMounted(() => {
  connectWebSocket();
});

onUnmounted(() => {
  if (ws && ws.readyState === WebSocket.OPEN) {
    ws.close();
  }
});

const sendMessage = () => {
  if (message.value.trim() !== '' && ws && ws.readyState === WebSocket.OPEN) {
    const chatMessage = {
      sender: senderName.value,
      message: message.value,
      timestamp: Date.now()
    };
    const jsonMessage = JSON.stringify(chatMessage);
    ws.send(jsonMessage);
    message.value = '';
  }
};
</script>

<style scoped>
/* 기존 스타일 */
</style>