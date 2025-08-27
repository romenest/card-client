<template>
  <div class="game-room-container">
    <div class="chat-area">
      <div class="chat-header">
        <h2>
          채널 {{ currentRoomId }}
          <span class="user-count" @click="toggleUserList">
            ({{ userCount }}/300) <i class="fa-solid fa-circle-info"></i>
          </span>
        </h2>
      </div>
      <div ref="messageListContainer" class="message-list-container">
        <ul class="message-list">
          <template v-for="(msg, index) in receivedMessages" :key="index">
            <div v-if="shouldShowDate(msg, index)" class="date-separator">
              {{ formatDate(msg.timestamp) }}
            </div>
            <li class="message-item">
              <template v-if="msg.sender === 'system'">
                <div class="system-message">{{ msg.message }}</div>
              </template>

              <template v-else>
                <div v-if="shouldShowSenderName(msg, index)" class="sender-name">
                  {{ msg.sender }}
                </div>
                <div class="message-line" :class="{'my-message-line': msg.sender === senderName}">
                  <span v-if="msg.sender === senderName && shouldShowTimestamp(msg, index)" class="timestamp my-timestamp">
                    {{ formatTimestamp(msg.timestamp) }}
                  </span>
                  <div :class="messageClass(msg)">
                    {{ msg.message }}
                  </div>
                  <span v-if="msg.sender !== senderName && shouldShowTimestamp(msg, index)" class="timestamp other-timestamp">
                    {{ formatTimestamp(msg.timestamp) }}
                  </span>
                </div>
              </template>
            </li>
          </template>
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
    
    <div v-if="isUserListVisible" class="user-list-popup">
      <div class="popup-header">
        <h3>채널 {{ currentRoomId }} 참가자</h3>
        <button @click="toggleUserList">닫기</button>
      </div>
      <ul class="user-list">
        <li v-for="user in usersInRoom" :key="user">{{ user }}</li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits, watch, nextTick } from 'vue';

const props = defineProps(['currentRoomId', 'receivedMessages', 'senderName']);
const emits = defineEmits(['send-message']);

const message = ref('');
const messageListContainer = ref(null);
const userCount = ref(0);
const usersInRoom = ref([]);
const isUserListVisible = ref(false);

const sendMessage = () => {
  if (message.value.trim() !== '') {
    emits('send-message', message.value.trim());
    message.value = '';
    
    // 메시지를 보내고, DOM 업데이트 후 아주 짧은 지연을 줍니다.
    nextTick(() => {
      setTimeout(() => {
        const container = messageListContainer.value;
        if (container) {
          container.scrollTop = container.scrollHeight;
        }
      }, 50); // 50ms (0.05초) 지연
    });
  }
};

const messageClass = (msg) => {
  if (msg.sender === props.senderName) {
    return 'my-message';
  } else {
    return 'other-message';
  }
};

const shouldShowSenderName = (msg, index) => {
  if (msg.sender === 'system' || msg.sender === props.senderName) {
    return false;
  }
  if (index === 0) {
    return true;
  }
  const prevMsg = props.receivedMessages[index - 1];
  const prevDate = new Date(prevMsg.timestamp);
  const currDate = new Date(msg.timestamp);
  return prevMsg.sender !== msg.sender || prevDate.getMinutes() !== currDate.getMinutes();
};

const shouldShowTimestamp = (msg, index) => {
  if (msg.sender === 'system') {
    return false;
  }
  if (index === props.receivedMessages.length - 1) {
    return true;
  }
  const nextMsg = props.receivedMessages[index + 1];
  const nextDate = new Date(nextMsg.timestamp);
  const currDate = new Date(msg.timestamp);
  return nextMsg.sender !== msg.sender || nextDate.getMinutes() !== currDate.getMinutes();
};

const shouldShowDate = (msg, index) => {
  if (index === 0) {
    return true;
  }
  const prevMsg = props.receivedMessages[index - 1];
  const prevDate = new Date(prevMsg.timestamp);
  const currDate = new Date(msg.timestamp);
  return prevDate.getDate() !== currDate.getDate();
};

const formatTimestamp = (timestamp) => {
  const date = new Date(timestamp);
  const hours = date.getHours().toString().padStart(2, '0');
  const minutes = date.getMinutes().toString().padStart(2, '0');
  return `${hours}:${minutes}`;
};

const formatDate = (timestamp) => {
  const date = new Date(timestamp);
  const year = date.getFullYear();
  const month = date.getMonth() + 1;
  const day = date.getDate();
  return `${year}년 ${month}월 ${day}일`;
};

const toggleUserList = () => {
  isUserListVisible.value = !isUserListVisible.value;
};

// watch 함수는 이제 사용자 수 업데이트 로직만 담당합니다.
watch(() => props.receivedMessages, (newMessages) => {
  const lastMessage = newMessages[newMessages.length - 1];
  if (lastMessage && lastMessage.sender === 'system' && lastMessage.userCount !== undefined) {
    userCount.value = lastMessage.userCount;
    usersInRoom.value = lastMessage.usersInRoom;
  }
}, { deep: true });
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
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.chat-header h2 {
  margin: 0;
  display: flex;
  align-items: center;
  gap: 10px;
}
.user-count {
  font-size: 0.8em;
  color: #555;
  cursor: pointer;
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
.message-item {
  display: flex;
  flex-direction: column;
  margin-bottom: 10px;
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
.sender-name {
  font-weight: bold;
  color: #333;
  font-size: 0.9em;
  margin-bottom: 5px;
  margin-left: 5px;
}
.timestamp {
  font-size: 0.8em;
  color: #999;
  margin: auto 5px;
}
.system-message {
  text-align: center;
  font-style: italic;
  color: #888;
  border-bottom: 1px solid #eee;
  padding: 10px 0;
  white-space: pre-wrap;
  margin-bottom: 10px;
}
.message-line {
  display: flex;
  align-items: flex-end;
  gap: 5px;
}
.my-message-line {
  justify-content: flex-end;
}
.my-message {
  background-color: #e0f7fa;
  border-radius: 5px;
  padding: 5px 10px;
  max-width: 80%;
  word-wrap: break-word;
  white-space: pre-wrap;
}
.other-message {
  background-color: #f1f1f1;
  border-radius: 5px;
  padding: 5px 10px;
  max-width: 80%;
  word-wrap: break-word;
  white-space: pre-wrap;
}
.my-timestamp {
  order: -1;
}
/* 사용자 목록 팝업 스타일 */
.user-list-popup {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 10px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  width: 300px;
  max-height: 400px;
  padding: 20px;
  display: flex;
  flex-direction: column;
}
.popup-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #eee;
  padding-bottom: 10px;
}
.user-list {
  list-style-type: none;
  padding: 0;
  margin: 10px 0 0;
  overflow-y: auto;
}
.user-list li {
  padding: 5px 0;
  border-bottom: 1px solid #f0f0f0;
}
.user-list li:last-child {
  border-bottom: none;
}
.date-separator {
  text-align: center;
  font-size: 0.8em;
  color: #a0a0a0;
  margin: 15px 0;
}
</style>