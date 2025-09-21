<template>
  <div class="game-container">
    <header>
      <h1>SWAPPLE</h1>
      <div class="date-label">{{ date }}</div>
    </header>

    <div class="stats-container">
      <div class="stat-box">
        <span class="stat-number">{{ counter }}</span>
        <span class="stat-label">Moves</span>
      </div>
      <div class="button-container">
        <button class="actionbutton" @click="undo" title="Undo last move" aria-label="Undo last move">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24">
            <path fill="currentColor" d="M12.5 8c-2.65 0-5.05 1-6.9 2.6L2 7v9h9l-3.62-3.62c1.39-1.16 3.16-1.88 5.12-1.88 3.54 0 6.55 2.31 7.6 5.5l2.37-.78C21.08 11.03 17.15 8 12.5 8z"/>
          </svg>
          <span class="button-label">Undo</span>
        </button>
        <button class="actionbutton" @click="reset" title="Reset puzzle" aria-label="Reset puzzle">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24">
            <path fill="currentColor" d="M17.65 6.35C16.2 4.9 14.21 4 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08c-.82 2.33-3.04 4-5.65 4-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/>
          </svg>
          <span class="button-label">Reset</span>
        </button>
      </div>
    </div>

    <div v-if="hasWon" class="modal-overlay" @click="closeModal">
      <div class="modal" @click.stop>
        <button class="close-button" @click="closeModal">×</button>
        <div class="message success">
          <strong>YOU WON!</strong>
          <span v-if="counter > optimal" class="hint">There is a solution that uses fewer moves.<br />Can you find it?</span>
          <span v-else-if="counter == optimal" class="perfect">Perfect score!</span>
          <button class="copy-button" @click="copyResults">Share results</button>
        </div>
      </div>
    </div>

    <div class="fieldcontainer" :style="{'grid-template-columns': 'repeat(' + (state.length + 1) + ', 1fr)'}">
      <template v-for="(row, rowIndex) in state" :key="rowIndex">
        <div :style="{'grid-column': 1, 'grid-row': rowIndex + 1}">
          <button @click="rowClick(rowIndex)" :disabled="activeColumn !== null">→</button>
        </div>
        <template v-for="(col, colIndex) in row" :key="colIndex">
          <div :class="`cell-${rowIndex}-${colIndex}`" :style="{'grid-column': colIndex + 2, 'grid-row': rowIndex + 1}">
            <div :style="[colIndex == activeColumn || rowIndex == activeRow ? {backgroundColor: 'hsl(120, 100%, 90%)'} : {backgroundColor: '#eeeeee'}]">
              <div style="padding: 20%" :style="[col ? {backgroundColor: 'hsl(0, 100%, 90%)'} : {backgroundColor: 'white'}]"></div>
            </div>
          </div>
        </template>
      </template>
      <template v-for="(col2, index) in state[0]" :key="index">
        <div :style="{'grid-column': index + 2, 'grid-row': state.length + 1 }">
          <button @click="columnClick(index)" :disabled="activeRow !== null">↑</button>
        </div>
      </template>
    </div>

    <div>
      <h3>Target</h3>
      <div class="fieldcontainer" :style="{'grid-template-columns': 'repeat(' + (state.length + 1) + ', 1fr)'}">
        <template v-for="(row, rowIndex) in target" :key="rowIndex">
          <template v-for="(col, colIndex) in row" :key="colIndex">
            <div style="background-color: #eeeeee;" :style="{'grid-column': colIndex + 2, 'grid-row': rowIndex + 1}">
              <div style="padding: 20%" :style="[col ? {backgroundColor: 'hsl(0, 100%, 90%)'} : {backgroundColor: 'white'}]"></div>
            </div>
          </template>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import JSConfetti from 'js-confetti'

export default {
  name: 'SwappleMain',
  props: {
    msg: String
  },
  methods: {
    copyResults() {
      navigator.clipboard.writeText(`Swapple ${this.date}\n\nSolved in ${this.counter} moves!\n\nhttps://fuglede.github.io/swapple/`);
      const button = event.target;
      button.textContent = "Result copied to clipboard!";
      setTimeout(() => {
        button.textContent = "Copy Results";
      }, 5000);
    },
    closeModal() {
      this.hasWon = false;
    },
    reset() {
      this.state = [[true, false, false, false], [false, true, false, false], [false, false, true, false], [false, false, false, true]];
      this.moves = [];
      this.activeColumn = null;
      this.activeRow = null;
      this.counter = 0;
    },
    undo() {
      if (this.moves.length === 0) {
        return
      }
      const move = this.moves.pop();
      console.log(move);
      if (move.type === "row") {
        const activeRow = move.activeRow;
        const index = move.index
        for (let i = 0; i < this.state.length; i++) {
          this.state[index][i] ^= this.state[activeRow][i];
        }
      } else {
        const activeColumn = move.activeColumn;
        const index = move.index;
        for (let i = 0; i < this.state.length; i++) {
          this.state[i][index] ^= this.state[i][activeColumn];
        }
      }
      this.counter -= 1;
      this.activeColumn = null;
      this.activeRow = null;
    },
    rowClick(index) {
      console.log(index)
      if (this.activeColumn !== null) {
        this.activeColumn = null;
      }
      if (this.activeRow === index) {
        this.activeRow = null;
        return
      }
      if (this.activeRow === null) {
        this.activeRow = index;
      } else {
        const sourceRow = this.state[this.activeRow];
        sourceRow.forEach((value, i) => {
          if (value) {
            const cell = document.querySelector(`.cell-${index}-${i} > div`);
            cell.classList.add('cell-updating');
            setTimeout(() => cell.classList.remove('cell-updating'), 300);
          }
        });
        
        for (let i = 0; i < this.state.length; i++) {
          this.state[index][i] ^= this.state[this.activeRow][i];
        }
        this.moves.push({type: "row", activeRow: this.activeRow, index: index})
        this.activeRow = null;
        this.counter += 1;
      }
    },
    columnClick(index) {
      if (this.activeRow !== null) {
        this.activeRow = null;
      }
      if (this.activeColumn === index) {
        this.activeColumn = null;
        return
      }
      if (this.activeColumn === null) {
        this.activeColumn = index;
      } else {
        for (let i = 0; i < this.state.length; i++) {
          if (this.state[i][this.activeColumn]) {
            const cell = document.querySelector(`.cell-${i}-${index} > div`);
            cell.classList.add('cell-updating');
            setTimeout(() => cell.classList.remove('cell-updating'), 300);
          }
        }
        
        for (let i = 0; i < this.state.length; i++) {
          this.state[i][index] ^= this.state[i][this.activeColumn];
        }
        this.moves.push({type: "column", activeColumn: this.activeColumn, index: index})
        this.activeColumn = null;
        this.counter += 1;
      }
    },
  },
  data() {
    // Convert current date to days since epoch
    const thisDay = new Date();
    const todayString = thisDay.toISOString().slice(0, 10);

    const epoch = new Date(2024, 0, 0);
    const daysSinceEpoch = Math.floor((thisDay - epoch) / (1000 * 60 * 60 * 24));
    const puzzleIndex = daysSinceEpoch % 200;
    const dists = [5, 5, 6, 6, 8, 7, 5, 6, 7, 7, 7, 8, 8, 6, 5, 8, 7, 5, 7, 7, 5, 8, 7, 6, 8, 6, 7, 6, 8, 6, 5, 8, 6, 5, 8, 5, 8, 6, 7, 8, 8, 8, 8, 8, 5, 5, 6, 7, 5, 7, 7, 5, 7, 5, 8, 8, 7, 8, 5, 7, 6, 8, 7, 8, 5, 6, 8, 7, 6, 7, 6, 8, 6, 7, 6, 8, 6, 5, 7, 8, 8, 6, 8, 7, 5, 6, 8, 8, 5, 8, 8, 5, 7, 5, 8, 8, 5, 6];
    const puzzles = [[[true, true, true, true], [true, true, false, true], [true, false, true, false], [false, false, false, true]],[[true, false, false, true], [true, true, true, false], [true, false, false, false], [true, false, true, false]],[[false, false, true, false], [true, true, true, true], [true, false, false, true], [false, true, false, true]],[[false, true, true, true], [false, false, true, true], [true, true, true, false], [true, false, false, false]],[[false, false, false, true], [true, false, true, false], [false, true, false, false], [false, false, true, false]],[[false, true, false, false], [false, true, true, false], [true, true, true, true], [true, true, true, false]],[[true, true, false, true], [true, true, true, false], [true, true, false, false], [false, true, false, true]],[[false, true, false, false], [true, true, true, true], [true, false, true, false], [true, false, false, false]],[[false, false, false, true], [false, false, true, false], [false, true, false, false], [true, true, true, false]],[[false, true, true, false], [false, false, true, false], [false, false, true, true], [true, true, true, true]],[[true, true, false, true], [true, true, false, false], [true, false, false, false], [true, true, true, true]],[[false, true, false, true], [false, false, true, true], [true, false, false, true], [false, true, false, false]],[[false, false, false, true], [false, false, true, false], [true, true, false, false], [false, true, true, false]],[[true, true, true, false], [false, false, true, false], [true, true, false, true], [false, true, false, true]],[[false, true, false, true], [true, true, true, true], [true, true, false, true], [true, false, false, true]],[[false, true, true, true], [false, false, false, true], [true, false, false, false], [false, false, true, true]],[[false, false, false, true], [true, false, true, true], [false, true, false, false], [true, true, false, false]],[[true, false, true, true], [true, false, false, true], [false, true, true, false], [false, false, false, true]],[[false, true, true, false], [true, false, true, false], [false, true, true, true], [true, false, false, false]],[[false, true, false, false], [false, false, true, true], [true, false, true, true], [false, true, false, true]],[[true, true, false, true], [false, true, true, false], [true, true, true, false], [false, true, true, true]],[[true, false, true, true], [false, false, true, false], [true, false, true, false], [false, true, false, false]],[[false, true, false, false], [true, true, true, true], [false, true, false, true], [true, false, false, true]],[[false, true, true, false], [true, false, true, false], [false, false, true, false], [true, true, true, true]],[[false, false, false, true], [true, true, true, true], [false, true, true, true], [false, false, true, false]],[[false, true, true, false], [true, true, true, false], [false, false, true, true], [true, true, false, false]],[[false, true, false, false], [true, false, true, false], [false, true, true, true], [false, false, true, false]],[[false, false, true, false], [false, true, false, true], [true, true, false, true], [true, false, false, true]],[[false, false, true, false], [true, false, false, false], [false, true, false, true], [false, true, false, false]],[[false, true, true, true], [true, true, false, true], [true, false, true, true], [false, false, true, false]],[[false, true, false, false], [true, true, true, true], [false, true, true, true], [false, false, false, true]],[[false, true, true, false], [false, true, true, true], [true, true, true, true], [false, false, true, false]],[[false, false, true, true], [false, false, true, false], [false, true, false, true], [true, false, true, true]],[[true, false, true, false], [true, true, true, true], [false, false, true, true], [false, true, false, false]],[[false, true, true, true], [false, false, true, false], [false, true, false, false], [true, true, false, false]],[[true, true, true, true], [false, true, false, true], [false, false, true, true], [true, false, true, true]],[[false, true, false, true], [true, false, false, true], [false, false, false, true], [true, true, true, false]],[[true, true, true, false], [true, false, false, false], [true, false, false, true], [false, false, true, true]],[[false, true, false, true], [true, false, true, false], [true, false, false, true], [false, true, false, false]],[[false, false, true, false], [false, false, true, true], [true, true, true, false], [true, false, false, false]],[[false, false, true, false], [true, false, false, false], [false, true, false, true], [true, true, false, false]],[[false, false, true, true], [false, false, true, false], [true, false, false, false], [true, true, true, false]],[[false, false, false, true], [false, false, true, false], [true, false, false, false], [true, true, false, false]],[[false, true, false, true], [false, false, true, true], [true, false, false, true], [false, false, true, false]],[[true, false, true, false], [true, true, false, true], [true, true, true, true], [false, true, false, false]],[[true, true, false, false], [false, true, true, false], [false, true, false, false], [true, false, true, true]],[[true, false, true, false], [true, true, true, false], [false, false, false, true], [false, true, true, false]],[[false, true, false, true], [true, true, true, false], [true, false, false, false], [false, false, true, false]],[[true, true, true, false], [true, false, true, false], [true, true, false, false], [false, true, false, true]],[[true, true, true, true], [true, false, false, true], [true, true, true, false], [true, false, true, true]],[[false, true, true, true], [true, true, true, false], [true, false, false, false], [false, false, true, false]],[[true, true, true, false], [false, true, false, false], [true, true, false, true], [true, true, false, false]],[[false, true, false, true], [true, false, false, true], [false, false, false, true], [true, true, true, true]],[[false, false, false, true], [false, true, true, false], [true, true, false, false], [true, true, true, true]],[[false, false, false, true], [false, false, true, true], [true, false, false, false], [true, true, false, false]],[[false, true, true, false], [true, false, true, true], [false, true, true, true], [false, true, false, false]],[[true, false, true, true], [true, false, true, false], [true, false, false, false], [true, true, false, false]],[[false, true, false, true], [true, false, true, false], [false, false, false, true], [true, true, false, false]],[[true, true, true, true], [false, true, true, true], [false, false, false, true], [true, false, true, true]],[[false, false, true, false], [false, true, true, true], [false, true, true, false], [true, false, false, true]],[[false, false, false, true], [false, true, true, false], [false, false, true, true], [true, true, true, false]],[[false, false, false, true], [false, false, true, false], [true, false, false, false], [true, true, false, false]],[[false, true, true, false], [true, false, false, true], [false, true, false, true], [false, true, false, false]],[[false, false, false, true], [false, false, true, true], [true, true, false, true], [false, true, false, false]],[[true, false, false, true], [false, true, false, false], [false, true, false, true], [false, false, true, false]],[[false, false, false, true], [false, true, true, true], [false, true, false, false], [true, false, false, false]],[[false, true, true, false], [true, false, true, true], [false, false, false, true], [true, false, false, false]],[[false, false, true, false], [true, false, true, true], [true, false, false, false], [false, true, true, true]],[[false, false, true, false], [true, false, true, true], [true, true, false, true], [false, false, true, true]],[[false, false, true, false], [true, false, false, false], [false, true, true, true], [false, true, false, false]],[[true, false, true, true], [false, false, false, true], [false, true, true, false], [true, true, true, false]],[[true, false, true, false], [false, false, false, true], [false, true, false, false], [true, false, false, false]],[[true, false, true, false], [false, true, true, true], [true, true, true, true], [true, true, false, false]],[[false, false, false, true], [false, false, true, true], [true, false, false, false], [true, true, true, true]],[[false, true, false, true], [true, true, true, true], [false, true, true, true], [false, true, false, false]],[[false, false, false, true], [true, false, false, false], [true, true, false, true], [true, false, true, false]],[[true, false, true, false], [false, false, true, false], [false, true, false, true], [true, false, false, true]],[[false, true, true, true], [true, false, false, true], [true, false, true, true], [true, true, false, true]],[[false, false, true, false], [true, false, true, true], [false, false, true, true], [true, true, true, true]],[[false, false, false, true], [true, false, true, true], [false, true, false, true], [false, false, true, false]],[[false, true, false, false], [false, true, false, true], [true, true, false, true], [false, false, true, false]],[[true, true, true, false], [false, false, false, true], [true, false, false, true], [true, false, true, true]],[[false, true, true, false], [true, false, true, true], [true, true, false, false], [true, false, false, false]],[[false, false, true, true], [true, false, false, false], [false, true, true, false], [false, true, false, false]],[[true, true, false, true], [true, true, false, false], [false, true, true, false], [true, false, false, false]],[[true, true, true, true], [true, false, true, false], [true, false, false, false], [false, false, true, true]],[[false, true, false, false], [true, false, true, true], [true, true, true, false], [false, true, true, false]],[[false, true, false, true], [false, false, true, false], [true, false, false, false], [true, true, false, false]],[[false, false, true, true], [true, true, true, false], [false, false, true, false], [false, true, false, true]],[[false, false, true, true], [true, false, false, false], [false, false, false, true], [false, true, false, false]],[[false, true, false, false], [false, false, false, true], [true, false, false, true], [false, true, true, true]],[[false, false, true, false], [false, true, true, false], [true, false, true, true], [true, false, true, false]],[[false, true, true, true], [true, false, false, false], [true, true, false, true], [true, true, false, false]],[[true, false, true, true], [true, true, false, true], [false, false, true, true], [true, true, false, false]],[[false, true, true, false], [true, false, false, true], [false, false, false, true], [false, false, true, false]],[[false, true, true, true], [false, true, false, true], [false, true, false, false], [true, true, false, true]],[[true, false, false, false], [true, false, true, true], [false, true, false, false], [true, false, false, true]],[[true, false, true, true], [true, false, false, false], [true, true, false, false], [false, true, false, true]]];
    

    return {
      moves: [],
      counter: 0,
      activeRow: null,
      activeColumn: null,
      hasWon: false,
      optimal: dists[puzzleIndex],
      date: todayString,

      state: [[true, false, false, false], [false, true, false, false], [false, false, true, false], [false, false, false, true]],
      target: puzzles[puzzleIndex],
    }
  },
  watch: {
    counter: function() {
      for (let i = 0; i < this.state.length; i++) {
        for (let j = 0; j < this.state[i].length; j++) {
          if (this.state[i][j] != this.target[i][j]) {
            this.hasWon = false;
            return;
          }
        }
      }
      this.hasWon = true;
      if (this.counter == this.optimal) {
        const jsConfetti = new JSConfetti()
        jsConfetti.addConfetti({
          emojis: ['🌈', '⚡️', '💥', '✨', '💫', '🌸'],
        })
      }
    }
  }
}
</script>

<style scoped>
.game-container {
  max-width: 500px;
  margin: 0 auto;
  padding: 16px;
}

header {
  border-bottom: 1px solid #d3d6da;
  padding-bottom: 16px;
  margin-bottom: 24px;
  text-align: center;
}

h1 {
  font-size: 36px;
  font-weight: 700;
  margin: 0;
  letter-spacing: 0.2rem;
}

.date-label {
  color: #787c7e;
  font-size: 14px;
  margin-top: 4px;
}

.stats-container {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 24px;
  gap: 24px;
  flex-wrap: wrap;
}

.stat-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #f3f3f3;
  padding: 8px 16px;
  border-radius: 8px;
}

.stat-number {
  font-size: 24px;
  font-weight: bold;
  color: #202020;
}

.stat-label {
  font-size: 14px;
  color: #787c7e;
  text-transform: uppercase;
}

.message {
  text-align: center;
  padding: 16px;
  font-size: 14px;
  border-radius: 8px;
  min-width: 200px;
}

.success {
  background: #c9f7c9;
  font-size: 16px;
  color: #1a651a;
}

.hint {
  display: block;
  font-size: 16px;
  margin-top: 4px;
}

.perfect {
  display: block;
  font-size: 16px;
  margin-top: 4px;
  color: #1a651a;
  font-weight: bold;
}

.button-container {
  display: flex;
  gap: 8px;
}

.fieldcontainer {
  display: grid;
  row-gap: 0px;
  width: 300px;
  margin: 16px auto;
  text-align: center;
}

.fieldcontainer div {
  aspect-ratio: 1;
  transition: all 0.3s ease;
  background-clip: content-box;
}

.fieldcontainer div > div {
  transition: background-color 0.3s ease, transform 0.2s ease;
}

.cell-updating {
  animation: update-flash 0.3s ease;
}

@keyframes update-flash {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}

.fieldcontainer button {
  margin: 20%;
  height: 60%;
  width: 60%;
  background: hsl(240, 100%, 90%);
  color: #555555;
  border: none;
  font-weight: bold;
}

.actionbutton {
  height: 60px;
  width: 60px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: hsl(240, 100%, 90%);
  color: #303030;
  border: none;
  gap: 2px;
}

.button-label {
  font-size: 12px;
}

button {
  background-color: hsl(240, 100%, 90%);
  border: 0px;
  margin: 0px;
  font-size: 16px;
  border-radius: 2px;
  align-items: center;
  padding: 0px;
}

button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

button:disabled:hover {
  background-color: hsl(240, 100%, 90%);
  opacity: 0.7;
}

button:disabled:active {
  transform: none;
}

h3 {
  margin: 24px 0 8px;
  color: #202020;
}

@media(hover:hover) {
  button:hover {
    background-color: hsl(120, 100%, 90%);
    transition-duration: 200ms;
    opacity: 0.9;
  }
}

button:active {
  transform: scale(0.9);
  transition-duration: 50ms;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal {
  background: white;
  padding: 24px;
  border-radius: 24px;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.2);
  font-size: 16px;
  max-width: 90%;
  animation: modal-appear 0.3s ease;
  position: relative;
}

@keyframes modal-appear {
  from { 
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.close-button {
  position: absolute;
  right: 20px;
  top: 20px;
  background: none;
  border: none;
  font-size: 30px;
  cursor: pointer;
  color: #666;
  padding: 5px 10px;
}

.close-button:hover {
  color: #000;
  background: none;
}

.copy-button {
  margin-top: 16px;
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
}

.copy-button:hover {
  background: #45a049;
}
</style>
