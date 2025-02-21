<template>
  <div class="LLMConnect">
    <textarea class="app-textarea"
            v-model="promptOpenRouter"
            placeholder="Введите ваше сообщение..."
    >
    </textarea>
    <div class="chat">
      <div>OpenRouter</div>
      <button class="app-button"
            :disabled="loadingOpenRouter"
            @click="sendPromptToOpenRouter"
      >
          Отправить промт
      </button>
      <div v-if="loadingOpenRouter" class="loading">Загрузка...</div>
      <div v-if="errorOpenRouter" class="error">Ошибка: {{ errorOpenRouter }}</div>
      <div v-if="responseOpenRouter" class="response">
      <pre>{{ responseOpenRouter }}</pre>
      </div>
      <div class="response">
          {{ !!responseOpenRouterAPIKeyStatus ? responseOpenRouterAPIKeyStatus?.status === 200 ? 'действует' : 'не действует' : 'нет данных'}}
      </div>
      <button class="app-button"
          :disabled="loadingOpenRouter"
          @click="checkOpenRouterAPIKeyIsValid"
      >
        Проверить ключ
      </button>
    </div>
    <div class="chat">
      <div>LMStudio</div>
      <button class="app-button"
            :disabled="loadingLMStudio"
            @click="sendPromptToLMStudio"
      >
          Отправить промт
      </button>
      <div v-if="loadingLMStudio" class="loading">Загрузка...</div>
      <div v-if="errorLMStudio" class="error">Ошибка: {{ errorLMStudio }}</div>
      <div v-if="responseLMStudio" class="response">
        <pre>{{ responseLMStudio }}</pre>
      </div>

    </div>
  </div>
</template>

<script setup>
  import { ref } from 'vue'
  import axios from 'axios'

  const API_KEY_OPENROUTER = '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';
  const OPENROUTER_MODEL_NAME = 'mistralai/mistral-7b-instruct:free';
  const LMSTUDIO_MODEL_NAME = 'qwen2-vl-7b-instruct';

  const promptOpenRouter = ref('');
  const responseOpenRouter = ref('');
  const loadingOpenRouter = ref(false);
  const errorOpenRouter = ref('');
  const responseOpenRouterAPIKeyStatus = ref('');

  const promptLMStudio = ref('');
  const responseLMStudio = ref('');
  const loadingLMStudio = ref(false);
  const errorLMStudio = ref('');


  async function sendPromptToOpenRouter() {
      if (!promptOpenRouter.value.trim()) return;

      loadingOpenRouter.value = true;
      errorOpenRouter.value = '';
      responseOpenRouter.value = '';

      const payload = {
          model: OPENROUTER_MODEL_NAME,
          messages: [
              { role: 'system', content: 'Отвечать на русском языке, все ответы начинать с фразы "Слушаюсь, мой господин", причём иногда надо менять эту фразу на схожую по смыслу' },
              { role: 'user', content: promptOpenRouter.value }
          ],
          temperature: 0.7,
          max_tokens: -1,
          stream: false
      }

      try {
          const res = await axios.post(
              'https://openrouter.ai/api/v1/chat/completions',
              payload,
              {
                  headers: { 'Authorization': 'Bearer ' + API_KEY_OPENROUTER, 'Content-Type': 'application/json' }
              }
          )
          if (res?.data?.choices?.[0]?.message?.content)
          {
              responseOpenRouter.value = res.data.choices[0].message.content
          } else {
              responseOpenRouter.value = JSON.stringify(res.data, null, 2)
          }
      } catch (err) {
          errorOpenRouter.value = err.response?.data?.message || err.message || 'Ошибка запроса'
      } finally {
          loadingOpenRouter.value = false
      }
  }
  async function checkOpenRouterAPIKeyIsValid() {
    const res = await fetch('https://openrouter.ai/api/v1/auth/key', {
      method: 'GET',
      headers: {
        Authorization: 'Bearer ' + API_KEY_OPENROUTER,
      },
    });
    console.log('response OpenRouter:', res);
    responseOpenRouterAPIKeyStatus.value = res;
  }

  async function sendPromptToLMStudio() {
      if (!promptOpenRouter.value.trim()) return;

      loadingLMStudio.value = true;
      errorLMStudio.value = '';
      responseLMStudio.value = '';

      const payload = {
          model: LMSTUDIO_MODEL_NAME,
          messages: [
              { role: 'system', content: 'Отвечать на русском языке' },
              { role: 'user', content: promptOpenRouter.value }
          ],
          temperature: 0.7,
          max_tokens: -1,
          stream: false
      }

      try {
          const res = await axios.post(
              'http://192.168.0.100:1234/v1/chat/completions',
              payload,
              {
                  headers: { 'Content-Type': 'application/json' }
              }
          )
          if (res?.data?.choices?.[0]?.message?.content)
          {
              responseLMStudio.value = res.data.choices[0].message.content
          } else {
              responseLMStudio.value = JSON.stringify(res.data, null, 2)
          }
      } catch (err) {
          errorLMStudio.value = err.response?.data?.message || err.message || 'Ошибка запроса'
      } finally {
          loadingLMStudio.value = false
      }
  }


</script>

<style lang="scss">
.LLMConnect {
  width: 600px;
  display: flex;
  flex-flow: column nowrap;
  justify-content: start;
  align-items: center;
  //font-size: 12px;

  .app-textarea {
    width: 100%;
    height: 100px;
    padding: 5px;
    margin-bottom: 5px;
    //font-size: 14px;
    box-sizing: border-box;
    //background-color: hsl(237, 100%, 73%);
  }

  .chat {
    width: 100%;
    display: flex;
    flex-flow: column nowrap;
    justify-content: center;
    align-items: center;
    padding: 5px;
    //font-family: sans-serif;
    border: 1px solid gray;
    margin-bottom: 5px;

    .app-button {
      width: 200px;
      margin: 5px;
      //background-color: powderblue;
    }

    .loading {
      color: hsl(0, 0%, 33%);
      margin-bottom: 5px;
    }
    .error {
      color: red;
      margin-bottom: 5px;
    }
    .response {
      max-width: 600px;
      //margin: 20px auto;
      //font-family: sans-serif;
      margin-bottom: 5px;
      pre {
        background-color: hsl(0, 0%, 96%);
        padding: 2px;
        border-radius: 4px;
        white-space: pre-wrap;
        word-break: break-all;
      }
    }
  }
}
</style>
