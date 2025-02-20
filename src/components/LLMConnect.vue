<template>
    <div class="lm-chat">
        <h1>Чат LM Studio</h1>
        <textarea
                v-model="userMessage"
                placeholder="Введите ваше сообщение..."
        ></textarea>
        <button @click="sendMessage" :disabled="loading">
            Отправить
        </button>
        <div v-if="loading" class="loading">Загрузка...</div>
        <div v-if="error" class="error">Ошибка: {{ error }}</div>
        <div v-if="response" class="response">
            <h2>Ответ модели:</h2>
            <pre>{{ response }}</pre>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const userMessage = ref('')
const response = ref('')
const loading = ref(false)
const error = ref('')

async function sendMessage() {
    if (!userMessage.value.trim()) return

    loading.value = true
    error.value = ''
    response.value = ''

    // Формирование payload для LM Studio REST API.
    // Здесь используется пример модели 'granite-3.0-2b-instruct', при необходимости замените на актуальный ID модели.
    const payload = {
        model: 'text-embedding-nomic-embed-text-v1.5',
        messages: [
            { role: "system", content: "Отвечать на русском языке" },
            { role: "user", content: userMessage.value }
        ],
        temperature: 0.7,
        max_tokens: -1,
        stream: false
    }

    try {
        const res = await axios.post(
            'http://192.168.0.100:1234/api/v0/chat/completions',
            payload,
            {
                headers: { 'Content-Type': 'application/json' }
            }
        )
        // Извлекаем ответ из первого варианта ответа
        if (
            res.data &&
            res.data.choices &&
            res.data.choices[0] &&
            res.data.choices[0].message &&
            res.data.choices[0].message.content
        ) {
            response.value = res.data.choices[0].message.content
        } else {
            response.value = JSON.stringify(res.data, null, 2)
        }
    } catch (err) {
        error.value = err.response?.data?.message || err.message || 'Ошибка запроса'
    } finally {
        loading.value = false
    }
}
</script>

<style scoped>
.lm-chat {
    max-width: 600px;
    margin: 20px auto;
    font-family: sans-serif;
}

textarea {
    width: 100%;
    height: 100px;
    padding: 8px;
    margin-bottom: 10px;
    font-size: 14px;
    box-sizing: border-box;
}

button {
    padding: 8px 16px;
    font-size: 14px;
    cursor: pointer;
    margin-bottom: 10px;
}

.loading {
    color: #555;
}

.error {
    color: red;
}

.response pre {
    background: #f5f5f5;
    padding: 10px;
    border-radius: 4px;
    white-space: pre-wrap;
    word-break: break-all;
}
</style>
