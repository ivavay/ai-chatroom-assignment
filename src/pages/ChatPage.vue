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
                    <div class="message" :class="[msg.sender, { thinking: msg.loading }]">
                        <span v-if="msg.loading" class="thinking">
                            Thinking<span class="dots"></span>
                        </span>

                        <span v-else v-html="msg.text"></span>
                    </div>
                </div>
                <div v-if="!messages.some(msg => msg.sender === 'user')" class="reminder-container">
                    <transition name="fade-up" mode="out-in">
                        <div :key="currentReminder.text" class="reminder-text">
                            <img :src="currentReminder.icon" alt="reminder icon" class="reminder-icon" />
                            {{ currentReminder.text }}
                        </div>
                    </transition>
                </div>
            </div>



            <div class="composer row items-center">
                <q-input v-model="input" dense borderless placeholder="Say something..." @keyup.enter="sendMessage"
                    class="flex placeholder" />

                <img src="~assets/paperclip.png" alt="clip" class="paperclip-icon" />
                <img src="~assets/send-btn.png" alt="send" class="send-icon" @click="sendMessage" />
            </div>
        </div>

        <q-btn class="cta-button" unelevated no-caps label="Ask Nitra AI" @click="visible = !visible" />

    </div>
</template>

<script setup>
import { nextTick, onMounted, onUnmounted, reactive, ref } from 'vue'
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


// ensure messages are scrolled to bottom when new message is sent 
function scrollToBottom() {
    nextTick(() => {
        const el = messagesRef.value
        if (el) el.scrollTop = el.scrollHeight
    })
}

onMounted(scrollToBottom)

// for formatting the bot response to HTML display from mock data JSON
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

/*  typewriter helper function  */
function typeMessage(messageIndex, fullText, speed = 20) {
    let i = 0
    messages[messageIndex].typing = true

    const timer = setInterval(() => {
        if (i <= fullText.length) {
            const partialText = fullText.slice(0, i)
            messages[messageIndex].text = formatMessage(partialText)
            i++
            scrollToBottom()
        } else {
            clearInterval(timer)
            messages[messageIndex].typing = false
        }
    }, speed)
}

/* sendMessage -- adds the user's messages to arr, shows "Thinking" message for 5 secs before response/fallback */
function sendMessage() {
    const text = input.value?.trim()
    if (!text) return

    const now = Date.now()

    // add user's message
    messages.push({ id: now, text, sender: 'user' })

    // clear composer and scroll
    input.value = ''
    scrollToBottom()

    // lookup mock response
    const lower = text.toLowerCase()
    const matchedKey = Object.keys(MESSAGE_MOCK_MAP).find(k => k.toLowerCase() === lower)

    let botText = 'Sorry, I encountered an error. Please try again later.'
    if (matchedKey) {
        const entry = MESSAGE_MOCK_MAP[matchedKey]
        const raw = entry?.message?.content
        botText = raw || botText
    }

    // insert temporary 'Thinking...' message and remember its id
    const thinkingId = now + 1
    messages.push({ id: thinkingId, text: 'Thinking...', sender: 'bot', loading: true })
    scrollToBottom()

    // after 5 seconds replace the temporary message with the final response
    setTimeout(() => {
        const idx = messages.findIndex(m => m.id === thinkingId)
        if (idx !== -1) {
            messages[idx].loading = false
            messages[idx].text = ''
            messages[idx].typing = true
            typeMessage(idx, botText, 20)
        }
        scrollToBottom()
    }, 5000)
}


const reminders = [
    {
        text: 'Upload your supplier list ',
        icon: new URL('../assets/clip.png', import.meta.url).href
    },
    {
        text: 'Check if Avastin is in stock',
        icon: new URL('../assets/cart.png', import.meta.url).href
    },
    {
        text: "Check if there's a better price for Xeomin",
        icon: new URL('../assets/hand.png', import.meta.url).href
    },
    {
        text: 'What are some generic options for Restylane',
        icon: new URL('../assets/glass.png', import.meta.url).href
    },
    {
        text: 'What’s the best product for Xeomin',
        icon: new URL('../assets/thumb.png', import.meta.url).href
    }
]

const index = ref(0)
const currentReminder = ref(reminders[0])

let interval

onMounted(() => {
    interval = setInterval(() => {
        index.value = (index.value + 1) % reminders.length
        currentReminder.value = reminders[index.value]
    }, 5000)
})

onUnmounted(() => {
    clearInterval(interval)
})
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
    min-height: 591px;
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
    gap: 12px;
    padding: 8px 0;
}

.message-row.user {
    justify-content: flex-end;
}

.message-row.bot {
    justify-content: flex-start;
}

.message {
    padding: 10px 16px;
    font-size: 18px;
    line-height: 1.5;
    margin: 0;
    box-shadow: 0 1px 2px rgba(16, 24, 40, 0.04);
    word-wrap: break-word;
}

.message.user {
    background: #A9D4D6;
    max-width: 400px;
    border-radius: 12px 0 12px 12px;
}

.message.bot {
    background: #e9ecef;
    max-width: calc(100% - 56px);
    border-radius: 0 12px 12px 12px;
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

.composer {
    flex: 0 0 auto;
    padding: 16px 20px;
    border-top: 1px solid #eee;
    gap: 8px;
    font-size: 16px;
    height: 68px;
}

.placeholder {
    font-size: 16px;
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
}

.reminder-container {
    position: absolute;
    bottom: 88px;
    left: 20px;
    display: flex;
    align-items: center;
    justify-content: flex-start;
}

.reminder-text {
    color: #5C6970;
    font-size: 16px;
}

.reminder-icon {
    width: 16px;
    height: auto;
    margin-right: 8px;
}

.fade-up-enter-active,
.fade-up-leave-active {
    transition: all 0.4s ease;
}

.fade-up-enter-from {
    opacity: 0;
    transform: translateY(10px);

}

.fade-up-enter-to {
    opacity: 1;
    transform: translateY(0);
}

.fade-up-leave-from {
    opacity: 1;
    transform: translateY(0);
}

.fade-up-leave-to {
    opacity: 0;
    transform: translateY(-10px);
}

.message.bot.thinking {
    width: 115px;
    display: flex;
    align-items: center;
}

.thinking .dots::after {
    content: '';
    animation: dots 1s steps(4, end) infinite;
}

.dots {
    opacity: 60%;
}

.thinking .dots {
    display: inline-block;
}

@keyframes dots {
    0% {
        content: '';
    }

    25% {
        content: '.';
    }

    50% {
        content: '..';
    }

    75% {
        content: '...';
    }

    100% {
        content: '';
    }
}
</style>