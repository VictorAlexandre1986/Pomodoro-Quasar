<template>
  <router-view />
</template>

<script setup>
defineOptions({
  name: 'App'
});

import { onMounted, onUnmounted } from 'vue'

let tempoAtivo = 0
let tempoInativo = 0
let tempoOcioso = 0
let ultimaTroca = Date.now()
let ultimoInput = Date.now()
let visivel = document.visibilityState === 'visible'
let ocioso = false
const TEMPO_OCIOSO_MS = 60000 // 1 minuto sem interação

function salvarTempo() {
  const hoje = new Date().toISOString().slice(0, 10)
  const dados = JSON.parse(localStorage.getItem('relatorio-atividade') || '{}')
  if (!dados[hoje]) dados[hoje] = { ativo: 0, inativo: 0, ocioso: 0 }
  dados[hoje].ativo += tempoAtivo
  dados[hoje].inativo += tempoInativo
  dados[hoje].ocioso += tempoOcioso
  localStorage.setItem('relatorio-atividade', JSON.stringify(dados))
  tempoAtivo = 0
  tempoInativo = 0
  tempoOcioso = 0
}

function handleVisibilityChange() {
  const agora = Date.now()
  if (document.visibilityState === 'visible') {
    tempoInativo += agora - ultimaTroca
  } else {
    if (ocioso) {
      tempoOcioso += agora - ultimaTroca
    } else {
      tempoAtivo += agora - ultimaTroca
    }
  }
  ultimaTroca = agora
}

function handleUserInput() {
  if (ocioso) {
    // Saiu do modo ocioso
    const agora = Date.now()
    tempoOcioso += agora - ultimaTroca
    ultimaTroca = agora
    ocioso = false
  }
  ultimoInput = Date.now()
}

function checarOciosidade() {
  const agora = Date.now()
  if (document.visibilityState === 'visible') {
    if (!ocioso && agora - ultimoInput > TEMPO_OCIOSO_MS) {
      // Entrou em modo ocioso
      tempoAtivo += agora - ultimaTroca
      ultimaTroca = agora
      ocioso = true
    }
    if (ocioso && agora - ultimoInput <= TEMPO_OCIOSO_MS) {
      // Saiu do modo ocioso
      tempoOcioso += agora - ultimaTroca
      ultimaTroca = agora
      ocioso = false
    }
  }
}

onMounted(() => {
  document.addEventListener('visibilitychange', handleVisibilityChange)
  window.addEventListener('beforeunload', salvarTempo)
  window.addEventListener('mousemove', handleUserInput)
  window.addEventListener('keydown', handleUserInput)
  setInterval(() => {
    const agora = Date.now()
    checarOciosidade()
    if (document.visibilityState === 'visible') {
      if (ocioso) {
        tempoOcioso += agora - ultimaTroca
      } else {
        tempoAtivo += agora - ultimaTroca
      }
    } else {
      tempoInativo += agora - ultimaTroca
    }
    ultimaTroca = agora
    salvarTempo()
  }, 60000) // salva a cada minuto
})

onUnmounted(() => {
  salvarTempo()
  document.removeEventListener('visibilitychange', handleVisibilityChange)
  window.removeEventListener('beforeunload', salvarTempo)
  window.removeEventListener('mousemove', handleUserInput)
  window.removeEventListener('keydown', handleUserInput)
})
</script>
