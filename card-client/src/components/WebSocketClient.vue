<template>
<div>
    <h1>WebSocket 테스트</h1>
    <input v-model="message" @keyup.enter="sendMessage" placeholder="메시지를 입력하세요" />
    <button @click="sendMessage">전송</button>
    <div v-if="receivedMessages.length">
    <h2>수신 메시지:</h2>
    <ul>
        <li v-for="(msg, index) in receivedMessages" :key="index">{{ msg }}</li>
    </ul>
    </div>
</div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const message = ref('');
const receivedMessages = ref([]);
// 웹소켓 서버의 주소로 연결합니다.
// Spring Boot의 기본 포트 8080을 사용하고, 웹소켓 매핑 경로인 /ws/game을 추가합니다.
const ws = new WebSocket('ws://localhost:8080/ws/game');

// 컴포넌트가 화면에 나타날 때 실행되는 훅
onMounted(() => {
// 웹소켓 연결이 성공적으로 이루어졌을 때
ws.onopen = () => {
    console.log('웹소켓 연결 성공!');
};

// 서버로부터 메시지를 받았을 때
ws.onmessage = (event) => {
    console.log('서버로부터 메시지 수신:', event.data);
    receivedMessages.value.push(event.data);
};

// 웹소켓 연결이 닫혔을 때
ws.onclose = () => {
    console.log('웹소켓 연결 종료!');
};

// 웹소켓 연결 중 에러가 발생했을 때
ws.onerror = (error) => {
    console.error('웹소켓 에러 발생:', error);
};
});

// 컴포넌트가 화면에서 사라질 때 실행되는 훅
onUnmounted(() => {
// 컴포넌트가 언마운트되면 웹소켓 연결을 정리합니다.
ws.close();
});

// 입력한 메시지를 서버로 전송하는 함수
const sendMessage = () => {
if (message.value.trim() !== '') {
    ws.send(message.value);
    message.value = ''; // 메시지 전송 후 입력 필드를 비웁니다.
}
};
</script>

<style scoped>
/* 여기에 스타일을 추가하여 컴포넌트를 꾸밀 수 있습니다. */
</style>