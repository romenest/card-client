<template>
  <div class="game-room-container">
    <div class="chat-area">
      <div class="chat-header">
        <h2>채널 {{ currentRoomId }}</h2>
      </div>
      <div class="message-list-container">
        <ul class="message-list">
          <li v-for="(msg, index) in receivedMessages" :key="index" :class="messageClass(msg)">
            <template v-if="msg.sender === 'system'">
              {{ msg.message }}
            </template>
            <template v-else>
              <strong>{{ msg.sender }}</strong>: {{ msg.message }}
            </template>
          </li>
        </ul>
      </div>
      <div class="message-input">
        <span class="nickname-prefix">[{{ senderName }}]</span>
        <input v-model="message" @keyup.enter="sendMessage" placeholder="메시지를 입력하세요" />
        <button @click="sendMessage">전송</button>
      </div>
    </div>
    <div class="game-area">
      <h1>게임 영역</h1>
      <p>여기에 카드 게임 UI가 구현될 예정입니다.</p>
    </div>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from 'vue';

const props = defineProps(['currentRoomId', 'receivedMessages', 'senderName']);
const emits = defineEmits(['send-message']);

const message = ref('');

const sendMessage = () => {
  if (message.value.trim() !== '') {
    emits('send-message', message.value.trim());
    message.value = '';
  }
};

// 메시지 보낸 사람에 따라 클래스를 반환하는 함수
const messageClass = (msg) => {
  if (msg.sender === 'system') {
    return 'system-message';
  } else if (msg.sender === props.senderName) {
    return 'my-message'; // 내 메시지
  } else {
    return 'other-message'; // 다른 사람의 메시지
  }
};
</script>

<style scoped>
.game-room-container {
  display: flex;
  width: 100%;
  height: 100vh;
  padding: 20px;
  gap: 20px;
  box-sizing: border-box;
}
.chat-area {
  width: 350px;
  height: 100%;
  border: 1px solid #ccc;
  border-radius: 10px;
  background-color: #fff;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
}
.chat-header {
  padding: 10px 20px;
  border-bottom: 1px solid #eee;
}
.message-list-container {
  flex-grow: 1;
  overflow-y: auto;
  padding: 10px 20px;
}
.message-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
}
.message-list li {
  padding: 5px 0;
}
.message-input {
  display: flex;
  gap: 10px;
  padding: 10px 20px;
  border-top: 1px solid #eee;
}
.nickname-prefix {
  font-weight: bold;
  color: #007bff;
  white-space: nowrap;
  display: flex;
  align-items: center;
}
.message-input input {
  flex-grow: 1;
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 8px;
  outline: none;
}
.game-area {
  flex-grow: 1;
  border: 1px solid #ccc;
  border-radius: 10px;
  background-color: #fff;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  padding: 20px;
}
/* 메시지 스타일 */
.system-message {
  text-align: center;
  font-style: italic;
  color: #888;
  border-bottom: 1px solid #eee;
  padding: 10px 0;
}
.my-message {
  text-align: right;
  background-color: #e0f7fa; /* 내 메시지 배경색 */
  border-radius: 5px;
  padding: 5px 10px;
  margin-left: auto; /* 오른쪽 정렬 */
  max-width: 80%; /* 너무 길어지지 않게 */
}
.other-message {
  text-align: left;
  background-color: #f1f1f1; /* 다른 사람 메시지 배경색 */
  border-radius: 5px;
  padding: 5px 10px;
  margin-right: auto;
  max-width: 80%;
}
</style>