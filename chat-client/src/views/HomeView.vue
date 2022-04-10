<script setup lang="ts">
import { io } from "socket.io-client";
import { onBeforeMount, ref } from "vue";

interface Message {
  name: string;
  text: string;
}

const socket = io("http://localhost:3001");

const messages = ref<Message[]>([]);
const messageText = ref<string>("");
const name = ref<string>("");
const joined = ref<boolean>(false);
const typingDisplay = ref<string>("");

onBeforeMount(() => {
  socket.emit("findAllMessages", {}, (response: Message[]) => {
    messages.value = response;
  });

  socket.on("message", (message) => {
    messages.value.push(message);
  });

  socket.on("typing", ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} is typing...`;
    } else {
      typingDisplay.value = "";
    }
  });
});

const join = () => {
  socket.emit("join", { name: name.value }, () => {
    joined.value = true;
  });
};

const sendMessage = () => {
  socket.emit("createMessage", { text: messageText.value }, () => {
    messageText.value = "";
  });
};

let timeout;
const emitTyping = () => {
  socket.emit("typing", { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit("typing", { isTyping: false });
  }, 2000);
};
</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label for="name">What's your name?</label>
        <input v-model="name" type="text" />
        <button type="submit">Send</button>
      </form>
    </div>
    <div v-else class="chat__container">
      <div class="messages__container">
        <div v-for="(message, index) in messages" :key="index">
          [{{ message.name }}]: {{ message.text }}
        </div>
      </div>

      <div v-if="typingDisplay">{{ typingDisplay }}</div>
      <hr />

      <div class="message__input">
        <form @submit.prevent="sendMessage">
          <label for="message">Message:</label>
          <input v-model="messageText" type="text" @input="emitTyping" />
          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>
<style scoped>
.chat {
  padding: 20px;
  height: 100vh;
}
.chat__container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.messages_container {
  flex: 1;
  overflow-y: scroll;
}
</style>
