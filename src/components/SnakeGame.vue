<template>
  <div class="row mt-5 p-5">
    <div class="col mx-auto">
      <div class="card">
        <div class="card-header">
          <h2 class="card-title">Snake Game 🐍</h2>
        </div>
        <div v-if="!loggedIn" class="p-5">
          <p>please enter your name</p>
          <b-input type="text" class="w-50" v-model="userName"></b-input>
          <div>
            <b-button
              :disabled="!userName"
              varinat="primary"
              class="mt-3"
              @click="login"
              >Login</b-button
            >
          </div>
        </div>
        <div v-else>
          <div class="px-0 d-flex justify-content-center my-3">
            <div class="game-board px-0" :style="boardStyles">
              <svg :style="boardStyles">
                <g v-for="segment in snake" :key="segment.id">
                  <rect
                    :x="segment.x * squareSize"
                    :y="segment.y * squareSize"
                    :width="squareSize"
                    :height="squareSize"
                    :fill="segment.color"
                  />
                </g>
              </svg>
              <div
                class="snake"
                v-for="(segment, i) in snake"
                :style="{
                  width: `${squareSize}px`,
                  height: `${squareSize}px`,
                  left: `${segment.x}px`,
                  top: `${segment.y}px`
                }"
                :key="i"
              ></div>
              <div
                class="food"
                :style="{
                  width: `${squareSize}px`,
                  height: `${squareSize}px`,
                  left: `${food.x}px`,
                  top: `${food.y}px`
                }"
              ></div>
            </div>
          </div>
          <div class="ml-3 mt-3">
            <div class="player mb-3">
              <label>player:</label>

              <span class="">{{ userName }}</span>
            </div>
            <div class="score mb-3">
              <label>Score:</label>

              <span class="">{{ score }}</span>
            </div>
            <div class="time-elapsed mb-3">
              <label>time:</label>

              <span class="">{{ timeElapsed }}</span>
            </div>
            <div v-if="gameOver">
              <p>🐍 GAME OVER 💀</p>
            </div>
          </div>
          <div class="card-footer d-flex justify-content-center">
            <b-button
              v-if="!gameOver"
              @click="startGame"
              variant="primary"
              class="mr-3"
              >Play</b-button
            >
            <b-button @click="restart" class="mr-3">Restart</b-button>

            <b-button v-b-modal.top-scores>Top scores</b-button>
          </div>
        </div>
      </div>
    </div>

    <b-modal id="top-scores" title="Top scores 🏆">
      <b-list-group class="w-50">
        <b-list-group-item v-for="score in topScores" :key="score.name">
          {{ score.s }} - {{ score.name }}
        </b-list-group-item>
      </b-list-group>
    </b-modal>
  </div>
</template>

<script>
import { DateTime } from 'luxon'
export default {
  name: 'SnakeGame',
  data() {
    return {
      userName: null,
      loggedIn: false,
      snake: [],
      food: {},
      head: {},
      squareSize: 20,
      score: 0,
      direction: 'right',
      gameOver: false,
      boardStyles: {
        width: '500px',
        height: '500px',
        backgroundColor: 'lightgray'
      },
      windowWidth: 0,
      cardWidth: 0,
      startTime: null,
      finishTime: null,
      timeElapsed: 0,
      gameStarted: false,
      topScores: [
        { name: 'John', s: 20 },
        { name: 'Kim', s: 32 },
        { name: 'Julian', s: 54 },
        { name: 'Evie', s: 11 },
        { name: 'Bob', s: 2 },
        { name: 'Moe', s: 5 },
        { name: 'Kate', s: 3 },
        { name: 'Simon', s: 4 },
        { name: 'Sophie', s: 10 },
        { name: 'Smith', s: 2 }
      ]
    }
  },

  mounted() {
    this.cardWidth = document.querySelector('.card').offsetWidth

    window.addEventListener('resize', this.handleResize)
    this.handleCardSize()
    this.generateFood()
    this.createNewSnake()
    this.sortScores()
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.handleResize)
  },
  watch: {
    cardWidth() {
      this.handleCardSize()
    },
    gameStarted(newValue) {
      if (newValue) {
        this.stopTime()
      }
    },
    gameOver(newValue) {
      if (newValue) {
        this.stopTimer()
      }
    }
  },
  computed: {
    didCollideWithWalls() {
      return (
        this.head.x >= parseInt(this.boardStyles.width) ||
        this.head.x < 0 ||
        this.head.y >= parseInt(this.boardStyles.height) ||
        this.head.y < 0
      )
    },
    didCollideWithOwnBody() {
      return this.snake.map((sq) => {
        return this.head.x === sq.x && this.head.y === sq.y
      })
    }
  },
  methods: {
    startGame() {
      this.startTimer()
      document.addEventListener('keydown', this.changeDirection)
      setInterval(this.move, 200)
    },
    startTimer() {
      this.startTime = DateTime.now()
      this.timerInterval = setInterval(() => {
        this.timeElapsed = DateTime.now()
          .diff(this.startTime)
          .toFormat('hh:mm:ss:S')
      }, 100)
    },
    stopTimer() {
      clearInterval(this.timerInterval)
      this.startTime = null
    },
    move() {
      this.head = { ...this.snake[0] }
      this.moveInDirection()

      if (this.didCollideWithWalls) {
        this.gameOver = true
        return
      }
      if (this.didCollideWithOwnBody.includes(true)) {
        this.gameOver = true
        return
      }

      if (this.head.x === this.food.x && this.head.y === this.food.y) {
        this.updateScore()
        this.generateFood()
      } else {
        this.snake.pop()
      }

      this.snake.unshift(this.head)
    },
    changeDirection(event) {
      const c = event.keyCode
      if (c === 65) {
        this.direction = 'left'
      }
      if (c === 87) {
        this.direction = 'up'
      }
      if (c === 68) {
        this.direction = 'right'
      }
      if (c === 83) {
        this.direction = 'down'
      }
    },
    moveInDirection() {
      if (this.direction === 'right') {
        this.head.x += this.squareSize
      }
      if (this.direction === 'left') {
        this.head.x -= this.squareSize
      }
      if (this.direction === 'up') {
        this.head.y -= this.squareSize
      }
      if (this.direction === 'down') {
        this.head.y += this.squareSize
      }
    },
    createNewSnake() {
      this.snake = []
      const newSnake = {
        x:
          Math.floor(
            (Math.random() *
              (parseInt(this.boardStyles.width) - this.squareSize)) /
              this.squareSize
          ) * this.squareSize,
        y:
          Math.floor(
            (Math.random() *
              (parseInt(this.boardStyles.height) - this.squareSize)) /
              this.squareSize
          ) * this.squareSize,
        color: 'red'
      }
      this.snake.push(newSnake)
    },
    generateFood() {
      this.food = {
        x:
          Math.floor(
            (Math.random() *
              (parseInt(this.boardStyles.width) - this.squareSize)) /
              this.squareSize
          ) * this.squareSize,
        y:
          Math.floor(
            (Math.random() *
              (parseInt(this.boardStyles.height) - this.squareSize)) /
              this.squareSize
          ) * this.squareSize
      }
    },
    updateScore() {
      this.score++
      this.topScores.push({ name: this.userName, s: this.score })
      this.sortScores()
      this.topScores.pop()
    },
    sortScores() {
      this.topScores.sort((a, b) => {
        if (a.s < b.s) return 1
        if (a.s > b.s) return -1
        return 0
      })
    },
    restart() {
      this.createNewSnake()
      this.generateFood()
      this.direction = 'right'
      this.gameOver = false
      this.score = 0
      this.timeElapsed = 0
    },
    handleResize() {
      this.windowWidth = window.innerWidth
      this.cardWidth = document.querySelector('.card').offsetWidth
    },
    handleCardSize() {
      this.boardStyles.width =
        this.cardWidth < 500 ? `${this.cardWidth - 20}px` : '500px'
      this.boardStyles.height =
        this.cardWidth < 500 ? `${this.cardWidth - 20}px` : '500px'
    },
    login() {
      if (this.userName) {
        this.loggedIn = true
      }
    }
  }
}
</script>

<style>
.game-board {
  border: 1px solid black;

  position: relative;
}

.snake {
  background-color: green;
  position: absolute;
}

.food {
  background-color: red;
  position: absolute;
}
</style>
