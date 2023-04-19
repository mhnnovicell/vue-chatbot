<template>
  <div class="flex flex-col h-screen">
    <div class="flex flex-col flex-1 p-4 bg-gray-50">
      <div class="flex-1 overflow-y-scroll">
        <div v-for="message in messages" :key="message.id" class="my-2">
          <div
            v-if="message.from === 'bot'"
            class="inline-block p-2 bg-gray-200 rounded-lg"
          >
            <p class="text-gray-800">{{ message.text }}</p>
          </div>
          <div
            v-else
            class="inline-block p-2 text-white bg-blue-500 rounded-lg"
          >
            <p>{{ message.text }}</p>
          </div>
        </div>
      </div>
      <form @submit.prevent="sendMessage" class="flex mt-4">
        <input
          v-model="question"
          class="flex-1 px-4 py-2 mr-2 border border-gray-400 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          type="text"
          placeholder="Type your message here"
        />
        <button
          class="px-4 py-2 text-white bg-blue-500 rounded-md"
          type="submit"
        >
          Send
        </button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { Configuration, OpenAIApi } from 'openai';
import { onMounted, ref, reactive } from 'vue';

const configuration = new Configuration({
  apiKey: import.meta.env.VITE_OPENAI_SECRET_KEY,
});

const openai = new OpenAIApi(configuration);

const state = reactive({
  question: '',
  messages: [{ id: 1, from: 'bot', text: 'Hello, how can I help you today?' }],
});

const sendMessage = async () => {
  const prompt = `What is ${state.question}?`;

  try {
    // Use OpenAI to generate a response to the user's question
    const result = await openai.completions.create({
      engine: 'text-davinci-002',
      prompt,
      max_tokens: 100,
      n: 1,
      stop: '\n',
    });

    const answer = result.choices[0].text.trim();
    console.log(`${prompt} ${answer}`);

    // Save the answer as a message
    state.messages.push({ id: Date.now(), from: 'bot', text: answer });
  } catch (err) {
    console.error(`Error generating response for ${prompt}`, err);
  }

  // Clear the input field
  state.question = '';
};
</script>
