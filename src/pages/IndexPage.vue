<template>
  <q-page class="flex flex-center">
    <div class="text-center">
      <q-card class="q-pa-md">
        <q-card-section>
          <div class="text-h5">{{ isBreak ? 'Intervalo' : 'Trabalho' }}</div>
          <div class="text-h4 q-my-md">{{ formatTime(time) }}</div>
        </q-card-section>
        <q-card-actions align="center">
          <q-btn @click="startTimer" label="Iniciar" :disable="isRunning" />
          <q-btn @click="pauseTimer" label="Pausar" :disable="!isRunning" />
          <q-btn @click="resetTimer" label="Resetar" />
        </q-card-actions>
      </q-card>
      <audio ref="audio" preload="auto">
          <source src="../sounds/alerta.mp3" type="audio/mpeg" />
      </audio>

    </div>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useQuasar } from 'quasar';

const time = ref(25 * 60); // 25 minutos de trabalho
const isBreak = ref(false);
const isRunning = ref(false);
let timer = null;

const $q = useQuasar();
const audio = ref(null); // Referência para o áudio

onMounted(() => {
  // Checa se o audio está carregado corretamente
  if (!audio.value) {
    console.error('Áudio não está carregado corretamente!');
  }
});

const startTimer = () => {
  if (!isRunning.value) {
    isRunning.value = true;
    timer = setInterval(() => {
      if (time.value > 0) {
        time.value--;
      } else {
        if ($q.notify) {
          $q.notify({
            color: 'positive',
            message: isBreak.value ? 'Volte ao trabalho!' : 'Hora do intervalo!',
            timeout: 2000,
          });
        } else {
          console.error('Notificação Quasar não está funcionando!');
        }

        playSound(); // Toca o som quando o tempo terminar

        isBreak.value = !isBreak.value;
        time.value = isBreak.value ? 5 * 60 : 25 * 60; // 5 minutos de intervalo ou 25 minutos de trabalho
      }
    }, 1000);
  }
};

const pauseTimer = () => {
  isRunning.value = false;
  clearInterval(timer);
};

const resetTimer = () => {
  isRunning.value = false;
  clearInterval(timer);
  isBreak.value = false;
  time.value = 25 * 60; // Reset para 25 minutos
};

const formatTime = (seconds) => {
  const minutes = Math.floor(seconds / 60);
  const secs = seconds % 60;
  return `${String(minutes).padStart(2, '0')}:${String(secs).padStart(2, '0')}`;
};

const playSound = () => {
  if (audio.value) {
    const playPromise = audio.value.play();

    // Verifica se o navegador bloqueou a reprodução automática de som
    if (playPromise !== undefined) {
      playPromise
        .then(() => {
          console.log('Som tocando.');
        })
        .catch((error) => {
          console.error('Erro ao tentar tocar o som:', error);
        });
    }
  }
};
</script>

<style scoped>
.text-center {
  max-width: 300px;
  margin: 0 auto;
  box-shadow: 1px 1px 10px 1px rgba(0, 0, 0, 0.3);
}
</style>
