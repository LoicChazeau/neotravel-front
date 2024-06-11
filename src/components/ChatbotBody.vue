<template>
  <div class="chatbot-body">
    <div class="chatbot-messages" ref="messagesContainer">
      <div
        v-for="message in messages"
        :key="message.id"
        :class="{ 'chatbot-message bot': message.isBot, 'chatbot-message user': !message.isBot }"
      >
        <img v-if="message.isBot" class="bot-avatar" :src="logoUrl" alt="Bot Avatar" />
        <span v-html="message.text"></span>
      </div>
      <div v-if="showQuickActions && !isFormStep" class="quick-actions">
        <div class="quick-actions-btn" @click="handleQuickAction('Demande de devis')">Demande de devis</div>
        <div class="quick-actions-btn" @click="handleQuickAction('Parler à un commercial')">Parler à un commercial</div>
        <div class="quick-actions-btn" @click="handleQuickAction('J\'ai une question')">J'ai une question</div>
      </div>
      <div v-if="isFormStep" class="quick-actions">
        <div v-if="currentStep === 1">
          <div class="quick-actions-btn" @click="handleFormStep('Un particulier')">Un particulier</div>
          <div class="quick-actions-btn" @click="handleFormStep('Une entreprise')">Une entreprise</div>
          <div class="quick-actions-btn" @click="handleFormStep('Une association')">Une association</div>
          <div class="quick-actions-btn" @click="handleFormStep('Une collectivité')">Une collectivité</div>
        </div>
        <!-- Ajoutez d'autres étapes ici -->
      </div>
    </div>
    <div class="chatbot-input-container">
      <input class="chatbot-input" type="text" placeholder="Une question ?" v-model="userInput" @keyup.enter="sendMessage" />
      <img class="chatbot-input-btn" @click="sendMessage" :src="sendUrl" alt="Send button" />
    </div>
  </div>
</template>

<script>
import { nextTick } from "vue";

export default {
  data() {
    return {
      sendUrl: import.meta.env.VITE_SEND_URL,
      logoUrl: import.meta.env.VITE_LOGO_URL,
      messages: [{ id: 1, text: "Bonjour ! Comment puis-je vous aider ?", isBot: true }],
      userInput: "",
      showQuickActions: true,
      isFormStep: false,
      currentStep: 0,
      response: null,
      error: null,
    };
  },
  methods: {
    async sendMessage() {
      console.log("sendMessage called");
      if (this.userInput.trim()) {
        this.messages.push({ id: this.messages.length + 1, text: this.userInput, isBot: false });
        this.showQuickActions = false;
        this.scrollToBottom();

        try {
          const res = await fetch("http://10.76.203.217:5000/conversation", {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              description: this.userInput,
            }),
          });
          if (!res.ok) {
            throw new Error(`HTTP error! status: ${res.status}`);
          }
          const data = await res.json();
          this.response = data.response;
          console.log("response :", this.response);
          this.error = null;
        } catch (error) {
          this.error = error.toString();
          this.response = null;
        }
        this.userInput = "";
      }
    },
    handleQuickAction(action) {
      console.log(`Quick action selected: ${action}`);
      this.messages.push({ id: this.messages.length + 1, text: action, isBot: false });
      this.showQuickActions = false;
      this.scrollToBottom();
      if (action === "Demande de devis") {
        this.startForm();
      }
    },
    startForm() {
      this.messages.push({
        id: this.messages.length + 1,
        text: "Pour la création du devis nous avons besoin de quelques informations.",
        isBot: true,
      });
      this.scrollToBottom();
      setTimeout(() => {
        this.messages.push({ id: this.messages.length + 1, text: "Vous êtes :", isBot: true });
        this.scrollToBottom();
        this.currentStep = 1;
      }, 1500);
      this.isFormStep = true;
    },
    handleFormStep(selection) {
      console.log(`Form step selection: ${selection}`);
      this.messages.push({ id: this.messages.length + 1, text: selection, isBot: false });
      this.nextFormStep();
      this.scrollToBottom();
    },
    nextFormStep() {
      // Ajoutez ici la logique pour aller à l'étape suivante du formulaire
      this.currentStep++;
      // Exemple: pour l'étape 2
      if (this.currentStep === 2) {
        this.messages.push({ id: this.messages.length + 1, text: "Deuxième étape du formulaire...", isBot: true });
        this.scrollToBottom();
        // Ajoutez les boutons de l'étape suivante si nécessaire
      }
    },
    scrollToBottom() {
      nextTick(() => {
        const container = this.$refs.messagesContainer;
        container.scrollTop = container.scrollHeight;
      });
    },
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
  display: flex;
  flex-direction: column;
}
.chatbot-message {
  font-size: 12px;
  margin: 4px 0;
  padding: 10px;
  border-radius: 10px;
  max-width: 58%;
  word-wrap: break-word;
}
.chatbot-message.bot {
  background-color: #f5f5f5;
  display: flex;
  align-items: center;
  align-self: flex-start;
  margin-left: 40px;
  padding-right: 15px;
}
.chatbot-message.bot span {
  margin-left: -25px;
}
.bot-avatar {
  width: 30px;
  height: 30px;
  position: relative;
  left: -45px;
}
.chatbot-message.user {
  background-color: #4e38cc;
  color: white;
  align-self: flex-end;
  margin-right: 10px;
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
.quick-actions {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
}
.quick-actions-btn {
  background-color: white;
  border: 1px solid #4e38cc;
  color: #4e38cc;
  text-align: center;
  border-radius: 10px;
  padding: 9.25px;
  margin: 4px 0;
  margin-right: 10px;
  margin-bottom: 7px;
  cursor: pointer;
  font-size: 12px;
  box-shadow: 0px 3px 8px 0px rgb(0 0 0 / 15%);
}
.quick-actions-btn:hover {
  background-color: #4e38cc;
  color: white;
}
</style>
