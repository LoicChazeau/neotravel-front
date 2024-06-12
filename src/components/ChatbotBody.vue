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
        <div v-for="option in currentOptions" :key="option" class="quick-actions-btn" @click="handleFormStep(option)">
          {{ option }}
        </div>
      </div>
    </div>
    <div class="chatbot-input-container">
      <input
        class="chatbot-input"
        :type="inputType"
        :placeholder="inputPlaceholder"
        v-model="userInput"
        @keyup.enter="sendMessage()"
        :disabled="inputDisabled"
        ref="userInput"
      />
      <img class="chatbot-input-btn" @click="sendMessage()" :src="sendUrl" alt="Send button" :class="{ disabled: inputDisabled }" />
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
      formData: {},
      inputDisabled: false,
      inputPlaceholder: "Une question ?",
      currentOptions: [],
      isRoundTrip: false,
      optionAller: null,
      optionRetour: null,
      inputType: "text",
    };
  },
  methods: {
    validatePhone(phone) {
      const phoneRegex = /^0[1-9]\d{8}$/; // Format sans espace
      const phoneWithSpacesRegex = /^0[1-9]( \d{2}){4}$/; // Format avec espace
      return phoneRegex.test(phone.replace(/\s/g, "")) || phoneWithSpacesRegex.test(phone);
    },
    validateEmail(email) {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return emailRegex.test(email);
    },
    async sendMessage(message = null) {
      const content = message || this.userInput.trim();
      this.userInput = "";
      if (content) {
        console.log("sendMessage called", content);

        if (this.isFormStep) {
          if (this.currentStep === 5 && !this.validatePhone(content)) {
            this.messages.push({
              id: this.messages.length + 1,
              text: "Numéro de téléphone invalide. <br> Veuillez entrer un numéro valide.",
              isBot: true,
            });
            this.scrollToBottom();
            return;
          }
          if (this.currentStep === 6 && !this.validateEmail(content)) {
            this.messages.push({
              id: this.messages.length + 1,
              text: "Adresse email invalide. <br> Veuillez entrer une adresse valide.",
              isBot: true,
            });
            this.scrollToBottom();
            return;
          }
          this.handleFormInput(content);
          return;
        }

        this.messages.push({ id: this.messages.length + 1, text: content, isBot: false });
        this.showQuickActions = false;
        this.scrollToBottom();

        try {
          const res = await fetch("http://10.76.203.217:5000/conversation", {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              description: content,
            }),
          });
          if (!res.ok) {
            throw new Error(`HTTP error! status: ${res.status}`);
          }
          const data = await res.json();
          console.log("data", data);
          this.response = data.response;
          if (data.type == "functionCall") {
            if (data.functionName == "estimate") {
              this.startForm();
            } else if (data.functionName == "askCommercial") {
              this.messages.push({
                id: this.messages.length + 1,
                text: "Vous pouvez nous appeler au : <br><span style='font-weight: bold'>09 80 40 04 84</span><br>Du lundi au vendredi de 10h à 18h.",
                isBot: true,
              });
              this.scrollToBottom();
            }
          } else if (data.type == "conversation") {
            this.messages.push({
              id: this.messages.length + 1,
              text: this.response,
              isBot: true,
            });
            this.scrollToBottom();
          }
          this.error = null;
        } catch (error) {
          this.error = error.toString();
          console.error(this.error);
          this.response = null;
        }
      }
    },
    async sendFormData() {
      console.log("sendFormData called", this.formData);
      try {
        const res = await fetch("http://10.76.203.217:5000/sendDevis", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(this.formData),
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
        console.error(this.error);
        this.response = null;
      }
    },
    handleQuickAction(action) {
      console.log(`Quick action selected: ${action}`);
      this.sendMessage(action);
      this.showQuickActions = false;
      this.scrollToBottom();
    },
    startForm() {
      // Etape 1
      this.messages.push({
        id: this.messages.length + 1,
        text: "Pour la création du devis nous avons besoin de quelques informations.",
        isBot: true,
      });
      this.inputDisabled = true;
      this.inputPlaceholder = " ";
      this.scrollToBottom();
      setTimeout(() => {
        this.messages.push({ id: this.messages.length + 1, text: "Vous êtes :", isBot: true });
        this.scrollToBottom();
        this.currentStep = 1;
        this.currentOptions = ["Particulier", "Entreprise", "Association", "Collectivité"];
      }, 1500);
      this.isFormStep = true;
    },
    handleFormStep(selection) {
      console.log(`Form step selection: ${selection}`);
      this.messages.push({ id: this.messages.length + 1, text: selection, isBot: false });

      switch (this.currentStep) {
        case 1:
          this.formData.Type_prospect = selection;
          break;
        case 2:
          this.formData.Civilite = selection;
          break;
        case 7:
          this.isRoundTrip = selection === "Aller retour";
          this.formData.Type_voyage = selection;
          break;
        case 12:
          this.optionAller = selection;
          this.formData.Option_aller = selection;
          break;
        case 15:
          this.optionRetour = selection;
          this.formData.Option_retour = selection;
          break;
        case 18:
          if (selection == "Oui") {
            this.formData["Acceptation CGU/CGV"] = "on";
          } else if (selection == "Non") {
            this.formData["Acceptation CGU/CGV"] = "off";
          }
          break;
      }
      this.nextFormStep();
      this.scrollToBottom();
    },
    handleFormInput(content) {
      console.log(`Form step content: ${content}`);
      this.messages.push({ id: this.messages.length + 1, text: content, isBot: false });

      switch (this.currentStep) {
        case 3:
          this.formData.Nom_prospect = content;
          break;
        case 4:
          this.formData.Prenom_prospect = content;
          break;
        case 5:
          this.formData.Telephone_prospect = content;
          break;
        case 6:
          this.formData.Email_prospect = content;
          break;
        case 8:
          this.formData.Nombre_participants = parseInt(content, 10);
          break;
        case 9:
          this.formData.Lieu_depart = content;
          break;
        case 10:
          this.formData.Lieu_arrivee = content;
          break;
        case 11:
          this.formData.Date_aller = content;
          break;
        case 13:
          this.formData.Horaire_aller = content;
          break;
        case 14:
          this.formData.Date_retour = content;
          break;
        case 16:
          this.formData.Horaire_retour = content;
          break;
        case 17:
          this.formData.Informations_complementaires = content;
          this.sendFormData();
          break;
      }
      this.nextFormStep();
      this.scrollToBottom();
    },
    nextFormStep() {
      this.currentOptions = [];
      this.inputDisabled = true;
      this.inputPlaceholder = " ";
      this.inputType = "text";
      this.currentStep++;
      // Etape 2
      if (this.currentStep === 2) {
        setTimeout(() => {
          this.messages.push({ id: this.messages.length + 1, text: "Quelle est votre civilité ?", isBot: true });
          this.scrollToBottom();
          this.inputDisabled = true;
          this.inputPlaceholder = " ";
          this.currentOptions = ["M.", "Mme.", "Melle."];
        }, 1000);
      }
      // Etape 3
      if (this.currentStep === 3) {
        setTimeout(() => {
          this.messages.push({ id: this.messages.length + 1, text: "Quelle est votre nom de famille ?", isBot: true });
          this.scrollToBottom();
          this.inputDisabled = false;
          this.inputPlaceholder = "Nom de famille";
          this.$nextTick(() => this.$refs.userInput.focus());
        }, 1000);
      }
      // Etape 4
      if (this.currentStep === 4) {
        setTimeout(() => {
          this.messages.push({ id: this.messages.length + 1, text: "Quelle est votre prénom ?", isBot: true });
          this.scrollToBottom();
          this.inputDisabled = false;
          this.inputPlaceholder = "Prénom";
          this.$nextTick(() => this.$refs.userInput.focus());
        }, 1000);
      }
      // Etape 5
      if (this.currentStep === 5) {
        setTimeout(() => {
          this.messages.push({
            id: this.messages.length + 1,
            text: "Nous avons maintenant besoin de vos coordonées. <br> Quel est votre numéro téléphone ?",
            isBot: true,
          });
          this.scrollToBottom();
          this.inputDisabled = false;
          this.inputPlaceholder = "Téléphone";
          this.inputType = "tel";
          this.$nextTick(() => this.$refs.userInput.focus());
        }, 1000);
      }
      // Etape 6
      if (this.currentStep === 6) {
        setTimeout(() => {
          this.messages.push({ id: this.messages.length + 1, text: "Et votre adresse email ?", isBot: true });
          this.scrollToBottom();
          this.inputDisabled = false;
          this.inputPlaceholder = "Email";
          this.inputType = "email";
          this.$nextTick(() => this.$refs.userInput.focus());
        }, 1000);
      }
      // Etape 7
      if (this.currentStep === 7) {
        setTimeout(() => {
          this.messages.push({
            id: this.messages.length + 1,
            text: "Nous allons maintenant passer aux détails de votre voyage. <br> Souhaitez-vous faire un aller simple ou un aller retour ?",
            isBot: true,
          });
          this.scrollToBottom();
          this.inputDisabled = true;
          this.inputPlaceholder = " ";
          this.currentOptions = ["Aller simple", "Aller retour"];
        }, 1000);
      }
      // Etape 8
      if (this.currentStep === 8) {
        setTimeout(() => {
          this.messages.push({
            id: this.messages.length + 1,
            text: "Pour combien de passagers ?",
            isBot: true,
          });
          this.scrollToBottom();
          this.inputDisabled = false;
          this.inputPlaceholder = "Nombre de passagers";
          this.inputType = "number";
          this.$nextTick(() => this.$refs.userInput.focus());
        }, 1000);
      }
      // Etape 9
      if (this.currentStep === 9) {
        setTimeout(() => {
          this.messages.push({
            id: this.messages.length + 1,
            text: "Quel est le lieu de départ ?",
            isBot: true,
          });
          this.scrollToBottom();
          this.inputDisabled = false;
          this.inputPlaceholder = "Lieu de départ";
          this.inputType = "text";
          this.$nextTick(() => this.$refs.userInput.focus());
        }, 1000);
      }
      // Etape 10
      if (this.currentStep === 10) {
        setTimeout(() => {
          this.messages.push({
            id: this.messages.length + 1,
            text: "Et le lieu d'arrivée ?",
            isBot: true,
          });
          this.scrollToBottom();
          this.inputDisabled = false;
          this.inputPlaceholder = "Lieu d'arrivée";
          this.$nextTick(() => this.$refs.userInput.focus());
        }, 1000);
      }
      // Etape 11
      if (this.currentStep === 11) {
        setTimeout(() => {
          this.messages.push({
            id: this.messages.length + 1,
            text: "Quelle est la date de l'aller ?",
            isBot: true,
          });
          this.scrollToBottom();
          this.inputDisabled = false;
          this.inputPlaceholder = "Date de l'aller";
          this.inputType = "date";
          this.$nextTick(() => this.$refs.userInput.focus());
        }, 1000);
      }
      // Etape 12
      if (this.currentStep === 12) {
        setTimeout(() => {
          this.messages.push({
            id: this.messages.length + 1,
            text: "Vous souhaitez nous donner l'heure de départ ou d'arrivée de l'aller ?",
            isBot: true,
          });
          this.scrollToBottom();
          this.inputDisabled = true;
          this.inputPlaceholder = " ";
          this.currentOptions = ["Départ à", "Arrivée à"];
        }, 1000);
      }
      // Etape 13
      if (this.currentStep === 13) {
        if (this.optionAller == "Départ à") {
          setTimeout(() => {
            this.messages.push({
              id: this.messages.length + 1,
              text: "Quelle est l'heure de départ de l'aller ?",
              isBot: true,
            });
            this.scrollToBottom();
            this.inputDisabled = false;
            this.inputPlaceholder = "Heure de départ aller";
            this.inputType = "time";
            this.$nextTick(() => this.$refs.userInput.focus());
          }, 1000);
        } else if (this.optionAller == "Arrivée à") {
          setTimeout(() => {
            this.messages.push({
              id: this.messages.length + 1,
              text: "Quelle est l'heure d'arrivée de l'aller ?",
              isBot: true,
            });
            this.scrollToBottom();
            this.inputDisabled = false;
            this.inputPlaceholder = "Heure d'arrivée aller";
            this.$nextTick(() => this.$refs.userInput.focus());
          }, 1000);
        }
      }
      // Etape 14
      if (this.currentStep === 14) {
        if (this.isRoundTrip) {
          setTimeout(() => {
            this.messages.push({
              id: this.messages.length + 1,
              text: "Quelle est la date du retour ?",
              isBot: true,
            });
            this.scrollToBottom();
            this.inputDisabled = false;
            this.inputPlaceholder = "Date du retour";
            this.$nextTick(() => this.$refs.userInput.focus());
          }, 1000);
        } else {
          this.currentStep++;
        }
      }
      // Etape 15
      if (this.currentStep === 15) {
        if (this.isRoundTrip) {
          setTimeout(() => {
            this.messages.push({
              id: this.messages.length + 1,
              text: "Vous souhaitez nous donner l'heure de départ ou d'arrivée du retour ?",
              isBot: true,
            });
            this.scrollToBottom();
            this.inputDisabled = true;
            this.inputPlaceholder = " ";
            this.currentOptions = ["Départ à", "Arrivée à"];
          }, 1000);
        } else {
          this.currentStep++;
        }
      }
      // Etape 16
      if (this.currentStep === 16) {
        if (this.isRoundTrip) {
          if (this.optionRetour == "Départ à") {
            setTimeout(() => {
              this.messages.push({
                id: this.messages.length + 1,
                text: "Quelle est l'heure de départ du retour ?",
                isBot: true,
              });
              this.scrollToBottom();
              this.inputDisabled = false;
              this.inputPlaceholder = "Heure de départ retour";
              this.$nextTick(() => this.$refs.userInput.focus());
            }, 1000);
          } else if (this.optionRetour == "Arrivée à") {
            setTimeout(() => {
              this.messages.push({
                id: this.messages.length + 1,
                text: "Quelle est l'heure d'arrivée du retour ?",
                isBot: true,
              });
              this.scrollToBottom();
              this.inputDisabled = false;
              this.inputPlaceholder = "Heure d'arrivée retour";
              this.$nextTick(() => this.$refs.userInput.focus());
            }, 1000);
          }
        } else {
          this.currentStep++;
        }
      }
      // Etape 17
      if (this.currentStep === 17) {
        setTimeout(() => {
          this.messages.push({
            id: this.messages.length + 1,
            text: "Décrivez votre projet. Vous pouvez aussi donner des précisions qui pourraient impacter le prix comme par exemple des étapes ou des besoins en équipements spécifiques...",
            isBot: true,
          });
          this.scrollToBottom();
          this.inputDisabled = false;
          this.inputPlaceholder = "Description";
          this.$nextTick(() => this.$refs.userInput.focus());
        }, 1000);
      }
      // Etape 18
      if (this.currentStep === 18) {
        setTimeout(() => {
          this.messages.push({
            id: this.messages.length + 1,
            text: "J'accepte de recevoir par email, les informations en provenance du site www.autocar-location.com",
            isBot: true,
          });
          this.scrollToBottom();
          this.inputDisabled = true;
          this.inputPlaceholder = " ";
          this.currentOptions = ["Oui", "Non"];
        }, 1000);
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
  max-width: 57%;
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
  margin-left: -20px;
}
.bot-avatar {
  height: 25px;
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
.chatbot-input-btn.disabled {
  cursor: not-allowed;
  opacity: 0.5;
}
</style>
