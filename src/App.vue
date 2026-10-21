
<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue'

type Difficulty = 'Fácil' | 'Médio' | 'Difícil'

const selectedTheme = ref('verde')
const themeMenuOpen = ref(false)
const themePickerRef = ref<HTMLElement | null>(null)

const themes = [
  { id: 'verde', name: 'Verde', accent: '#b8f36b' },
  { id: 'roxo', name: 'Roxo Arcade', accent: '#d19aff' },
  { id: 'laranja', name: 'Laranja Solar', accent: '#ffb454' },
  { id: 'azul-claro', name: 'Azul Gelo', accent: '#2479b8' },
  { id: 'grafite', name: 'Grafite', accent: '#45e0d0' },
  { id: 'areia', name: 'Areia', accent: '#8a6238' },
  { id: 'rosa', name: 'Rosa', accent: '#f08ab5' },
]

const boardSize = ref(5)
const difficulty = ref<Difficulty>('Fácil')

const moveRanges: Record<number, Record<Difficulty, { min: number; max: number }>> = {
  4: {
    Fácil: { min: 3, max: 4 },
    Médio: { min: 5, max: 6 },
    Difícil: { min: 7, max: 8 },
  },
  5: {
    Fácil: { min: 4, max: 6 },
    Médio: { min: 7, max: 10 },
    Difícil: { min: 11, max: 16 },
  },
  6: {
    Fácil: { min: 5, max: 8 },
    Médio: { min: 9, max: 15 },
    Difícil: { min: 16, max: 25 },
  },
  7: {
    Fácil: { min: 6, max: 10 },
    Médio: { min: 11, max: 19 },
    Difícil: { min: 20, max: 32 },
  },
}
const lights = ref<boolean[]>([])
const moves = ref(0)
const elapsedSeconds = ref(0)
const started = ref(false)
const completed = ref(false)
const showCompletionModal = ref(false)
const initialLights = ref<boolean[]>([])
const hintIndex = ref<number | null>(null)
const solution = ref<number[]>([])
const hintRequested = ref(false)
const hintsUsed = ref(0)
const optimalMoveCount = ref(0)

let intervalId: number | undefined

const totalLightsOn = computed(
  () => lights.value.filter(Boolean).length,
)
const difficultyRange = computed(() => moveRanges[boardSize.value][difficulty.value])

const formattedTime = computed(() => {
  const minutes = Math.floor(elapsedSeconds.value / 60)
  const seconds = elapsedSeconds.value % 60

  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`
})

function solveBoard(board: boolean[], size: number): number[] | null {
  let bestSolution: number[] | null = null

  // Cada combinação da primeira linha determina uma solução possível.
  // Avaliamos todas e guardamos a solução com menos movimentos.
  for (let firstRow = 0; firstRow < (1 << size); firstRow++) {
    const state = [...board]
    const presses = Array<boolean>(size * size).fill(false)

    function press(row: number, col: number) {
      const index = row * size + col
      presses[index] = !presses[index]

      const neighbors: [number, number][] = [
        [row, col],
        [row - 1, col],
        [row + 1, col],
        [row, col - 1],
        [row, col + 1],
      ]

      for (const [r, c] of neighbors) {
        if (r >= 0 && r < size && c >= 0 && c < size) {
          state[r * size + c] = !state[r * size + c]
        }
      }
    }

    for (let col = 0; col < size; col++) {
      if ((firstRow & (1 << col)) !== 0) press(0, col)
    }

    for (let row = 1; row < size; row++) {
      for (let col = 0; col < size; col++) {
        if (state[(row - 1) * size + col]) press(row, col)
      }
    }

    if (state.every((light) => !light)) {
      const candidate = presses
        .map((pressed, index) => (pressed ? index : -1))
        .filter((index) => index !== -1)

      if (bestSolution === null || candidate.length < bestSolution.length) {
        bestSolution = candidate
      }
    }
  }

  return bestSolution
}

function showHint() {
  if (completed.value) return

  hintsUsed.value++
  hintRequested.value = true

  const result = solveBoard(lights.value, boardSize.value)

  solution.value = result ?? []
  hintIndex.value = solution.value[0] ?? null
}

function clearHint() {
  hintIndex.value = null
  solution.value = []
  hintRequested.value = false
}

function stopTimer() {
  if (intervalId !== undefined) {
    window.clearInterval(intervalId)
    intervalId = undefined
  }
}

function startTimer() {
  if (intervalId !== undefined) return

  intervalId = window.setInterval(() => {
    elapsedSeconds.value++
  }, 1000)
}

function toggleLight(index: number) {
  if (completed.value) return

  clearHint()

  if (!started.value) {
    started.value = true
    startTimer()
  }

  const row = Math.floor(index / boardSize.value)
  const col = index % boardSize.value
  const affected = [index]

  if (row > 0) affected.push(index - boardSize.value)
  if (row < boardSize.value - 1) affected.push(index + boardSize.value)
  if (col > 0) affected.push(index - 1)
  if (col < boardSize.value - 1) affected.push(index + 1)

  for (const position of affected) {
    lights.value[position] = !lights.value[position]
  }

  moves.value++

  if (totalLightsOn.value === 0) {
    completed.value = true
    showCompletionModal.value = true
    stopTimer()
  }
}

function applyMoveWithoutCounting(index: number) {
  const row = Math.floor(index / boardSize.value)
  const col = index % boardSize.value
  const affected = [index]

  if (row > 0) affected.push(index - boardSize.value)
  if (row < boardSize.value - 1) affected.push(index + boardSize.value)
  if (col > 0) affected.push(index - 1)
  if (col < boardSize.value - 1) affected.push(index + 1)

  for (const position of affected) {
    lights.value[position] = !lights.value[position]
  }
}

function newGame() {
  stopTimer()
  hintsUsed.value = 0

  const total = boardSize.value * boardSize.value
  const range = moveRanges[boardSize.value][difficulty.value]
  let generatedBoard: boolean[] | null = null
  let generatedSolution: number[] | null = null
  let closestBoard: boolean[] | null = null
  let closestSolution: number[] | null = null
  let closestDistance = Number.POSITIVE_INFINITY

  // Tentamos tabuleiros até encontrar um cuja solução mínima esteja
  // dentro da faixa definida para o tamanho e a dificuldade escolhidos.
  for (let attempt = 0; attempt < 1500; attempt++) {
    const candidateBoard = Array<boolean>(total).fill(false)
    const positions = Array.from({ length: total }, (_, index) => index)

    // Embaralha as posições para evitar movimentos repetidos no embaralhamento.
    for (let i = positions.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1))
      ;[positions[i], positions[j]] = [positions[j], positions[i]]
    }

    const scrambleCount =
      Math.floor(Math.random() * (range.max - range.min + 1)) + range.min

    lights.value = candidateBoard
    for (let i = 0; i < scrambleCount; i++) {
      applyMoveWithoutCounting(positions[i])
    }

    const candidateSolution = solveBoard(lights.value, boardSize.value)
    if (candidateSolution === null) continue

    const distance = candidateSolution.length < range.min
      ? range.min - candidateSolution.length
      : candidateSolution.length > range.max
        ? candidateSolution.length - range.max
        : 0

    if (distance < closestDistance) {
      closestDistance = distance
      closestBoard = [...lights.value]
      closestSolution = candidateSolution
    }

    if (distance === 0) {
      generatedBoard = [...lights.value]
      generatedSolution = candidateSolution
      break
    }
  }

  // Salvaguarda: se a faixa não for atingida após muitas tentativas,
  // utiliza o tabuleiro mais próximo encontrado para não bloquear o jogo.
  if (generatedBoard === null || generatedSolution === null) {
    generatedBoard = closestBoard ?? Array<boolean>(total).fill(false)
    generatedSolution = closestSolution ?? []
  }

  lights.value = generatedBoard
  optimalMoveCount.value = generatedSolution.length
  initialLights.value = [...generatedBoard]

  moves.value = 0
  elapsedSeconds.value = 0
  started.value = false
  completed.value = false
  showCompletionModal.value = false
  clearHint()
}

function restartGame() {
  stopTimer()
  hintsUsed.value = 0
  lights.value = [...initialLights.value]
  moves.value = 0
  elapsedSeconds.value = 0
  started.value = false
  completed.value = false
  showCompletionModal.value = false

  clearHint()
}

function changeSettings() {
  newGame()
}

function handleOutsideThemeClick(event: PointerEvent) {
  if (!themePickerRef.value?.contains(event.target as Node)) {
    themeMenuOpen.value = false
  }
}

function handleThemeEscape(event: KeyboardEvent) {
  if (event.key === 'Escape') themeMenuOpen.value = false
}

onMounted(() => {
  document.addEventListener('pointerdown', handleOutsideThemeClick)
  document.addEventListener('keydown', handleThemeEscape)
})

onUnmounted(() => {
  stopTimer()
  document.removeEventListener('pointerdown', handleOutsideThemeClick)
  document.removeEventListener('keydown', handleThemeEscape)
})

newGame()
</script>

<template>
  <main class="app-shell" :data-theme="selectedTheme">
    <header class="topbar">
      <a class="brand" href="#" aria-label="Apague as Luzes">
        <span class="brand-mark">✳</span>
        <span>PUZZLE<span class="brand-accent">LAB</span></span>
      </a>

    <section ref="themePickerRef" class="theme-picker" aria-label="Seletor de cores">
      <button
        type="button"
        class="theme-toggle"
        :aria-expanded="themeMenuOpen"
        @click="themeMenuOpen = !themeMenuOpen"
      >
        <span class="dev-badge">DEV</span>
        <span>Tema</span>
        <span class="current-theme-swatch" :style="{ '--theme-swatch': themes.find(theme => theme.id === selectedTheme)?.accent }"></span>
        <span class="theme-chevron" :class="{ open: themeMenuOpen }">⌄</span>
      </button>

      <div v-if="themeMenuOpen" class="theme-options">
        <button
          v-for="theme in themes"
          :key="theme.id"
          type="button"
          class="theme-option"
          :class="{ active: selectedTheme === theme.id }"
          :style="{ '--theme-swatch': theme.accent }"
          :aria-label="`Tema ${theme.name}`"
          :aria-pressed="selectedTheme === theme.id"
          @click="selectedTheme = theme.id; themeMenuOpen = false"
        >
          <span class="theme-swatch"></span>
          <span class="theme-name">{{ theme.name }}</span>
        </button>
      </div>
    </section>

            <span class="topbar-tag">DESAFIO DE LÓGICA</span>
    </header>

    <section class="hero">
      <p class="eyebrow">DESAFIE SUA MENTE</p>
      <h1>Apague as <span>luzes.</span></h1>
      <p class="subtitle">
        Cada movimento muda as luzes ao redor.
        Apague todas usando o menor número de movimentos.
      </p>
    </section>

    <section class="game-layout">
      <div class="board-panel">
        <div class="panel-heading">
          <div>
            <span class="section-label">TABULEIRO</span>
            <h2>{{ boardSize }} × {{ boardSize }}</h2>
          </div>

          <span class="live-indicator">
            <span class="live-dot"></span>
            {{ completed ? 'CONCLUÍDO' : 'EM JOGO' }}
          </span>
        </div>

        <div
          class="board"
          :style="{
            gridTemplateColumns: `repeat(${boardSize}, minmax(0, 1fr))`,
          }"
          role="group"
          aria-label="Tabuleiro de luzes"
        >
          <button
            v-for="(isOn, index) in lights"
            :key="`${boardSize}-${index}`"
            class="light-tile"
            :class="{
              'is-on': isOn,
              hinted: hintIndex === index,
            }"
            :aria-label="`Luz ${index + 1}, ${isOn ? 'acesa' : 'apagada'}${hintIndex === index ? ', dica sugerida' : ''}`"
            :aria-pressed="isOn"
            :disabled="completed"
            @click="toggleLight(index)"
          >
            <span class="tile-core"></span>
          </button>
        </div>

        <p v-if="!completed" class="board-tip">
          <span>✦</span>
          Clique em uma luz para alternar ela e as vizinhas.
        </p>
      </div>

      <aside class="sidebar">
        <section class="stats-card">
          <span class="section-label">SUA JORNADA</span>

          <div class="main-stat">
            <span>Tempo decorrido</span>
            <strong>{{ formattedTime }}</strong>
          </div>

          <div class="stats-divider"></div>

          <div class="stat-row">
            <span>Movimentos</span>
            <strong>{{ moves }}</strong>
          </div>

          <div class="stat-row">
            <span>Luzes acesas</span>
            <strong>{{ totalLightsOn }} / {{ lights.length }}</strong>
          </div>

          <div class="stat-row">
            <span>Luzes apagadas</span>
            <strong>{{ lights.length - totalLightsOn }} / {{ lights.length }}</strong>
          </div>

          <div class="stat-row">
            <span>Dicas utilizadas</span>
            <strong>{{ hintsUsed }}</strong>
          </div>
        </section>

        <section class="controls-card">
          <span class="section-label">CONFIGURAÇÕES</span>

          <label for="board-size">Tamanho do tabuleiro</label>
          <select
            id="board-size"
            v-model.number="boardSize"
            @change="changeSettings"
          >
            <option :value="4">4 × 4</option>
            <option :value="5">5 × 5</option>
            <option :value="6">6 × 6</option>
            <option :value="7">7 × 7</option>
          </select>

          <label for="difficulty">Dificuldade</label>
          <select
            id="difficulty"
            v-model="difficulty"
            @change="changeSettings"
          >
            <option>Fácil</option>
            <option>Médio</option>
            <option>Difícil</option>
          </select>
          <p class="difficulty-hint">
            Este tabuleiro pode ser resolvido em no mínimo {{ optimalMoveCount }} movimentos.
            Faixa do nível: {{ difficultyRange.min }}–{{ difficultyRange.max }} movimentos mínimos.
          </p>

          <button class="primary-button" @click="newGame">
            <span>↻</span> Novo jogo
          </button>

          <button class="secondary-button" @click="restartGame">
            Reiniciar partida
          </button>

          <button
            class="secondary-button"
            :disabled="completed"
            @click="showHint"
          >
            💡 Dica
          </button>

          <p v-if="hintIndex !== null" class="hint-feedback">
            💡 Experimente clicar na luz destacada!
          </p>

          <p
            v-else-if="hintRequested && solution.length === 0 && !completed"
            class="hint-feedback"
          >
            Não foi encontrada uma solução para este tabuleiro.
          </p>

          <p
            v-else-if="hintRequested && solution.length === 0 && completed"
            class="hint-feedback"
          >
            Parabéns! O tabuleiro já está resolvido.
          </p>

          <p
            v-if="hintIndex === null && hintRequested && solution.length > 0"
            class="hint-feedback"
          >
            Não há mais jogadas sugeridas.
          </p>
        </section>

        <section class="hint-card">
          <span class="hint-icon">✳</span>
          <div>
            <strong>Dica estratégica</strong>
            <p>
              Um clique afeta a própria casa e as vizinhas na vertical
              e horizontal. As diagonais não mudam.
            </p>
          </div>
        </section>
      </aside>
    </section>

    
    <Transition name="completion-fade">
      <div
        v-if="showCompletionModal"
        class="completion-overlay"
        role="presentation"
        @click.self="showCompletionModal = false"
        @keydown.esc="showCompletionModal = false"
      >
        <section
          class="completion-modal"
          role="dialog"
          aria-modal="true"
          aria-labelledby="completion-title"
        >
          <button
            type="button"
            class="completion-close"
            aria-label="Fechar resultado"
            @click="showCompletionModal = false"
          >×</button>

          <div class="completion-celebration" aria-hidden="true">
            <span class="celebration-orbit orbit-one"></span>
            <span class="celebration-orbit orbit-two"></span>
            <span class="celebration-star">✦</span>
          </div>

          <p class="completion-eyebrow">MISSÃO CUMPRIDA</p>
          <h2 id="completion-title">Parabéns!</h2>
          <p class="completion-description">
            Você apagou todas as luzes. Mandou muito bem!
          </p>

          <div class="completion-result">
            <div class="completion-primary-stat">
              <span>SEU TEMPO</span>
              <strong>{{ formattedTime }}</strong>
            </div>
            <div class="completion-primary-stat">
              <span>MOVIMENTOS</span>
              <strong>{{ moves }}</strong>
            </div>
          </div>

          <div class="completion-details">
            <div class="completion-detail">
              <span class="completion-detail-icon">▦</span>
              <span>Tabuleiro</span>
              <strong>{{ boardSize }} × {{ boardSize }}</strong>
            </div>
            <div class="completion-detail">
              <span class="completion-detail-icon">◆</span>
              <span>Dificuldade</span>
              <strong>{{ difficulty }}</strong>
            </div>
            <div class="completion-detail">
              <span class="completion-detail-icon">💡</span>
              <span>Dicas utilizadas</span>
              <strong>{{ hintsUsed }}</strong>
            </div>
            <div class="completion-detail">
              <span class="completion-detail-icon">◎</span>
              <span>Solução mínima</span>
              <strong>{{ optimalMoveCount }} movimentos</strong>
            </div>
            <div class="completion-detail">
              <span class="completion-detail-icon">✓</span>
              <span>Luzes apagadas</span>
              <strong>{{ lights.length }} / {{ lights.length }}</strong>
            </div>
          </div>

          <div class="completion-actions">
            <button type="button" class="primary-button" @click="newGame">
              ↻ Jogar novamente
            </button>
            <button
              type="button"
              class="secondary-button"
              @click="showCompletionModal = false"
            >
              Ver tabuleiro
            </button>
          </div>
          <p class="completion-footer">CADA MOVIMENTO CONTA ✦ PUZZLELAB</p>
        </section>
      </div>
    </Transition>

    <footer class="footer">
      <p>Desenvolvido por <strong>Ricardo Spicka</strong> © 2026</p>

      <div class="footer-links">
        <a href="https://github.com/RicardoSpicka" target="_blank" rel="noopener noreferrer">
          GitHub
        </a>
        <span>·</span>
        <a href="https://www.linkedin.com/in/ricardo-spicka-b79049160" target="_blank" rel="noopener noreferrer">
          LinkedIn
        </a>
        <span>·</span>
        <span>Apague as Luzes</span>
      </div>
    </footer>

  </main>
</template>
