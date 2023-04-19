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
  answer: '',
});

const products = ref([]);

onMounted(async () => {
  const response = await fetch('https://fakestoreapi.com/products');
  const data = await response.json();
  products.value = data;
});

const sendMessage = async () => {
  const searchQuery = state.question;
  const productMatches = [];

  for (const product of products.value) {
    const productTitle = product.title.toLowerCase();
    if (productTitle.includes(searchQuery.toLowerCase())) {
      productMatches.push(product);
    }
  }

  let prompt = `The user is searching for ${searchQuery}. Please recommend a product from the following:\n`;

  for (const product of productMatches) {
    prompt += `- ${product.title}\n`;
  }

  const response = await openai.createCompletion({
    model: 'text-davinci-003',
    prompt: prompt,
    max_tokens: 1,
    temperature: 0,
    n: 1,
    stop: '\n',
  });

  state.answer = response.choices[0].text.trim();
};
</script>
