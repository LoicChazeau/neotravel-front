<template>
  <h2 class="chatbot-title">Notre ChatBot vous permet de créer un devis en moins de 5 minutes ou de répondre à vos questions !</h2>
  <div class="chatbot-container">
    <div class="chatbot-background">
      <!-- <ChatbotHeader />
      <ChatbotBody /> -->
      <button @click="test()">test</button>
    </div>
  </div>
</template>

<script>
import ChatbotHeader from "./components/ChatbotHeader.vue";
import ChatbotBody from "./components/ChatbotBody.vue";

export default {
  components: {
    ChatbotHeader,
    ChatbotBody,
  },
  methods: {
    test() {
      console.log("test")
    }
  }
};
</script>

<style scoped>
.chatbot-title {
  text-align: center;
  margin: 0;
  margin-top: 5px;
  font-size: 18px;
  line-height: 28px;
}
.chatbot-container {
  display: flex;
  justify-content: center;
  height: 100%;
  flex: 1;
  padding: 30px 0;
}
.chatbot-background {
  display: flex;
  flex-direction: column;
  height: 476px;
  width: 80%;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0px 2px 10px 0px rgb(0 0 0 / 30%);
}
</style>
