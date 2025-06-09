<script setup>
import { ref, computed } from 'vue'

// Game board: 3x3 array, '' | 'X' | 'O'
const board = ref([
  ['', '', ''],
  ['', '', ''],
  ['', '', ''],
])

const players = [
  { label: 'X', name: 'Player 1', color: '#2196f3' }, // Accent color
  { label: 'O', name: 'Player 2', color: '#000000' }, // Secondary (black)
]
const currentPlayer = ref(0) // 0: X, 1: O
const winner = ref(null) // null/'X'/'O'/'draw'
const showInstructions = ref(false)

const isCellEmpty = (row, col) => board.value[row][col] === '' && !winner.value

// PUBLIC_INTERFACE
function handleCellClick(row, col) {
  // Ignore clicks if cell filled or game over
  if (!isCellEmpty(row, col)) return
  board.value[row][col] = players[currentPlayer.value].label
  if (checkWinner()) {
    winner.value = players[currentPlayer.value].label
  } else if (isBoardFull()) {
    winner.value = 'draw'
  } else {
    currentPlayer.value = 1 - currentPlayer.value
  }
}

// PUBLIC_INTERFACE
function restartGame() {
  winner.value = null
  board.value = [
    ['', '', ''],
    ['', '', ''],
    ['', '', ''],
  ]
  currentPlayer.value = 0
}

// PUBLIC_INTERFACE
function isBoardFull() {
  return board.value.flat().every(cell => cell !== '')
}

// PUBLIC_INTERFACE
function checkWinner() {
  const winLines = [
    // Rows
    [[0,0], [0,1], [0,2]],
    [[1,0], [1,1], [1,2]],
    [[2,0], [2,1], [2,2]],
    // Cols
    [[0,0], [1,0], [2,0]],
    [[0,1], [1,1], [2,1]],
    [[0,2], [1,2], [2,2]],
    // Diags
    [[0,0], [1,1], [2,2]],
    [[0,2], [1,1], [2,0]],
  ]
  for (const line of winLines) {
    const [a, b, c] = line
    const cellA = board.value[a[0]][a[1]]
    const cellB = board.value[b[0]][b[1]]
    const cellC = board.value[c[0]][c[1]]
    if (cellA && cellA === cellB && cellA === cellC) {
      return true
    }
  }
  return false
}

const playerTurnText = computed(() => {
  if (winner.value) {
    if (winner.value === 'draw') return 'It\'s a draw!'
    const winnerObj = players.find(p => p.label === winner.value)
    return `${winnerObj.name} (${winnerObj.label}) wins!`
  }
  const pl = players[currentPlayer.value]
  return `Turn: ${pl.name} (${pl.label})`
})

const playerTurnColor = computed(() => {
  if (winner.value === 'draw') return '#808080'
  if (winner.value) {
    return players.find(p=>p.label===winner.value).color
  }
  return players[currentPlayer.value].color
})
</script>

<template>
  <div class="ttt-container">
    <div class="player-turn" :style="{ color: playerTurnColor }">
      {{ playerTurnText }}
    </div>
    <div class="ttt-board" role="grid" aria-label="Tic Tac Toe grid">
      <div
        v-for="(row, rowIndex) in board"
        :key="`row-${rowIndex}`"
        class="ttt-row"
        role="row"
      >
        <button
          v-for="(cell, colIndex) in row"
          :key="`cell-${rowIndex}-${colIndex}`"
          class="ttt-cell"
          :aria-label="cell ? `cell with ${cell}` : 'empty cell'"
          :disabled="!!cell || winner"
          @click="handleCellClick(rowIndex, colIndex)"
          :style="{
            color: cell === 'X' ? players[0].color : (cell === 'O' ? players[1].color : '#222'),
            backgroundColor: cell ? '#f5f7fb' : '#fff'
          }"
        >
          {{ cell }}
        </button>
      </div>
    </div>
    <div class="ttt-buttons">
      <button class="ttt-action restart" @click="restartGame">
        Restart
      </button>
      <button class="ttt-action instructions" @click="showInstructions = true">
        Instructions
      </button>
    </div>
    <div v-if="showInstructions" class="ttt-modal">
      <div class="ttt-modal-content">
        <div class="ttt-modal-header">How to Play</div>
        <div class="ttt-modal-body">
          <ul>
            <li>Two players alternate turns on a 3x3 grid.</li>
            <li>First player places X, second places O.</li>
            <li>Get three of your marks in a row — horizontally, vertically, or diagonally — to win.</li>
            <li>If all 9 squares are filled and nobody wins, it’s a draw.</li>
            <li>Click "Restart" to play again.</li>
          </ul>
        </div>
        <button class="ttt-modal-close" @click="showInstructions = false">
          Close
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.ttt-container {
  max-width: 400px;
  margin: 2.5rem auto;
  padding: 2rem 1rem 1.5rem 1rem;
  background: #fff;
  border-radius: 18px;
  box-shadow: 0 4px 16px 0 rgba(33,40,60,.07);
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: inherit;
}
.player-turn {
  font-size: 1.35rem;
  font-weight: 600;
  text-align: center;
  margin-bottom: 1.25rem;
  min-height: 2em;
  transition: color 0.3s;
}
.ttt-board {
  display: flex;
  flex-direction: column;
  gap: 0;
}
.ttt-row {
  display: flex;
  flex-direction: row;
}
.ttt-cell {
  width: 70px;
  height: 70px;
  margin: 0.15rem;
  font-size: 2.3rem;
  font-weight: 700;
  text-align: center;
  background: #fff;
  border: 2px solid #2196f3;
  border-radius: 8px;
  outline: none;
  cursor: pointer;
  transition: background 0.2s, color 0.2s;
  user-select: none;
}
.ttt-cell:disabled {
  opacity: 1;
  background: #f5f7fb;
  cursor: default;
}
.ttt-cell:not(:disabled):hover {
  background: #e4ecfa;
}

.ttt-buttons {
  display: flex;
  gap: 1.5em;
  margin-top: 1.5em;
}
.ttt-action {
  background: #2196f3;
  color: #fff;
  border: none;
  border-radius: 7px;
  padding: 0.65em 1.4em;
  font-size: 1.08em;
  font-weight: 500;
  cursor: pointer;
  outline: none;
  box-shadow: 0 1px 3px rgba(0,0,0,0.06);
  transition: background 0.18s;
}
.ttt-action.restart {
  background: #2196f3;
}
.ttt-action.instructions {
  background: #000;
  color: #fff;
}
.ttt-action:active {
  background: #1760a7;
}

.ttt-modal {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.23);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9000;
}
.ttt-modal-content {
  padding: 2.2em 2em;
  border-radius: 1em;
  background: #fff;
  max-width: 330px;
  box-shadow: 0 5px 30px 0 rgba(33,40,60,.17);
  text-align: left;
  color: #151b24;
}
.ttt-modal-header {
  font-size: 1.16em;
  font-weight: 600;
  margin-bottom: 0.7em;
  color: #2196f3;
}
.ttt-modal-body {
  font-size: 1.04em;
  line-height: 1.58;
  margin-bottom: 1.5em;
}
.ttt-modal-close {
  background: #2196f3;
  color: #fff;
  border: none;
  border-radius: 6px;
  padding: 0.5em 1.1em;
  font-size: 1em;
  cursor: pointer;
  margin-top: 0.2em;
}
@media (max-width: 600px) {
  .ttt-container { max-width: 98vw; padding: 1em 0.2em; }
  .ttt-cell { width: 44px; height: 44px; font-size: 1.35em; }
  .ttt-modal-content { max-width: 93vw; }
}
</style>
