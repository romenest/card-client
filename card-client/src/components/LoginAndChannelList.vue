<template>
  <div class="center-box">
    <div v-if="!isNicknameSet" class="input-area">
      <input
        v-model="inputNickname"
        @keyup.enter="setNickname"
        placeholder="사용할 닉네임을 입력하세요"
        class="nickname-input"
      />
      <button @click="setNickname" class="action-button">확인</button>
    </div>

    <div v-else>
      <div class="nickname-display">
        사용할 닉네임: <strong>{{ senderName }}</strong>
      </div>
      <div class="channel-list-container">
        <div class="channel-list">
          <button
            v-for="channel in channels"
            :key="channel"
            @click="joinChannel(channel)"
            class="channel-button"
          >
            채널 {{ channel }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, defineEmits } from 'vue';

const emits = defineEmits(['join-channel']); // 부모에게 이벤트를 보낼 때 사용

const inputNickname = ref('');
const senderName = ref('');
const isNicknameSet = ref(false);
const channels = [1, 2, 3, 4];

const setNickname = () => {
  if (inputNickname.value.trim() !== '') {
    senderName.value = inputNickname.value.trim();
    isNicknameSet.value = true;
  }
};

const joinChannel = (channel) => {
  // 닉네임과 채널 ID를 부모 컴포넌트로 전달
  emits('join-channel', { nickname: senderName.value, channelId: channel });
};
</script>

<style scoped>
.center-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 30px;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  width: 90%;
  max-width: 500px;
}
.input-area {
  display: flex;
  gap: 10px;
  border: 2px solid red;
  border-radius: 5px;
  padding: 5px;
  width: 100%;
}
.nickname-input {
  flex-grow: 1;
  border: none;
  outline: none;
}
.action-button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 8px 15px;
  cursor: pointer;
  border-radius: 5px;
}
.nickname-display {
  font-size: 1.2em;
  font-weight: bold;
}
.channel-list-container {
  border: 2px solid blue;
  padding: 20px;
  border-radius: 10px;
  width: 100%;
}
.channel-list {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}
.channel-button {
  background-color: #28a745;
  color: white;
  border: none;
  padding: 20px;
  font-size: 1.2em;
  cursor: pointer;
  border-radius: 5px;
}
</style>