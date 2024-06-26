<script>
import ollama from "ollama/browser";
import * as marked from 'marked';
import hljs from 'highlight.js';

marked.setOptions({
  highlight: (code, lang) => {
    if (hljs.getLanguage(lang)) {
      return hljs.highlight(code, { language: lang }).value;
    } else {
      return hljs.highlightAuto(code).value;
    }
  },
});

export default {
  props: {
    model: String, // Receive the model prop
  },
  data() {
    return {
      isOllamaRunning: true,
      list: [],
      conversation: [{ prompt: "", response: "" }],
      newMessage: "",
      warning: "",
    };
  },
  mounted() {
    this.getAvailableModels();
    this.fetchResponse();
  },
  watch: {
    model(newModel) {
      console.log("Selected model:", newModel); // Log the selected model
      this.warning = "Model: " + newModel
      this.fetchResponse();
      
    },
  },
  methods: {
    async getAvailableModels() {
      const models = await ollama.list();
      console.log(models);
      for (let i = 0, j = models.length; i < j; i++) {
        this.list.push(models[i].name);
      }
    },
    convertToMarkdown(text){
      console.log(marked.parse(text))
      return marked.parse(text)
      // return marked(text)
    },
    async fetchResponse() {
      try {
        if (this.model) {
          const response = await ollama.chat({
            model: this.model,
            messages: [
              {
                role: "user",
                content: this.conversation[this.conversation.length - 1].prompt,
              },
            ],
            stream: true,
          });
          for await (const part of response) {
            this.conversation[this.conversation.length - 1].response +=
              part.message.content;
          }
          this.reponseMarkdown()
        } else {
          this.warning = "Please select a model first.";
        }
      } catch (error) {
        console.error("Error fetching response:", error);
        this.response = "Error: " + error.message;
      }
    },
    handleNewMessage() {
      if (this.newMessage.trim() !== "") {
        this.conversation.push({ prompt: this.newMessage, response: "" });
        this.fetchResponse();
        this.newMessage = ""; // Clear the input field
      }
    },
  },
};
</script>

<template>
  <div class="flex flex-col justify-between h-full px-40">
    <div class="p-4 overflow-y-auto overflow-x-hidden">
      <div class="chat chat-start w-full">
        <div class="chat-image avatar">
          <div class="w-10 rounded-full">
            <img
              class="bg-white !object-contain"
              alt="Tailwind CSS chat bubble component"
              src="https://ollama.com/public/ollama.png"
            />
          </div>
        </div>
        <div class="flex flex-col gap-2 w-full">
          <div
            v-for="(item, index) in conversation"
            :key="index"
            class="flex flex-col gap-2 w-full"
          >
            <div class="chat chat-end">
              <span v-if="item.prompt" class="chat-bubble">{{
                item.prompt
              }}</span>
            </div>
            <div class="chat chat-start">
              <span v-if="item.response" class="chat-bubble" v-html="convertToMarkdown(item.response)" ></span>
              <span
                v-else-if="!item.prompt && !item.response"
                class="chat-bubble"
                >{{ warning }}</span
              >
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="p-4 flex justify-center gap-2">
      <input
        v-model="newMessage"
        type="text"
        placeholder="Message here..."
        class="input input-bordered w-full sticky bottom-0"
        @keyup.enter="handleNewMessage"
      />
      <button class="btn btn-primary">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          class="lucide lucide-send-horizontal"
        >
          <path d="m3 3 3 9-3 9 19-9Z" />
          <path d="M6 12h16" />
        </svg>
      </button>
    </div>
  </div>
</template>

<style scoped></style>
