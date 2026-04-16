<template>
    <div class="chat-widget">

        <div v-if="visible" class="chat-window">
            <div class="chat-header row items-center justify-between">
                <div class="header-content column justify-center">
                    <div class="row header-icons">
                        <img src="~assets/logo.png" alt="logo" class="header-icon" />
                        <img src="~assets/logo-text.png" alt="logo text" class="header-icon" />
                        <img src="~assets/magic-wand.png" alt="bot avatar" class="header-icon" />
                    </div>

                    <div class="subtitle">Hi there, how can we help?</div>
                </div>
                <q-btn class="exit-btn" flat round dense icon="close" @click="visible = false" />
            </div>

            <div class="messages" ref="messagesRef">
                <div v-for="msg in messages" :key="msg.id" class="message-row" :class="msg.sender">
                    <div v-if="msg.sender === 'bot'" class="avatar">
                        <img class="avatar" src="~assets/avatar.png" alt="bot avatar" />
                    </div>
                    <div class="message" :class="msg.sender" v-html="msg.text"></div>
                </div>
            </div>

            <div class="composer row items-center">
                <q-input v-model="input" dense borderless placeholder="Say something..." @keyup.enter="sendMessage"
                    class="flex" />

                <img src="~assets/paperclip.png" alt="clip" class="paperclip-icon" />
                <img src="~assets/send-btn.png" alt="send" class="send-icon" @click="sendMessage" />
            </div>
        </div>

        <q-btn class="cta-button" unelevated no-caps label="Ask Nitra AI" @click="visible = !visible" />

    </div>
</template>

<script setup>
import { nextTick, onMounted, reactive, ref } from 'vue'
import { MESSAGE_MOCK_MAP } from '../mock/messages'

const visible = ref(false)
const input = ref('')
const messagesRef = ref(null)

const messages = reactive([
    {
        id: Date.now(),
        text: 'Welcome to Nitra AI',
        sender: 'bot'
    }
])

function scrollToBottom() {
    nextTick(() => {
        const el = messagesRef.value
        if (el) el.scrollTop = el.scrollHeight
    })
}

onMounted(scrollToBottom)

function formatMessage(content) {
    if (!content) return ''

    let html = content

    html = html.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
    html = html.replace(
        /\[([^\]]+)\]\(([^)]+)\)/g,
        '<a href="$2" target="_blank">$1</a>'
    )
    html = html.replace(/^\d+\.\s/gm, '<li>')
    html = html.replace(/(<li>.*?)(?=\n\d+\.|\n\n|$)/gs, '$1</li>')
    html = html.replace(/(<li>.*<\/li>)/gs, '<ol>$1</ol>')
    html = html.replace(/\n/g, '<br/>')

    return html
}

function sendMessage() {
    const text = input.value?.trim()
    if (!text) return

    const now = Date.now()

    messages.push({ id: now, text, sender: 'user' })
    input.value = ''
    scrollToBottom()

    const lower = text.toLowerCase()
    const matchedKey = Object.keys(MESSAGE_MOCK_MAP).find(
        k => k.toLowerCase() === lower
    )

    let botText = 'Sorry, I encountered an error. Please try again later.'

    if (matchedKey) {
        const entry = MESSAGE_MOCK_MAP[matchedKey]
        const raw = entry?.message?.content
        botText = formatMessage(raw) || botText
    }

    setTimeout(() => {
        messages.push({ id: now + 1, text: botText, sender: 'bot' })
        scrollToBottom()
    }, 300)
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
    display: flex;
    flex-direction: column;
    position: absolute;
    bottom: calc(100% + 12px);
    right: 0;
    width: min(780px, calc(100vw - 40px));
    max-height: min(911px, calc(100vh - 80px));
    background: white;
    border-radius: 12px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
    overflow: hidden;
}

.chat-header {
    background: #264D4F;
    color: white;
    padding: 12px;
    flex: 0 0 auto;
    height: 102px;
}

.header-icon {
    margin-right: 16px;
    width: auto;
    height: 23px;
}

.send-icon {
    width: 36px;
    height: auto;
    margin-right: 20px;
}

.paperclip-icon {
    width: 20px;
    height: auto;
    margin-right: 20px;
}

.header-content {
    margin-left: 20px;
    height: 100%;
}

.header-icons {
    margin-bottom: 8px;
}

.exit-btn {
    position: absolute;
    top: 12px;
    right: 12px;
}

.avatar {
    width: 24px;
    height: auto;
}

.title {
    font-weight: 600;
}

.subtitle {
    font-size: 16px;

}

/* ✅ Single definition — scrollable container, no overflow on children */
.messages {
    flex: 1 1 auto;
    min-height: 0;
    overflow-y: auto;
    overflow-x: hidden;
    padding: 18px;
    background: #f5f5f5;
}

.message-row {
    display: flex;
    width: 100%;
    align-items: flex-start;
    /* ensure top alignment */
    gap: 12px;
    /* spacing between avatar and message */
    padding: 8px 0;
    /* vertical spacing between rows */
}

.message-row.user {
    justify-content: flex-end;
}

.message-row.bot {
    justify-content: flex-start;
}

/* ✅ No max-height or overflow-y — message expands to full height */
.message {
    padding: 10px 16px;
    border-radius: 12px;
    font-size: 18px;
    line-height: 1.5;
    margin: 0;
    /* remove margins so avatar and message align consistently */
    box-shadow: 0 1px 2px rgba(16, 24, 40, 0.04);
    word-wrap: break-word;
}

.message.user {
    background: #A9D4D6;
    max-width: 400px;
}

.message.bot {
    background: #e9ecef;
    max-width: calc(100% - 56px);
}

.message :deep(ol) {
    margin: 8px 0;
    padding-left: 0px;
    list-style-position: inside;
}

.message li {
    margin-bottom: 12px;
}

.message :deep(a) {
    color: #f97316;
    text-decoration: underline;
    font-weight: 500;
}

.message strong {
    font-weight: 600;
    color: #1f2937;
}

/* ✅ Single definition */
.composer {
    flex: 0 0 auto;
    padding: 10px;
    border-top: 1px solid #eee;
    gap: 8px;
}

.composer .q-btn {
    background-color: #264D4F !important;
    color: #fff !important;
}

.composer .q-btn:hover {
    filter: brightness(0.95);
}

.flex {
    flex: 1;
}

.avatar {
    flex: 0 0 40px;
    display: flex;
    align-items: flex-start;
    align-self: flex-start;
    /* ensure avatar aligns to top of the message */
}
</style>