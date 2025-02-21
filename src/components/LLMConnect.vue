<template>
  <div class="LLMConnect">
    <textarea class="app-textarea"
            v-model="promptOpenRouter"
            placeholder="Введите ваше сообщение..."
    >
    </textarea>
    <div class="chat">
      <div style="font-size: 12px">OpenRouter</div>
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
          {{ responseOpenRouterAPIKeyStatus?.status === 200 ? 'действует' : 'не действует'}}
      </div>
      <button class="app-button"
          :disabled="loadingOpenRouter"
          @click="checkOpenRouterAPIKeyIsValid"
      >
        Проверить ключ
      </button>
    </div>
    <div class="chat">
      <div style="font-size: 12px">LMStudio</div>
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
          model: 'mistralai/mistral-7b-instruct:free',
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
                  headers: { 'Authorization': 'Bearer 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', 'Content-Type': 'application/json' }
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
        Authorization: 'Bearer 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',
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
          model: 'llama-3.3-70b-instruct',
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
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
  font-size: 12px;

  .app-textarea {
    width: 100%;
    height: 100px;
    padding: 8px;
    margin-bottom: 5px;
    font-size: 14px;
    box-sizing: border-box;
    //background-color: hsl(237, 100%, 73%);
  }

  .chat {
    display: flex;
    flex-flow: column nowrap;
    justify-content: center;
    max-width: 600px;
    //margin: 20px auto;
    padding: 5px;
    font-family: sans-serif;
    border: 1px solid gray;
    margin-bottom: 5px;

    .loading {
      color: hsl(0, 0%, 33%);
    }
    .error {
      color: red;
    }
    .response {
      max-width: 600px;
      //margin: 20px auto;
      font-family: sans-serif;
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
