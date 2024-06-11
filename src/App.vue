<template>
  <h2 class="chatbot-title">Notre ChatBot vous permet de créer un devis en moins de 5 minutes ou de répondre à vos questions !</h2>
  <div class="chatbot-container">
    <div class="chatbot-background">
      <ChatbotHeader />
      <ChatbotBody />
    </div>
  </div>
</template>

<script setup>
import ChatbotHeader from "./components/ChatbotHeader.vue";
import ChatbotBody from "./components/ChatbotBody.vue";
</script>

<style scoped>
.chatbot-title {
  text-align: center;
  margin: 0;
  margin-top: 10px;
  font-size: 18px;
  line-height: 28px;
}
.chatbot-container {
  display: flex;
  justify-content: center;
  flex: 1;
  padding: 40px 0;
  overflow-y: auto;
}
.chatbot-background {
  height: 100%;
  width: 75%;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0px 2px 10px 0px rgb(0 0 0 / 30%);
}
</style>
