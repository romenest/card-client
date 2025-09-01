<template>
  <div class="main-container">
    <LoginAndChannelList v-if="!isJoined" @join-channel="handleJoin" />
    <ChatRoom
      v-else
      :currentRoomId="currentRoomId"
      :receivedMessages="receivedMessages"
      :senderName="senderName"
      @send-message="handleSendMessage"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import LoginAndChannelList from './components/LoginAndChannelList.vue';
import ChatRoom from './components/ChatRoom.vue';

const isJoined = ref(false);
const senderName = ref('');
const currentRoomId = ref(null);
const receivedMessages = ref([]);
let ws = null;

const handleJoin = (payload) => {
  senderName.value = payload.nickname;
  currentRoomId.value = payload.channelId;
  isJoined.value = true;
  connectWebSocket();
};

const handleSendMessage = (message) => {
  if (ws && ws.readyState === WebSocket.OPEN) {
    const chatMessage = {
      type: "chat",
      sender: senderName.value,
      message: message,
      roomId: currentRoomId.value,
      timestamp: Date.now()
    };
    ws.send(JSON.stringify(chatMessage));
  }
};

const connectWebSocket = () => {
  if (ws && ws.readyState === WebSocket.OPEN) {
    return;
  }
  ws = new WebSocket('ws://localhost:8080/ws/game');

  ws.onopen = () => {
    console.log('웹소켓 연결 성공!');
    const joinMessage = {
      type: "system",
      sender: senderName.value,
      message: `[${senderName.value}님이 채널 ${currentRoomId.value}에 입장했습니다.]`,
      roomId: currentRoomId.value,
      timestamp: Date.now()
    };
    ws.send(JSON.stringify(joinMessage));
  };

  ws.onmessage = (event) => {
    try {
      const chatMessage = JSON.parse(event.data);
      console.log('서버로부터 메시지 수신:', chatMessage);
      // 서버로부터 받은 메시지를 배열에 추가
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
    if (ws && ws.readyState !== WebSocket.CLOSED) {
      ws.close();
    }
  };
};

onUnmounted(() => {
  if (ws && ws.readyState === WebSocket.OPEN) {
    ws.close();
  }
});
</script>

<style>
.main-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f0f2f5;
  font-family: Arial, sans-serif;
}
</style>