<template>
  <v-container>
    <v-card class="bg-blue-lighten-5">
      <v-card-title class="text-center"> CHAT - {{ userId }} </v-card-title>
      <v-select
        v-model="selectedUser"
        :items="otherUserList"
        label="Select User to Chat With"
      ></v-select>
      <v-list class="bg-blue-lighten-5">
        <v-list-item
          v-for="(msg, index) in messages"
          :key="index"
          :class="{ 'sent-message': msg.fromUserId === userId }"
        >
          <span class="text-green">{{ msg.fromUserId }}</span
          ><br />
          {{ msg.message }}
        </v-list-item>
      </v-list>
      <v-card-actions>
        <v-text-field
          variant="outlined"
          v-model="newMessage"
          @keyup.enter="sendMessage"
          label="Type your message"
        ></v-text-field>
        <v-btn
          icon="mdi-send"
          color="green"
          rounded
          variant="elevated"
          class="mb-4 mx-2"
          @click="sendMessage"
        ></v-btn>
      </v-card-actions>
    </v-card>
  </v-container>
</template>

<script lang="ts" setup>
import { ref, onMounted } from "vue";
import { io, Socket } from "socket.io-client";

interface ChatMessage {
  fromUserId: string;
  toUserId: string;
  message: string;
  GroupChat: boolean;
}

const socket: Socket = io("http://localhost:3000");
const userId = ref<string>("");
const newMessage = ref<string>("");
const messages = ref<ChatMessage[]>([]);
const userList = ref<string[]>([
  "MARISELVAN",
  "JAIN",
  "ADITHYA",
  "ANISH",
  "KIRAN",
  "ALL",
]);
const otherUserList = ref<string[]>();
const selectedUser = ref<string>("");

onMounted(() => {
  socket.on("user connected", (id: string) => {
    userId.value = id;

    otherUserList.value = userList.value.filter((u) => u !== userId.value);
  });

  socket.on("chat message", (msg: ChatMessage) => {
    messages.value.push(msg);
  });
});

const sendMessage = () => {
  if (newMessage.value.trim() !== "" && selectedUser.value !== "") {
    const messageToSend: ChatMessage = {
      fromUserId: userId.value,
      toUserId: selectedUser.value,
      message: newMessage.value,
      GroupChat: selectedUser.value === "ALL" ? true : false,
    };
    socket.emit("chat message", messageToSend);
    newMessage.value = "";
  }
};
</script>

<style scoped>
.v-container {
  max-width: 500px;
  margin: auto;
  padding-top: 50px;
}

.sent-message {
  text-align: right;
}

.text-green {
  font-weight: bold;
}
</style>
