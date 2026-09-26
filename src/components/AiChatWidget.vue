<template>
  <div class="ai-widget-container">
    <!-- Floating Chat Window Card -->
    <q-slide-transition>
      <q-card v-if="isOpen" class="ai-chat-card shadow-24 rounded-borders">
        <!-- Card Header -->
        <q-card-section class="bg-gradient-header text-white q-py-sm flex items-center justify-between">
          <div class="flex items-center gap-2">
            <q-avatar size="36px" color="white" text-color="indigo-9" icon="smart_toy" />
            <div>
              <div class="text-subtitle1 text-bold leading-tight">Asistente IA Meteorológico</div>
              <div class="text-caption text-grey-3 flex items-center gap-1">
                <q-badge color="purple-4" label="Gemini 2.5 Flash" size="xs" />
                <span>• En vivo</span>
              </div>
            </div>
          </div>

          <div class="flex items-center">
            <q-btn
              flat
              round
              dense
              icon="analytics"
              color="white"
              title="Auditar anomalías ahora"
              :loading="isAuditing"
              @click="triggerAudit"
            >
              <q-tooltip>Auditar telemetría con IA</q-tooltip>
            </q-btn>
            <q-btn flat round dense icon="close" color="white" @click="isOpen = false" />
          </div>
        </q-card-section>

        <q-separator />

        <!-- Messages Body -->
        <q-card-section ref="chatContainer" class="ai-chat-body q-pa-md scroll">
          <div v-for="(msg, idx) in messages" :key="idx" class="q-mb-md">
            <q-chat-message
              :name="msg.sender === 'user' ? 'Usuario' : 'Asistente IA'"
              :avatar="msg.sender === 'user' ? undefined : 'https://cdn.quasar.dev/img/avatar.png'"
              :stamp="msg.time"
              :sent="msg.sender === 'user'"
              :bg-color="msg.sender === 'user' ? 'indigo-7' : 'grey-2'"
              :text-color="msg.sender === 'user' ? 'white' : 'grey-10'"
            >
              <div style="white-space: pre-line" v-html="formatMessage(msg.text)"></div>
            </q-chat-message>
          </div>

          <div v-if="isLoading" class="flex justify-start q-my-sm">
            <q-chat-message name="Asistente IA" bg-color="grey-2">
              <q-spinner-dots size="2rem" color="indigo-7" />
            </q-chat-message>
          </div>
        </q-card-section>

        <!-- Quick Suggestions Chips -->
        <div class="q-px-sm q-pb-xs flex gap-1 wrap bg-grey-1 border-top">
          <q-chip
            v-for="(chip, i) in quickPrompts"
            :key="i"
            clickable
            dense
            color="indigo-1"
            text-color="indigo-9"
            icon="auto_awesome"
            class="text-caption"
            @click="sendQuickPrompt(chip)"
          >
            {{ chip }}
          </q-chip>
        </div>

        <q-separator />

        <!-- Input Footer -->
        <q-card-actions class="q-pa-xs bg-white">
          <q-input
            v-model="inputPrompt"
            dense
            outlined
            placeholder="Consulte sobre el clima o los sensores..."
            class="full-width"
            :disable="isLoading"
            @keyup.enter="sendMessage"
          >
            <template #append>
              <q-btn
                round
                dense
                flat
                icon="send"
                color="indigo-7"
                :disable="!inputPrompt.trim() || isLoading"
                @click="sendMessage"
              />
            </template>
          </q-input>
        </q-card-actions>
      </q-card>
    </q-slide-transition>

    <!-- Floating Trigger Bubble Button -->
    <q-btn
      round
      size="lg"
      class="ai-bubble-btn shadow-12 animate-bounce-subtle"
      @click="toggleChat"
    >
      <q-avatar size="44px" icon="psychology" color="indigo-9" text-color="white" />
      <q-badge color="purple-7" floating class="q-mr-xs q-mt-xs" label="IA" />
      <q-tooltip anchor="left side" self="center right">
        Asistente IA Meteorológico (Gemini 2.5)
      </q-tooltip>
    </q-btn>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick } from 'vue';
import { api } from 'src/boot/axios';
import { useQuasar } from 'quasar';

const $q = useQuasar();
const isOpen = ref(false);
const isLoading = ref(false);
const isAuditing = ref(false);
const inputPrompt = ref('');
const chatContainer = ref<HTMLElement | null>(null);

interface Message {
  sender: 'user' | 'bot';
  text: string;
  time: string;
}

const messages = ref<Message[]>([
  {
    sender: 'bot',
    text: 'Bienvenido al Asistente Virtual de Inteligencia Artificial para la Red Meteorológica.\n\nPuedo analizar la telemetría en tiempo real, detectar anomalías técnicas o responder consultas ambientales.',
    time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
  },
]);

const quickPrompts = [
  '¿Existe alguna anomalía registrada?',
  'Resumen de temperatura y humedad',
  'Recomendaciones técnicas del clima',
];

function toggleChat() {
  isOpen.value = !isOpen.value;
  if (isOpen.value) {
    scrollToBottom();
  }
}

function formatMessage(text: string): string {
  return text.replace(/\*\*(.*?)\*\*/g, '<b>$1</b>');
}

function scrollToBottom() {
  nextTick(() => {
    if (chatContainer.value) {
      chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
    }
  });
}

async function sendMessage() {
  const text = inputPrompt.value.trim();
  if (!text || isLoading.value) return;

  const timeNow = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });

  messages.value.push({
    sender: 'user',
    text,
    time: timeNow,
  });

  inputPrompt.value = '';
  isLoading.value = true;
  scrollToBottom();

  try {
    const res = await api.post('/ia/chat', { prompt: text });
    const reply = res.data?.respuesta || 'No se obtuvo respuesta del servicio de IA.';

    messages.value.push({
      sender: 'bot',
      text: reply,
      time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
    });
  } catch (err) {
    console.error('Error al consultar chat IA:', err);
    messages.value.push({
      sender: 'bot',
      text: 'Ocurrió un problema al conectar con el servicio de IA. Por favor reintente.',
      time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
    });
  } finally {
    isLoading.value = false;
    scrollToBottom();
  }
}

function sendQuickPrompt(promptText: string) {
  inputPrompt.value = promptText;
  sendMessage();
}

async function triggerAudit() {
  if (isAuditing.value) return;
  isAuditing.value = true;

  $q.notify({
    type: 'info',
    message: 'Ejecutando auditoría de telemetría con Gemini IA...',
    icon: 'auto_awesome',
  });

  try {
    const res = await api.post('/ia/analizar');
    const resultado = res.data?.resultadoIA;

    if (resultado) {
      const resumenTexto = `**Resultado de Auditoría de IA:**\n\n**¿Anomalía?:** ${resultado.hayAnomalia ? 'SÍ' : 'NO'}\n**Nivel de Riesgo:** ${resultado.nivelRiesgo}\n\n**Resumen:** ${resultado.resumen}`;
      messages.value.push({
        sender: 'bot',
        text: resumenTexto,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
      });
      scrollToBottom();
    }

    $q.notify({
      type: 'positive',
      message: 'Auditoría completada exitosamente.',
      icon: 'check_circle',
    });
  } catch (err) {
    console.error('Error al ejecutar auditoría:', err);
    $q.notify({
      type: 'negative',
      message: 'Error al ejecutar la auditoría de IA.',
    });
  } finally {
    isAuditing.value = false;
  }
}
</script>

<style scoped>
.ai-widget-container {
  position: fixed;
  bottom: 24px;
  right: 24px;
  z-index: 9999;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
}

.ai-bubble-btn {
  background: linear-gradient(135deg, #3f51b5, #673ab7);
  color: white;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.ai-bubble-btn:hover {
  transform: scale(1.08);
  box-shadow: 0 8px 20px rgba(63, 81, 181, 0.4);
}

.ai-chat-card {
  width: 380px;
  max-width: calc(100vw - 32px);
  height: 520px;
  max-height: calc(100vh - 100px);
  display: flex;
  flex-direction: column;
  margin-bottom: 12px;
  overflow: hidden;
  border-radius: 16px;
}

.bg-gradient-header {
  background: linear-gradient(135deg, #1a237e, #311b92);
}

.ai-chat-body {
  flex: 1;
  background-color: #f8f9fa;
  overflow-y: auto;
}

.border-top {
  border-top: 1px solid #e0e0e0;
}

.gap-1 {
  gap: 4px;
}

.gap-2 {
  gap: 8px;
}
</style>
