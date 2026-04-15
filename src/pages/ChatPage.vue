<template>
    <div class="chat-widget">

        <!-- Chat Window -->
        <div v-if="visible" class="chat-window column">

            <!-- Header -->
            <div class="chat-header row items-center justify-between">
                <div>
                    <div class="title">Nitra AI</div>
                    <div class="subtitle">Hi there, how can we help?</div>
                </div>
                <q-btn flat round dense icon="close" @click="visible = false" />
            </div>

            <!-- Messages -->
            <div class="messages" ref="messagesRef">
                <div v-for="msg in messages" :key="msg.id" class="message-row">
                    <div class="message">
                        {{ msg.text }}
                    </div>
                </div>
            </div>

            <!-- Composer -->
            <div class="composer row items-center">
                <q-input v-model="input" dense borderless placeholder="Say something..." @keyup.enter="sendMessage"
                    class="flex" />
                <q-btn round unelevated icon="send" @click="sendMessage" />
            </div>

        </div>

        <!-- Ask Nitra button -->
        <q-btn class="cta-button" unelevated no-caps label="Ask Nitra AI" @click="visible = !visible" />

    </div>
</template>

<script setup>
import { nextTick, ref } from 'vue'

const visible = ref(false)
const input = ref('')
const messagesRef = ref(null)

const messages = ref([
    { id: 1, text: 'Welcome to Nitra AI!' }
])

function scrollToBottom() {
    nextTick(() => {
        const el = messagesRef.value
        if (el) el.scrollTop = el.scrollHeight
    })
}

function sendMessage() {
    const text = input.value?.trim()
    if (!text) return

    messages.value.push({
        id: Date.now(),
        text
    })

    input.value = ''
    scrollToBottom()
}
</script>

<style scoped>
.chat-widget {
    position: fixed;
    right: 20px;
    bottom: 20px;
    z-index: 1000;
}


.cta-button {
    background: #ff7a1a;
    color: white;
    border-radius: 24px;
    padding: 10px 16px;
    font-weight: 500;
}


.chat-window {
    position: absolute;
    bottom: calc(100% + 12px);
    right: 0;

    width: 340px;
    height: 480px;

    background: white;
    border-radius: 12px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
    overflow: hidden;
}


.chat-header {
    background: #264D4F;
    color: white;
    padding: 12px;
}

.title {
    font-weight: 600;
}

.subtitle {
    font-size: 12px;
    opacity: 0.8;
}


.messages {
    flex: 1;
    overflow-y: auto;
    padding: 16px;
    background: #f5f5f5;
}

.message {
    background: #e5e7eb;
    padding: 10px 12px;
    border-radius: 12px;
    max-width: 75%;
}


.composer {
    padding: 10px;
    border-top: 1px solid #eee;
    gap: 8px;
}

.flex {
    flex: 1;
}

.chat-page {
    --accent: #264D4F;
    display: flex;
    align-items: center;
    justify-content: center;
}

.chat-container {
    width: 780px;
    height: 591px;
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08);
    border-radius: 8px;
    background: #fff;
    overflow: hidden;
}

.messages {
    padding: 18px;
}

.composer .q-btn {
    background-color: var(--accent) !important;
    color: #fff !important;
}

.composer .q-btn:hover {
    filter: brightness(0.95);
}

.message-meta {
    color: var(--accent) !important;
}
</style>