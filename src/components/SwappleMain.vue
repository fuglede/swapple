<template>
  <h1>SWAPPLE</h1>
  <div>
    <h3>Moves: {{ counter }}</h3>
    <h3 v-if="hasWon" style="margin-top: 5px;">YOU WON!</h3>
    <h4 v-if="hasWon && counter != 5">But you can actually do it in fewer moves. Can you find a solution with fewer moves?</h4>
    <h4 v-if="hasWon && counter == 5">And that's even the shortest number of possible moves!</h4>
    <div class="fieldcontainer" :style="{'grid-template-columns': 'repeat(' + (state.length + 1) + ', 1fr)'}">
      <template v-for="(row, rowIndex) in state" :key="rowIndex">
        <div :style="{'grid-column': 1, 'grid-row': rowIndex + 1}">
          <button @click="rowClick(rowIndex)">→</button>
        </div>
        <template v-for="(col, colIndex) in row" :key="colIndex">
          <div :style="{'grid-column': colIndex + 2, 'grid-row': rowIndex + 1}">
            <div :style="[colIndex == activeColumn || rowIndex == activeRow ? {backgroundColor: 'hsl(120, 100%, 90%)'} : {backgroundColor: '#eeeeee'}]">
              <div style="padding: 20%" :style="[col ? {backgroundColor: 'hsl(0, 100%, 90%)'} : {backgroundColor: 'white'}]"></div>
            </div>
          </div>
        </template>
      </template>
      <template v-for="(col2, index) in state[0]" :key="index">
        <div :style="{'grid-column': index + 2, 'grid-row': state.length + 1 }">
          <button @click="columnClick(index)">↑</button>
        </div>
      </template>
    </div>
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

  <div style="display: float">
    <button class="actionbutton" @click="undo">Undo</button>
    <button class="actionbutton" @click="reset">Reset</button>
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
        console.log("boo")
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
    return {
      moves: [],
      counter: 0,
      activeRow: null,
      activeColumn: null,
      hasWon: false,

      state: [[true, false, false, false], [false, true, false, false], [false, false, true, false], [false, false, false, true]],
      target: [[true, false, true, false], [false, false, true, false], [true, true, true, false], [true, true, false, true]]
    }
  },
  watch: {
    counter: function() {
      for (let i = 0; i < this.state.length; i++) {
        for (let j = 0; j < this.state[i].length; j++) {
          if (this.state[i][j] != this.target[i][j]) return;
        }
      }
      this.hasWon = true;
      if (this.counter == 5) {
        const jsConfetti = new JSConfetti()
        jsConfetti.addConfetti({
          emojis: ['🌈', '⚡️', '💥', '✨', '💫', '🌸'],
        })
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1 {
  font-weight: 700;
  margin-bottom: 3%;
}
h2 { 
  margin: 0;
}
h3 {
  margin: 3% 0 0;
}
.fieldcontainer {
  display: grid;
  row-gap: 0px;
  width: 300px;
  margin-top: 1%;
  margin-bottom: 20px;
  margin-left: auto;
  margin-right: auto;
  text-align: center;
}
.fieldcontainer div {
  aspect-ratio: 1;
  transition-duration: 400ms;
  background-clip: content-box;
}
.fieldcontainer button {
  margin-top: 20%;
  margin-bottom: 20%;
  margin-left: 20%;
  margin-right: 20%;
  height: 60%;
  width: 60%;
}
.actionbutton {
  height: 30px;
  width: 60px;
  margin-left: 20px;
  margin-right: 20px;
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

@media(hover:hover) {
  button:hover {
    background-color: hsl(120, 100%, 90%);
    transition-duration: 200ms;
  }
}
button:active {
  background-color: hsl(120, 100%, 90%);
  transition-duration: 200ms;
}
</style>
