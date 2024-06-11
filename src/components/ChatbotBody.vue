<template>
  <div class="chatbot-body">
    <div class="chatbot-messages">
      <div v-for="message in messages" :key="message.id" class="chatbot-message">
        {{ message.text }}
      </div>
    </div>
    <div class="chatbot-input">
      <input type="text" placeholder="Une question ?" v-model="userInput" @keyup.enter="sendMessage" />
      <button @click="sendMessage">Envoyer</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const messages = ref([
  { id: 1, text: 'Bonjour ! Comment puis-je vous aider ?' },
  // Ajoutez d'autres messages ici
]);

const userInput = ref('');

const sendMessage = () => {
  if (userInput.value.trim()) {
    messages.value.push({ id: messages.value.length + 1, text: userInput.value });
    userInput.value = '';
  }
};
</script>

<style scoped>
.chatbot-body {
  display: flex;
  flex-direction: column;
  padding: 10px;
}
.chatbot-messages {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 10px;
}
.chatbot-message {
  margin: 10px 0;
  padding: 10px;
  background-color: #f1f1f1;
  border-radius: 5px;
}
.chatbot-input {
  display: flex;
  align-items: center;
}
.chatbot-input input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-right: 10px;
}
.chatbot-input button {
  padding: 10px 20px;
  border: none;
  background-color: #4CAF50;
  color: white;
  border-radius: 5px;
  cursor: pointer;
}
</style>
