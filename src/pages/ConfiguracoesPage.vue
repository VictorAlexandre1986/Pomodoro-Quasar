<template>
  <q-page class="q-pa-md">
    <q-card>
      <q-card-section>
        <div class="text-h6">Configurações do Pomodoro</div>
      </q-card-section>
      <q-card-section>
        <q-form @submit.prevent="salvarConfiguracoes">
          <q-input
            v-model.number="tempoTrabalho"
            type="number"
            label="Tempo de trabalho (minutos)"
            min="1"
            :rules="[val => val > 0 || 'Informe um valor válido']"
            class="q-mb-md"
          />
          <q-input
            v-model.number="tempoDescanso"
            type="number"
            label="Tempo de descanso (minutos)"
            min="1"
            :rules="[val => val > 0 || 'Informe um valor válido']"
            class="q-mb-md"
          />
          <q-btn label="Salvar" type="submit" color="primary" />
        </q-form>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useQuasar } from 'quasar'

const configKey = 'config-pomotime'
const tempoTrabalho = ref(25)
const tempoDescanso = ref(5)
const $q = useQuasar()

function carregarConfiguracoes() {
  const config = JSON.parse(localStorage.getItem(configKey) || '{}')
  if (config.tempoTrabalho) tempoTrabalho.value = config.tempoTrabalho
  if (config.tempoDescanso) tempoDescanso.value = config.tempoDescanso
}

function salvarConfiguracoes() {
  const config = {
    tempoTrabalho: tempoTrabalho.value,
    tempoDescanso: tempoDescanso.value
  }
  localStorage.setItem(configKey, JSON.stringify(config))
  $q.notify({ type: 'positive', message: 'Configurações salvas!' })
}

onMounted(() => {
  carregarConfiguracoes()
})
</script> 