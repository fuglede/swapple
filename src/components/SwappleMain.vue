<template>
  <div class="game-container">
    <header>
      <h1>SWAPPLE</h1>
      <div class="date-label">{{ date }}</div>
      <button class="info-button" @click="showInfo = true" title="About the game" aria-label="About the game">i</button>
      <button class="help-button" @click="showRules = true" title="Show rules" aria-label="Show rules">?</button>
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
          <div class="stars">
            <span class="star" :class="{ earned: true }">
              <svg viewBox="0 0 576 512"><path d="M316.9 18C311.6 7 300.4 0 288.1 0s-23.4 7-28.8 18L195 150.3 51.4 171.5c-12 1.8-22 10.2-25.7 21.7s-.7 24.2 7.9 32.7L137.8 329 113.2 474.7c-2 12 3 24.2 12.9 31.3s23 8 33.8 2.3l128.3-68.5 128.3 68.5c10.8 5.7 23.9 4.9 33.8-2.3s14.9-19.3 12.9-31.3L438.5 329 542.7 225.9c8.6-8.5 11.7-21.2 7.9-32.7s-13.7-19.9-25.7-21.7L381.2 150.3 316.9 18z"/></svg>
            </span>
            <span class="star" :class="{ earned: counter <= optimal + 2 }">
              <svg viewBox="0 0 576 512"><path d="M316.9 18C311.6 7 300.4 0 288.1 0s-23.4 7-28.8 18L195 150.3 51.4 171.5c-12 1.8-22 10.2-25.7 21.7s-.7 24.2 7.9 32.7L137.8 329 113.2 474.7c-2 12 3 24.2 12.9 31.3s23 8 33.8 2.3l128.3-68.5 128.3 68.5c10.8 5.7 23.9 4.9 33.8-2.3s14.9-19.3 12.9-31.3L438.5 329 542.7 225.9c8.6-8.5 11.7-21.2 7.9-32.7s-13.7-19.9-25.7-21.7L381.2 150.3 316.9 18z"/></svg>
            </span>
            <span class="star" :class="{ earned: counter <= optimal }">
              <svg viewBox="0 0 576 512"><path d="M316.9 18C311.6 7 300.4 0 288.1 0s-23.4 7-28.8 18L195 150.3 51.4 171.5c-12 1.8-22 10.2-25.7 21.7s-.7 24.2 7.9 32.7L137.8 329 113.2 474.7c-2 12 3 24.2 12.9 31.3s23 8 33.8 2.3l128.3-68.5 128.3 68.5c10.8 5.7 23.9 4.9 33.8-2.3s14.9-19.3 12.9-31.3L438.5 329 542.7 225.9c8.6-8.5 11.7-21.2 7.9-32.7s-13.7-19.9-25.7-21.7L381.2 150.3 316.9 18z"/></svg>
            </span>
          </div>
          <span v-if="counter > optimal" class="hint">There is a solution that uses fewer moves.<br />Can you find it?</span>
          <span v-else-if="counter == optimal" class="perfect">And that's a perfect score!<br />No solution with fewer moves exist!</span>
          <button class="copy-button" @click="copyResults">Share results</button>
        </div>
      </div>
    </div>

    <div v-if="showRules" class="modal-overlay" @click="showRules = false">
      <div class="modal rules-modal" @click.stop>
        <button class="close-button" @click="showRules = false">×</button>
        <h2>How to play</h2>
        <div class="rules-content">
          <p>Transform the starting pattern into the target pattern using row and column combinations.</p>
          <ol>
            <li>Click on a row arrow to select a row.</li>
            <li>Click on another row arrow to combine the two rows.</li>
          </ol>
          <p>When you combine two rows, the second one you click gets updated by combining its cells with the first one you clicked. The first row is unchanged.</p>
          <p>For each cell, the combination rule is the following:</p>
          <ul>
            <li><span class="white-cell"></span> + <span class="white-cell"></span> = <span class="white-cell"></span></li>
            <li><span class="white-cell"></span> + <span class="red-cell"></span> = <span class="red-cell"></span></li>
            <li><span class="red-cell"></span> + <span class="white-cell"></span> = <span class="red-cell"></span></li>
            <li><span class="red-cell"></span> + <span class="red-cell"></span> = <span class="white-cell"></span></li>
          </ul>
          <p>You can also combine columns with columns, but you can not combine rows with columns.</p>

          <p>Try to reach the target pattern in as few moves as possible!</p>
        </div>
      </div>
    </div>

    <div v-if="showInfo" class="modal-overlay" @click="showInfo = false">
      <div class="modal info-modal" @click.stop>
        <button class="close-button" @click="showInfo = false">×</button>
        <h2>About Swapple</h2>
        <div class="info-content">
          <p>Swapple is a daily puzzle game where you transform patterns by swapping rows and columns. Each day brings a new challenge!</p>
          <p>
            The game is inspired by the synthesis of linear reversible circuits; a problem in reversible and quantum computation. Here, the
            goal is to construct a target operation, the target pattern in Swapple, using a sequence of simpler operations, specifically
            controlled NOT (CNOT) gates, which flip the state of a target bit if and only if a control bit is set. In Swapple, each row and
            column operation corresponds to applying a CNOT gate. Your task is to find a sequence of these gates, i.e. a circuit, that
            transform the initial configuration, corresponding to an empty circuit, into the target configuration. Moreover, finding one of
            the shortest sequences of moves to achieve this goal corresponds to finding one of the most efficient circuits that implements the desired
            operation.
          </p>
          <p>Created by Søren Fuglede Jørgensen. Find the source code or drop requests on <a href="https://github.com/fuglede/swapple">GitHub</a>.</p>
        </div>
      </div>
    </div>

    <div class="fieldcontainer" :style="{'grid-template-columns': 'repeat(' + (state.length + 1) + ', 1fr)'}">
      <template v-for="(row, rowIndex) in state" :key="rowIndex">
        <div :style="{'grid-column': 1, 'grid-row': rowIndex + 1}"
             draggable="true"
             :data-row="rowIndex"
             @dragstart="dragStart($event, 'row', rowIndex)"
             @dragend="dragend"
             @dragenter="dragenter($event, 'row', rowIndex)"
             @dragleave="dragleave"
             @dragover="dragover"
             @drop="drop($event, 'row', rowIndex)"
             @touchstart="touchStart($event, 'row', rowIndex)"
             @touchmove.prevent="touchMove($event)"
             @touchend="touchEnd($event)"
             @touchcancel="touchcancel"
             class="draggable-container">
          <button @click="rowClick(rowIndex)" :disabled="activeColumn !== null">
            <span v-if="activeRow === rowIndex">×</span>
            <span v-else-if="activeRow !== null">⊕</span>
            <span v-else>→</span>
          </button>
        </div>
        <template v-for="(col, colIndex) in row" :key="colIndex">
          <div :class="`cell-${rowIndex}-${colIndex}`" :style="{'grid-column': colIndex + 2, 'grid-row': rowIndex + 1}">
            <div :style="[
              (dragType === 'column' && colIndex == activeColumn) ? {backgroundColor: 'hsl(120, 100%, 90%)'} :
              (dragType === 'row' && rowIndex == activeRow) ? {backgroundColor: 'hsl(120, 100%, 90%)'} :
              (dragType === 'column' && colIndex == dragHoverIndex) ? {backgroundColor: 'hsla(200, 100%, 85%, 60%)'} :
              (dragType === 'row' && rowIndex == dragHoverIndex) ? {backgroundColor: 'hsla(200, 100%, 85%, 60%)'} :
              colIndex == activeColumn ? {backgroundColor: 'hsl(120, 100%, 90%)'} :
              rowIndex == activeRow ? {backgroundColor: 'hsl(120, 100%, 90%)'} :
              {backgroundColor: '#eeeeee'}
            ]">
              <div style="padding: 20%" :style="[col ? {backgroundColor: 'hsl(0, 100%, 90%)'} : {backgroundColor: 'white'}]"></div>
            </div>
          </div>
        </template>
      </template>
      <template v-for="(col2, index) in state[0]" :key="index">
        <div :style="{'grid-column': index + 2, 'grid-row': state.length + 1 }"
             draggable="true"
             :data-column="index"
             @dragstart="dragStart($event, 'column', index)"
             @dragend="dragend"
             @dragenter="dragenter($event, 'column', index)"
             @dragleave="dragleave"
             @dragover="dragover"
             @drop="drop($event, 'column', index)"
             @touchstart="touchStart($event, 'column', index)"
             @touchmove.prevent="touchMove($event)"
             @touchend="touchEnd($event)"
             @touchcancel="touchcancel"
             class="draggable-container">
          <button @click="columnClick(index)" :disabled="activeRow !== null">
            <span v-if="activeColumn === index">×</span>
            <span v-else-if="activeColumn !== null">⊕</span>
            <span v-else>↑</span>
          </button>
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
      // Calculate stars earned
      let stars = '⭐';
      if (this.counter <= this.optimal + 2) {
        stars = '⭐⭐';
      }
      if (this.counter <= this.optimal) {
        stars = '⭐⭐⭐';
      }
      
      navigator.clipboard.writeText(`Swapple ${this.date}\n\nSolved in ${this.counter} moves! ${stars}\n\nhttps://swapple.fuglede.dk/`);
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
        const oldValues = [...this.state[index]];
        for (let i = 0; i < this.state.length; i++) {
          this.state[index][i] ^= this.state[this.activeRow][i];
          // Add animation class to changed cells; check difference as ints
          if (oldValues[i] != this.state[index][i]) {
            const cell = document.querySelector(`.cell-${index}-${i} > div > div`);
            cell.classList.add('cell-changed');
            setTimeout(() => cell.classList.remove('cell-changed'), 500);
          }
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
        const oldValues = this.state.map(row => row[index]);
        for (let i = 0; i < this.state.length; i++) {
          this.state[i][index] ^= this.state[i][this.activeColumn];
          // Add animation class to changed cells
          if (oldValues[i] != this.state[i][index]) {
            const cell = document.querySelector(`.cell-${i}-${index} > div > div`);
            cell.classList.add('cell-changed');
            setTimeout(() => cell.classList.remove('cell-changed'), 500);
          }
        }
        this.moves.push({type: "column", activeColumn: this.activeColumn, index: index})
        this.activeColumn = null;
        this.counter += 1;
      }
    },
    checkFirstVisit() {
      const hasVisited = localStorage.getItem('swappleHasVisited');
      if (!hasVisited) {
        this.showRules = true;
        localStorage.setItem('swappleHasVisited', 'true');
      }
    },
    dragStart(event, type, index) {
      // Disable drag and drop on Firefox
      if (navigator.userAgent.toLowerCase().indexOf('firefox') > -1) {
        event.preventDefault();
        return;
      }
      
      // Firefox requires dataTransfer data and effectAllowed to be set
      event.dataTransfer.effectAllowed = 'move';
      event.dataTransfer.setData('text/plain', index.toString());
      
      // Delay adding classes to next tick for Firefox
      setTimeout(() => {
        this.dragStarted = true;
        this.isDragging = true;
        this.dragType = type;
        this.dragIndex = index;
        
        const draggableElement = event.target;
        draggableElement.classList.add('dragging');
        
        const validTargets = document.querySelectorAll(`[data-${type}]`);
        validTargets.forEach(target => {
          if (target.getAttribute(`data-${type}`) !== index.toString()) {
            target.classList.add('droppable-target');
          }
        });
        if (type === 'row') {
          this.rowClick(this.dragIndex);
        } else if (type === 'column') {
          this.columnClick(this.dragIndex);
        }
      }, 0);
    },
    highlightValidTargets(type, index) {
      const validTargets = document.querySelectorAll(`[data-${type}]`);
      validTargets.forEach(target => {
        if (target.getAttribute(`data-${type}`) !== index.toString()) {
          target.classList.add('droppable-target');
        }
      });
    },
    clearHighlights() {
      const highlights = document.querySelectorAll('.droppable-target, .droppable-target-hover, .dragging');
      highlights.forEach(el => {
        el.classList.remove('droppable-target', 'droppable-target-hover', 'dragging');
      });
    },
    dragenter(event, type, index) {
      // Prevent default to allow drop
      event.preventDefault();
      
      if (!this.isDragging || this.dragType !== type || this.dragIndex === index) return;
      
      // Remove highlight from other elements
      const highlights = document.querySelectorAll('.droppable-target-hover');
      highlights.forEach(el => el.classList.remove('droppable-target-hover'));
      
      event.currentTarget.classList.add('droppable-target-hover');
      
      // Set hover index to highlight the target row/column
      this.dragHoverIndex = index;
    },
    dragover(event) {
      // Required for Firefox
      event.preventDefault();
      event.dataTransfer.dropEffect = 'move';
    },
    dragleave(event) {
      // Only remove if we're not entering a child element
      if (!event.relatedTarget || !event.currentTarget.contains(event.relatedTarget)) {
        event.currentTarget.classList.remove('droppable-target-hover');
        this.dragHoverIndex = null;
      }
    },
    drop(event, type, index) {
      const draggingElements = document.querySelectorAll('.dragging');
      draggingElements.forEach(el => el.classList.remove('dragging'));
      
      const validTargets = document.querySelectorAll('.droppable-target, .droppable-target-hover');
      validTargets.forEach(target => {
        target.classList.remove('droppable-target');
        target.classList.remove('droppable-target-hover');
      });
      
      this.isDragging = false;
      if (this.dragType === type) {
        if (type === 'row') {
          this.rowClick(index);
        } else if (type === 'column') {
          this.columnClick(index);
        }
      }
    },
    dragend() {
      this.dragStarted = false;
      this.isDragging = false;
      this.dragHoverIndex = null;
      this.clearHighlights();
    },
    touchStart(event, type, index) {
      this.touchStartX = event.touches[0].clientX;
      this.touchStartY = event.touches[0].clientY;
      this.touchElement = event.target;
      this.dragType = type;
      this.dragIndex = index;
      this.touchMoveCount = 0;
      this.touchDragStarted = false;
    },
    touchMove(event) {

      if (!this.touchElement) return;
      
      this.touchMoveCount++;
      const touch = event.touches[0];
      const moveX = touch.clientX - this.touchStartX;
      const moveY = touch.clientY - this.touchStartY;
      
      // Only start drag if moved more than 10px and at least 2 move events
      if (!this.touchDragStarted && (Math.abs(moveX) > 1 || Math.abs(moveY) > 1) && this.touchMoveCount > 1) {
        this.touchDragStarted = true;
        this.isDragging = true;
        this.touchElement.classList.add('dragging');
        this.highlightValidTargets(this.dragType, this.dragIndex);
      }
      
      if (this.touchDragStarted) {
        const currentTarget = document.elementFromPoint(touch.clientX, touch.clientY);
        if (currentTarget && currentTarget.classList.contains('droppable-target')) {
          const highlights = document.querySelectorAll('.droppable-target-hover');
          highlights.forEach(el => el.classList.remove('droppable-target-hover'));
          currentTarget.classList.add('droppable-target-hover');
          
          // Set hover index for highlighting
          const targetIndex = parseInt(currentTarget.getAttribute(`data-${this.dragType}`));
          if (!isNaN(targetIndex)) {
            this.dragHoverIndex = targetIndex;
          }
        } else {
          this.dragHoverIndex = null;
        }
      }
      
      this.touchStartX = touch.clientX;
      this.touchStartY = touch.clientY;
    },
    touchEnd(event) {
      if (!this.touchElement || !this.touchDragStarted) {
        this.touchElement = null;
        return;
      }
      
      const touch = event.changedTouches[0];
      const elements = document.elementsFromPoint(touch.clientX, touch.clientY);
      
      // Clean up ALL highlights first
      const highlights = document.querySelectorAll('.droppable-target, .droppable-target-hover');
      highlights.forEach(target => {
        target.classList.remove('droppable-target');
        target.classList.remove('droppable-target-hover');
      });
      
      const dropTarget = elements.find(el => {
        return el.hasAttribute(`data-${this.dragType}`) && 
               el.getAttribute(`data-${this.dragType}`) !== this.dragIndex.toString();
      });

      if (dropTarget) {
        const targetIndex = parseInt(dropTarget.getAttribute(`data-${this.dragType}`));
        if (this.dragType === 'row') {
          this.rowClick(this.dragIndex);
          this.rowClick(targetIndex);
        } else if (this.dragType === 'column') {
          this.columnClick(this.dragIndex);
          this.columnClick(targetIndex);
        }
      }

      this.touchElement.classList.remove('dragging');
      this.touchStartX = null;
      this.touchStartY = null;
      this.touchElement = null;
      this.isDragging = false;
      this.touchDragStarted = false;
      this.dragHoverIndex = null;
    },
    touchcancel() {
      if (!this.touchElement) return;
      
      const highlights = document.querySelectorAll('.droppable-target, .droppable-target-hover');
      highlights.forEach(target => {
        target.classList.remove('droppable-target');
        target.classList.remove('droppable-target-hover');
      });
      
      this.touchElement.classList.remove('dragging');
      this.touchStartX = null;
      this.touchStartY = null;
      this.touchElement = null;
      this.isDragging = false;
      this.touchDragStarted = false;
      this.dragHoverIndex = null;
    }
  },
  mounted() {
    this.checkFirstVisit();
  },
  data() {
    const thisDay = new Date();
    const todayString = thisDay.toLocaleDateString('en-US', { 
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    });

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
      showRules: false,
      showInfo: false,
      optimal: dists[puzzleIndex],
      date: todayString,

      state: [[true, false, false, false], [false, true, false, false], [false, false, true, false], [false, false, false, true]],
      target: puzzles[puzzleIndex],
      touchStartX: null,
      touchStartY: null,
      touchElement: null,
      dragStarted: false,
      touchDragStarted: false,
      touchMoveCount: 0,
      dragHoverIndex: null,
      dragType: null
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
      const jsConfetti = new JSConfetti()
      jsConfetti.addConfetti({
        emojis: ['🌈', '⚡️', '💥', '✨', '💫', '🌸'],
      })
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
  position: relative;
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
  font-size: 16px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.stars {
  display: flex;
  gap: 8px;
  font-size: 32px;
}

.star {
  opacity: 0;
  transform: scale(0);
  animation: star-pop 0.5s ease forwards;
}

.star svg {
  width: 48px;
  height: 48px;
  display: block;
}

.star path {
  fill: #d4d4d4;
}

.star.earned path {
  fill: #ffd700;
}

.star:nth-child(2) {
  animation-delay: 0.2s;
}

.star:nth-child(3) {
  animation-delay: 0.4s;
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
  transition: background-color 0.5s ease, transform 0.3s ease;
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

copy-button:hover {
  background: #45a049;
}

.help-button .info-button {
  position: absolute;
  right: 16px;
  top: 16px;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  font-size: 20px;
  font-weight: bold;
  font-family: monospace;
  display: flex;
  align-items: center;
  justify-content: center;
  background: hsl(240, 100%, 90%);
  color: #303030;
}

.rules-modal {
  max-width: 600px;
  width: 90%;
}

.rules-modal h2 {
  margin-top: 0;
  color: #202020;
}

.rules-content {
  font-size: 16px;
  line-height: 1.5;
  color: #404040;
  text-align: left;
}

.rules-content ol {
  padding-left: 20px;
  margin-top: 12px;
}

.rules-content li {
  margin-bottom: 8px;
}

.white-cell, .red-cell {
  display: inline-block;
  height: 12px;
  width: 12px;
  border-radius: 4px;
}

.white-cell {
  background-color: white;
  border: 1px solid #ccc;
}

.red-cell {
  background-color: hsl(0, 100%, 90%);
  border: 1px solid hsl(0, 100%, 80%);
}

.info-button {
  position: absolute;
  left: 16px;
  top: 16px;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  font-size: 18px;
  font-weight: bold;
  font-family: monospace;
  display: flex;
  align-items: center;
  justify-content: center;
  background: hsl(240, 100%, 90%);
  color: #303030;
}

.help-button {
  position: absolute;
  right: 16px;
  top: 16px;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  font-size: 18px;
  font-weight: bold;
  font-family: monospace;
  display: flex;
  align-items: center;
  justify-content: center;
  background: hsl(240, 100%, 90%);
  color: #303030;
}

.info-modal {
  max-width: 500px;
  width: 90%;
}

.info-content {
  font-size: 16px;
  line-height: 1.5;
  color: #404040;
  text-align: left;
}

.info-content a {
  color: #1a73e8;
  text-decoration: none;
}

.info-content a:hover {
  text-decoration: underline;
}

.droppable-target {
  background-color: rgba(0, 255, 0, 0.1);
  border: 2px dashed #4CAF50;
  transform: scale(1.02);
  transition: all 0.2s ease;
}

.droppable-target-hover {
  background-color: rgba(0, 255, 0, 0.2);
  border: 2px solid #4CAF50;
  transform: scale(1.05);
}

.dragging {
  opacity: 0.6;
  cursor: move;
  transform: scale(0.95);
  transition: all 0.2s ease;
  z-index: 1000;
  background-color: rgba(76, 175, 80, 0.2);
}

.draggable-container {
  touch-action: none;
  user-select: none;
  -webkit-user-select: none;
}

.cell-changed {
  animation: cell-pop 0.5s ease;
}

@keyframes cell-pop {
  0% { transform: scale(1); }
  50% { transform: scale(1.3); }
  100% { transform: scale(1); }
}

@keyframes star-pop {
  0% { 
    opacity: 0;
    transform: scale(0) rotate(-180deg);
  }
  70% {
    transform: scale(1.2) rotate(10deg);
  }
  100% {
    opacity: 1;
    transform: scale(1) rotate(0);
  }
}
</style>
