<template>
  <div class="LLMConnect">
    <div class="chat">
      <textarea class="app-textarea"
                 v-model="promptText"
                 placeholder="Введите ваше сообщение..."
      >
      </textarea>
    </div>
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
      <button class="app-button"
          :disabled="loadingOpenRouter"
          @click="checkOpenRouterAPIKey"
      >
        Проверить ключ
      </button>
      <div class="response">
          Статус ключа: {{ !!responseOpenRouterAPIKeyStatus ? responseOpenRouterAPIKeyStatus === 200 ? 'действует' : `не действует (${responseOpenRouterAPIKeyStatus})` : 'нет данных'}}
      </div>
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
    <div class="chat">
      <div>Ollama</div>
      <button class="app-button"
            :disabled="loadingOllama"
            @click="sendPromptToOllama"
      >
          Отправить промт (ollama lib))
      </button>
      <button class="app-button"
            :disabled="loadingOllama"
            @click="sendPromptToOllamaFetch"
      >
          Отправить промт (fetch)
      </button>
      <button class="app-button"
            :disabled="loadingOllama"
            @click="sendPromptToOllamaOpenAPI"
      >
          Отправить промт (openAPI)
      </button>
      <div v-if="loadingOllama" class="loading">Загрузка...</div>
      <div v-if="errorOllama" class="error">Ошибка: {{ errorOllama }}</div>
      <div v-if="responseOllama" class="response">
        <pre>{{ responseOllama }}</pre>
      </div>

    </div>
  </div>
</template>

<script setup>
  import { ref } from 'vue';
  import axios from 'axios';
  import {Ollama} from 'ollama';
  import OpenAI from "openai";

  const OPENROUTER_API_KEY = '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';
  const OPENROUTER_MODEL_NAME = 'mistralai/mistral-7b-instruct:free';
  const OPENROUTER_OPENAPI_URL = 'https://openrouter.ai/api/v1';
  const OPENROUTER_AUTH_URL = 'https://openrouter.ai/api/v1/auth/key';
  const LMSTUDIO_MODEL_NAME = 'qwen2-vl-7b-instruct';
  const LMSTUDIO_OPENAPI_URL = 'http://192.168.0.100:1234/v1/chat/completions';

  const OLLAMA_API_KEY = 'ollama';
  const OLLAMA_MODEL_NAME = 'llama3.2:1b';

  const OLLAMA_OPENAPI_URL = 'http://192.168.0.100:11434/v1/';
  const OLLAMA_HOST = 'http://192.168.0.100:11434';
  const OLLAMA_ENDPOINT = '/api/generate';
  // const OLLAMA_ENDPOINT = '/api/chat';

  const promptText = ref('2+2=?');
  const responseOpenRouter = ref('');
  const loadingOpenRouter = ref(false);
  const errorOpenRouter = ref('');

  const responseOpenRouterAPIKeyStatus = ref('');
  const loadingOpenRouterAPIKeyStatus = ref(false);
  const errorOpenRouterAPIKeyStatus = ref('');

  const responseLMStudio = ref('');
  const loadingLMStudio = ref(false);
  const errorLMStudio = ref('');

  const responseOllama = ref('');
  const loadingOllama = ref(false);
  const errorOllama = ref('');

  async function sendPromptToOpenRouter() {
      if (!promptText.value.trim()) return;

      loadingOpenRouter.value = true;
      errorOpenRouter.value = '';
      responseOpenRouter.value = '';

      const openai = new OpenAI({
        baseURL: OPENROUTER_OPENAPI_URL,
        apiKey: OPENROUTER_API_KEY,
        dangerouslyAllowBrowser: true
      });

      openai.chat.completions.create({
          model: OPENROUTER_MODEL_NAME,
          messages: [
            { role: 'system', content: 'Всегда добавляй вначале "Да, мой господин"'},
            { role: 'user', content: promptText.value },
          ],
          temperature: 0.7
        })
        .then((res)=>{
          responseOpenRouter.value = res.choices[0].message.content;
        }).catch((err)=>{
          errorOpenRouter.value = err.message || 'Ошибка запроса';
        }).finally(()=>{
          loadingOpenRouter.value = false;
        });
  }
  async function checkOpenRouterAPIKey() {
    loadingOpenRouterAPIKeyStatus.value = true;
    errorOpenRouterAPIKeyStatus.value = '';
    responseOpenRouterAPIKeyStatus.value = '';

    fetch(OPENROUTER_AUTH_URL, {
        method: 'GET',
        headers: {
          Authorization: 'Bearer ' + OPENROUTER_API_KEY,
        },
      }).then((res)=>{
        responseOpenRouterAPIKeyStatus.value = res.status;
      }).catch((err)=>{
        errorOpenRouterAPIKeyStatus.value = err.message || 'Ошибка запроса';
      }).finally(()=>{
        loadingOpenRouterAPIKeyStatus.value=false;
      });
  }

  async function sendPromptToLMStudio() {
      if (!promptText.value.trim()) return;

      loadingLMStudio.value = true;
      errorLMStudio.value = '';
      responseLMStudio.value = '';

      const payload = {
          model: LMSTUDIO_MODEL_NAME,
          messages: [
              { role: 'system', content: 'Отвечать на русском языке' },
              { role: 'user', content: promptText.value }
          ],
          temperature: 0.7,
          max_tokens: -1,
          stream: false
      }

      axios.post(
        LMSTUDIO_OPENAPI_URL,
        payload,
        {
            headers: { 'Content-Type': 'application/json' }
        }
        ).then((res)=> {
              responseLMStudio.value = res.data.choices[0].message.content
        }).catch((err)=> {
          errorLMStudio.value = err.response?.data?.message || err.message || 'Ошибка запроса'
        }).finally (()=> {
            loadingLMStudio.value = false;
          }
       );
  }

  async function sendPromptToOllama() {
    if (!promptText.value.trim()) return;

    loadingOllama.value = true;
    errorOllama.value = '';
    responseOllama.value = '';

    const payload = {
      model: OLLAMA_MODEL_NAME,
      messages: [
        { role: 'user', content: promptText.value },
      ],
      stream: false
    }
    const ollama = new Ollama({ host: OLLAMA_HOST });
    ollama.chat(payload)
      .then(res=>{
        console.log('ollama-js response: ', res);
        responseOllama.value = res?.message?.content;
      })
      .catch(err=>{
        errorOllama.value = err;
      })
      .finally(()=>{
        loadingOllama.value = false;
      });
  }
  async function sendPromptToOllamaFetch() {
    if (!promptText.value.trim()) return;

    loadingOllama.value = true;
    errorOllama.value = '';
    responseOllama.value = '';

    const payload = {
        model: OLLAMA_MODEL_NAME,
        prompt: promptText.value,
        stream: false
    }

    fetch(OLLAMA_HOST + OLLAMA_ENDPOINT, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(
        payload
      ),
    }).then(res=>{
        return res.json();
    }).then((data)=>{
        responseOllama.value = data.response;
    }).catch(err=>{
            errorOllama.value = err;
    }).finally(()=>{
            loadingOllama.value = false;
    });

  }

  async function sendPromptToOllamaOpenAPI() {
    if (!promptText.value.trim()) return;

    loadingOllama.value = true;
    errorOllama.value = '';
    responseOllama.value = '';

    const openai = new OpenAI({
      baseURL: OLLAMA_OPENAPI_URL,
      apiKey: OLLAMA_API_KEY,
      dangerouslyAllowBrowser: true
    });

    openai.chat.completions.create({
      model: OPENROUTER_MODEL_NAME,
      prompt: promptText.value
      // messages: [
      //   { role: 'system', content: 'Всегда добавляй вначале "Да, мой господин"'},
      //   { role: 'user', content: promptText.value },
      // ],
      // temperature: 0.7
    }).then((res)=>{
      responseOllama.value = res.choices[0].message.content;
    }).catch((err)=>{
      errorOllama.value = err.message || 'Ошибка запроса';
    }).finally(()=>{
      loadingOllama.value = false;
    });
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
