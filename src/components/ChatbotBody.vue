<template>
  <div class="chatbot-body">
    <div class="chatbot-messages">
      <div class="chatbot-message">
        test
      </div>
    </div>
    <div class="chatbot-input-container">
      <input class="chatbot-input" type="text" placeholder="Une question ?"/>
      <img class="chatbot-input-btn" :src="sendUrl" alt="Send button" />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      sendUrl: import.meta.env.VITE_SEND_URL,
    };
  },
};
</script>

<style scoped>
.chatbot-body {
  display: flex;
  flex-direction: column;
  flex: 1;
  height: 400px;
  padding: 10px 20px;
}
.chatbot-messages {
  flex: 1;
  overflow-y: auto;
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
  padding-top: 10px;
}
.chatbot-input {
  flex: 1;
  padding: 10px;
  border: 1px solid #e6e6e6;
  background-color: #fafafa;
  border-radius: 10px;
  margin-right: 10px;
  font-size: 13px;
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
