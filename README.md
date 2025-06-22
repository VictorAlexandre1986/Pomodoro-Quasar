# PomoTime - Pomodoro PWA

Aplicativo Pomodoro moderno, responsivo e instalável (PWA), feito com [Quasar Framework](https://quasar.dev/) e Vue 3.  
Gerencie seu tempo de trabalho, intervalos, visualize relatórios de produtividade e exporte seus dados!

---

## Funcionalidades

- **Timer Pomodoro** totalmente configurável (tempo de trabalho e descanso).
- **Modo escuro/claro** com switch no topo.
- **Relatório detalhado** de tempo ativo, inativo (descanso) e ocioso (sem interação).
- **Dashboard com gráfico de barras** (produtividade por dia).
- **Calendário** para seleção e análise de produtividade diária.
- **Exportação de dados** em JSON, Excel (XLSX) e PDF, com nome de arquivo customizável.
- **PWA**: instalável, funciona offline, ícones para dispositivos e splash screen.
- **Notificações sonoras** ao fim de cada ciclo.
- **Menu lateral** para fácil navegação entre Início, Configurações e Relatório.

---

## Instalação

### Pré-requisitos

- Node.js 16, 18 ou 20
- npm >= 6.13.4 ou yarn >= 1.21.1

### Instale as dependências

```bash
npm install
# ou
yarn
```

---

## Desenvolvimento

Para rodar o app em modo desenvolvimento (hot reload):

```bash
npm run dev
# ou
quasar dev
```

---

## Build e Teste para PWA

### Gerar a build de produção como PWA:

```bash
quasar build -m pwa
```

Os arquivos finais estarão em `dist/pwa`.

### Servir o PWA localmente (simular produção):

```bash
quasar serve dist/pwa
```

Acesse o endereço exibido no terminal (ex: http://localhost:4000).

#### Testar em dispositivos na mesma rede
- Certifique-se de que o computador e o dispositivo estão na mesma rede Wi-Fi.
- Use o IP mostrado no terminal (ex: http://192.168.0.10:4000) no navegador do celular/tablet.
- Instale o app como PWA pelo navegador (ícone de instalar).

---

## Uso

### Timer Pomodoro

- Acesse a tela **Início** pelo menu lateral.
- Clique em **Iniciar** para começar o ciclo.
- O tempo de trabalho e descanso pode ser configurado na tela **Configurações**.
- Notificações sonoras e visuais são exibidas ao fim de cada ciclo.

### Configurações

- Defina o tempo de trabalho e descanso (em minutos).
- As alterações são salvas automaticamente e refletidas no timer.

### Relatório e Dashboard

- Veja o tempo **Ativo** (trabalhando), **Inativo** (descanso) e **Ocioso** (sem interação) por dia.
- Use o **calendário** para selecionar um dia e ver detalhes.
- O **gráfico de barras** mostra a produtividade de cada dia.
- **Exportação**: baixe seus dados em JSON, Excel ou PDF, escolhendo o nome do arquivo.

#### Legenda do Gráfico

- <span style="color:#21ba45"><b>Ativo</b></span>: enquanto está ativo
- <span style="color:#f44336"><b>Inativo</b></span>: é descanso
- <span style="color:#ff9800"><b>Ocioso</b></span>: quando está ativo mas não está fazendo nada

---

## PWA: Instalação e Uso Offline

- O app pode ser instalado no desktop ou celular (Android/iOS) via navegador (ícone de instalar).
- Funciona offline após o primeiro acesso.
- Ícones otimizados para diferentes dispositivos.

---

## Estrutura do Projeto

- `src/pages/IndexPage.vue`: Timer Pomodoro principal
- `src/pages/ConfiguracoesPage.vue`: Configurações de tempo
- `src/pages/RelatorioPage.vue`: Relatório, dashboard e exportação
- `src/layouts/MainLayout.vue`: Layout principal, menu e tema
- `src/App.vue`: Monitoramento de atividade, inatividade e ociosidade
- `src-pwa/manifest.json`: Manifesto PWA e ícones

---

## Personalização

- Ícones em `public/icons/`
- Variáveis de tema em `src/css/quasar.variables.scss`
- Sons em `src/sounds/`

---

## Dependências principais

- [Quasar Framework](https://quasar.dev/)
- [Vue 3](https://vuejs.org/)
- [Chart.js](https://www.chartjs.org/) (gráficos)
- [xlsx](https://github.com/SheetJS/sheetjs) (exportação Excel)
- [jspdf](https://github.com/parallax/jsPDF) + [jspdf-autotable](https://github.com/simonbengtsson/jsPDF-AutoTable) (exportação PDF)

---

## Licença

MIT

---

Se precisar de mais detalhes, prints, exemplos de uso ou instruções para deploy em servidores/cloud, é só pedir!
