<template>
  <div class="chatbot-header">
    <img src="../assets/logo.svg" alt="Chatbot Logo" class="chatbot-logo" />
    <span class="chatbot-name">NeotravelBot</span>
  </div>
</template>

<script setup></script>

<style scoped>
.chatbot-header {
  display: flex;
  align-items: center;
  padding: 8px;
  background-color: #4e38cc;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
}
.chatbot-logo {
  height: 40px;
  margin-right: 20px;
  margin-left: 15px;
}
.chatbot-name {
  color: white;
  font-size: 16px;
  font-weight: 500;
}
</style>
