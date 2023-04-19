<div class="flex flex-col h-screen">

</div>

<script setup>
import { Configuration, OpenAIApi } from 'openai';
import { onMounted, ref } from 'vue';

const configuration = new Configuration({
  apiKey: import.meta.env.VITE_OPENAI_SECRET_KEY,
});
const openai = new OpenAIApi(configuration);

const question = ref('');
const messages = ref([
  {
    id: 1,
    from: 'bot',
    text: 'Hi, how can I help you today?',
  },
]);

const sendMessage = () => {
  messages.value.push({
    id: messages.value.length + 1,
    from: 'user',
    text: question.value,
  });
  question.value = '';
};
</script>
