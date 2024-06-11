<template>
  <div class="chatbot-body">
    <div class="chatbot-messages">
      <div v-for="message in messages" :key="message.id" class="chatbot-message">
        {{ message.text }}
      </div>
    </div>
    <div class="chatbot-input-container">
      <input class="chatbot-input" type="text" placeholder="Une question ?" v-model="userInput" @keyup.enter="sendMessage" />
      <img class="chatbot-input-btn" @click="sendMessage" src="../assets/send.svg" alt="Send button" />
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const messages = ref([
  { id: 1, text: "Bonjour ! Comment puis-je vous aider ?" },
  // Ajoutez d'autres messages ici
]);

const userInput = ref("");

const sendMessage = () => {
  if (userInput.value.trim()) {
    messages.value.push({ id: messages.value.length + 1, text: userInput.value });
    userInput.value = "";
  }
};
</script>

<style scoped>
.chatbot-body {
  display: flex;
  flex-direction: column;
  padding: 10px 20px;
}
.chatbot-messages {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 10px;
}
.chatbot-message {
  margin: 10px 0;
  padding: 10px;
  background-color: #f5f5f5;
  border-radius: 10px;
}
.chatbot-input-container {
  display: flex;
  align-items: center;
}
.chatbot-input {
  flex: 1;
  padding: 10px;
  border: 1px solid #e6e6e6;
  background-color: #fafafa;
  border-radius: 10px;
  margin-right: 10px;
}
.chatbot-input:focus-visible {
  outline: #4e38cc auto 1px;
}
.chatbot-input-btn {
  cursor: pointer;
  width: 40px;
  margin-left: 3px;
}
</style>
