<template>
  <div class="center-box">
    <div v-if="!isNicknameSet">
      <div class="input-area">
        <input
          v-model="inputNickname"
          @keyup.enter="setNickname"
          placeholder="사용할 닉네임을 입력하세요"
          class="nickname-input"
        />
        <button @click="setNickname" class="action-button">확인</button>
      </div>
      <div v-if="nicknameError" class="error-message">
        {{ nicknameError }}
      </div>
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

const emits = defineEmits(['join-channel']);

const inputNickname = ref('');
const senderName = ref('');
const isNicknameSet = ref(false);
const nicknameError = ref('');
const channels = [1, 2, 3, 4];

// 닉네임 유효성 검사 함수
const validateNickname = (nickname) => {
  if (!nickname) {
    return '닉네임을 입력해주세요.';
  }
  const hangulRegex = /[가-힣]/;
  const isHangul = hangulRegex.test(nickname);
  const maxLength = isHangul ? 5 : 8;
  if (nickname.length > maxLength) {
    return `닉네임은 ${isHangul ? '한글 5자' : '영문 8자'} 이내로 입력해주세요.`;
  }
  return null; // 유효성 검사 통과
};

// 닉네임을 설정하고 채널 목록 화면을 표시하는 함수
const setNickname = () => {
  nicknameError.value = validateNickname(inputNickname.value.trim());
  if (!nicknameError.value) {
    senderName.value = inputNickname.value.trim();
    isNicknameSet.value = true;
  }
};

// 채널에 입장하는 함수
const joinChannel = (channel) => {
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