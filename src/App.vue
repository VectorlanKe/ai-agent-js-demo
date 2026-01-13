<template>
  <h1>You did it!</h1>
  <div>
    <input type="text" v-model="message" @keydown.enter="sendMessage"><input type="button" value="提交" @click="sendMessage"/>
  </div>
   <div>
    <p v-for="item in historyMessage">{{ item }}</p>
   </div>
</template>

<script setup lang="ts">
import { aisdk } from '@openai/agents-extensions';
import { Agent, run } from '@openai/agents';
import { createOllama } from 'ollama-ai-provider-v2';
import { ref } from 'vue';
import { tool } from '@openai/agents';
import { z } from 'zod';

const message=ref('');
const historyMessage=ref<String[]>([]);

const getWeatherTool = tool({
  name: 'get_weather',
  description: 'Get the weather for a given city',
  parameters: z.object({ city: z.string() }),
  async execute({ city }) {
    console.log("调用本地工具");
    return `The weather in ${city} is sunny.`;
  },
});

const ollama = createOllama({
  baseURL: 'http://localhost:11434/api',
});
const model = aisdk(ollama('qwen3:8b'));
/**
 * 创建agent
 */
const agent = new Agent({
  name: 'Assistant',
  instructions: '简洁答复,一句话回答',
  model: model,
  tools:[getWeatherTool]
});
/**
 * 发送消息
 */
async function sendMessage(){
  
  const result = await run(
    agent,
    message.value,
    {
      context:historyMessage.value
    }
  );
  console.log(result.finalOutput);
  historyMessage.value.push(message.value);
  message.value='';
}
</script>
<style scoped></style>
