<template>
  <q-page class="q-pa-md">
    <div class="row q-col-gutter-md items-start">
      <div class="col-12 col-md-8">
        <q-card class="q-mb-md">
          <q-card-section class="row items-center justify-between">
            <div class="text-h6">Relatório de Atividade</div>
            <div>
              <q-input v-model="nomeArquivo" label="Nome do arquivo" dense style="width: 200px;" class="q-mr-sm" />
              <q-btn label="Exportar JSON" color="primary" @click="exportarJson" icon="file_download" class="q-mr-sm" />
              <q-btn label="Exportar Excel" color="secondary" @click="exportarExcel" icon="file_download" class="q-mr-sm" />
              <q-btn label="Exportar PDF" color="accent" @click="exportarPdf" icon="picture_as_pdf" />
            </div>
          </q-card-section>
          <q-card-section>
            <q-table
              :rows="relatorio"
              :columns="columns"
              row-key="data"
              flat
            />
          </q-card-section>
        </q-card>
        <q-card class="q-mb-md">
          <q-card-section>
            <div class="text-h6">Tempo total de app aberto por dia</div>
          </q-card-section>
          <q-card-section>
            <canvas ref="openTimeChart"></canvas>
          </q-card-section>
        </q-card>
      </div>
      <div class="col-12 col-md-4">
        <q-card class="q-mb-md">
          <q-card-section>
            <div class="text-h6">Selecionar Dia</div>
          </q-card-section>
          <q-card-section>
            <q-date v-model="diaSelecionado" :options="datasDisponiveis" mask="YYYY-MM-DD" color="primary" />
            <div v-if="produtividadeSelecionada" class="q-mt-md">
              <div class="text-subtitle1">Produtividade em {{ diaSelecionado }}</div>
              <q-banner class="q-mt-sm" dense>
                Tempo Ativo: <b>{{ produtividadeSelecionada.ativo }} min</b><br>
                Tempo Inativo: <b>{{ produtividadeSelecionada.inativo }} min</b><br>
                Tempo Ocioso: <b>{{ produtividadeSelecionada.ocioso }} min</b>
              </q-banner>
            </div>
          </q-card-section>
        </q-card>
      </div>
    </div>
    <q-card class="q-mb-md">
      <q-card-section>
        <div class="text-h6">Dashboard - Gráfico de Barras</div>
      </q-card-section>
      <q-card-section>
        <canvas ref="barChart"></canvas>
      </q-card-section>
      <q-card-section class="q-pt-none">
        <div class="row items-center q-gutter-md">
          <div class="row items-center"><span style="display:inline-block;width:16px;height:16px;background:#21ba45;margin-right:6px;border-radius:3px;"></span> <b>Ativo</b>: enquanto está ativo</div>
          <div class="row items-center"><span style="display:inline-block;width:16px;height:16px;background:#f44336;margin-right:6px;border-radius:3px;"></span> <b>Inativo</b>: é descanso</div>
          <div class="row items-center"><span style="display:inline-block;width:16px;height:16px;background:#ff9800;margin-right:6px;border-radius:3px;"></span> <b>Ocioso</b>: quando está ativo mas não está fazendo nada</div>
        </div>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { date } from 'quasar'
import Chart from 'chart.js/auto'
import * as XLSX from 'xlsx'
import jsPDF from 'jspdf'
import 'jspdf-autotable'

const columns = [
  { name: 'data', label: 'Data', field: 'data', align: 'left' },
  { name: 'ativo', label: 'Tempo Ativo (min)', field: 'ativo', align: 'center' },
  { name: 'inativo', label: 'Tempo Inativo (min)', field: 'inativo', align: 'center' },
  { name: 'ocioso', label: 'Tempo Ocioso (min)', field: 'ocioso', align: 'center' }
]

const relatorio = ref([])
const barChart = ref(null)
let chartInstance = null
const diaSelecionado = ref('')
const produtividadeSelecionada = ref(null)
const nomeArquivo = ref('relatorio-atividade')
const openTimeChart = ref(null)
let openTimeChartInstance = null

function atualizarRelatorio() {
  const dados = JSON.parse(localStorage.getItem('relatorio-atividade') || '{}')
  relatorio.value = Object.keys(dados).map(data => ({
    data,
    ativo: Math.round((dados[data].ativo || 0) / 60000),
    inativo: Math.round((dados[data].inativo || 0) / 60000),
    ocioso: Math.round((dados[data].ocioso || 0) / 60000)
  })).sort((a, b) => a.data.localeCompare(b.data))
}

function atualizarGrafico() {
  if (!barChart.value) return
  if (chartInstance) chartInstance.destroy()
  const labels = relatorio.value.map(r => r.data)
  const ativos = relatorio.value.map(r => r.ativo)
  const inativos = relatorio.value.map(r => r.inativo)
  const ociosos = relatorio.value.map(r => r.ocioso)
  chartInstance = new Chart(barChart.value, {
    type: 'bar',
    data: {
      labels,
      datasets: [
        {
          label: 'Ativo (min)',
          data: ativos,
          backgroundColor: '#21ba45'
        },
        {
          label: 'Inativo (min)',
          data: inativos,
          backgroundColor: '#f44336'
        },
        {
          label: 'Ocioso (min)',
          data: ociosos,
          backgroundColor: '#ff9800'
        }
      ]
    },
    options: {
      responsive: true,
      plugins: {
        legend: { position: 'top' }
      },
      scales: {
        x: { title: { display: true, text: 'Dias do mês' } },
        y: { title: { display: true, text: 'Minutos' }, beginAtZero: true }
      }
    }
  })
}

function atualizarOpenTimeChart() {
  if (!openTimeChart.value) return
  if (openTimeChartInstance) openTimeChartInstance.destroy()
  const labels = relatorio.value.map(r => r.data)
  const totais = relatorio.value.map(r => r.ativo + r.ocioso)
  openTimeChartInstance = new Chart(openTimeChart.value, {
    type: 'bar',
    data: {
      labels,
      datasets: [
        {
          label: 'Tempo total aberto (min)',
          data: totais,
          backgroundColor: '#1976d2'
        }
      ]
    },
    options: {
      responsive: true,
      plugins: {
        legend: { display: false }
      },
      scales: {
        x: { title: { display: true, text: 'Dias do mês' } },
        y: { title: { display: true, text: 'Minutos' }, beginAtZero: true }
      }
    }
  })
}

function datasDisponiveis(dateStr) {
  return relatorio.value.some(r => r.data === dateStr)
}

function exportarJson() {
  const dados = JSON.parse(localStorage.getItem('relatorio-atividade') || '{}')
  const blob = new Blob([JSON.stringify(dados, null, 2)], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = nomeArquivo.value.trim() ? nomeArquivo.value + '.json' : 'relatorio-atividade.json'
  a.click()
  URL.revokeObjectURL(url)
}

function exportarExcel() {
  const dados = JSON.parse(localStorage.getItem('relatorio-atividade') || '{}')
  const rows = Object.keys(dados).map(data => ({
    Data: data,
    'Tempo Ativo (min)': Math.round((dados[data].ativo || 0) / 60000),
    'Tempo Inativo (min)': Math.round((dados[data].inativo || 0) / 60000),
    'Tempo Ocioso (min)': Math.round((dados[data].ocioso || 0) / 60000)
  }))
  const ws = XLSX.utils.json_to_sheet(rows)
  const wb = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(wb, ws, 'Relatorio')
  const wbout = XLSX.write(wb, { bookType: 'xlsx', type: 'array' })
  const blob = new Blob([wbout], { type: 'application/octet-stream' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = nomeArquivo.value.trim() ? nomeArquivo.value + '.xlsx' : 'relatorio-atividade.xlsx'
  a.click()
  URL.revokeObjectURL(url)
}

function exportarPdf() {
  const dados = JSON.parse(localStorage.getItem('relatorio-atividade') || '{}')
  const rows = Object.keys(dados).map(data => ([
    data,
    Math.round((dados[data].ativo || 0) / 60000),
    Math.round((dados[data].inativo || 0) / 60000),
    Math.round((dados[data].ocioso || 0) / 60000)
  ]))
  const doc = new jsPDF()
  doc.text('Relatório de Atividade', 14, 16)
  doc.autoTable({
    head: [['Data', 'Tempo Ativo (min)', 'Tempo Inativo (min)', 'Tempo Ocioso (min)']],
    body: rows,
    startY: 22
  })
  const nome = nomeArquivo.value.trim() ? nomeArquivo.value + '.pdf' : 'relatorio-atividade.pdf'
  doc.save(nome)
}

watch(relatorio, () => {
  atualizarGrafico()
  atualizarOpenTimeChart()
})

watch(diaSelecionado, (novoDia) => {
  produtividadeSelecionada.value = relatorio.value.find(r => r.data === novoDia) || null
})

onMounted(() => {
  atualizarRelatorio()
  atualizarGrafico()
  atualizarOpenTimeChart()
})
</script> 