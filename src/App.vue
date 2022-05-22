<script setup>
  import { $vfm, VueFinalModal, ModalsContainer } from 'vue-final-modal'
  import { ref, onUnmounted, onMounted, computed } from 'vue'
  import Board from './components/Board.vue'
  import Keyboard from './components/Keyboard.vue'
  import { PencilIcon, QuestionMarkCircleIcon, LightBulbIcon, XIcon, ChartBarIcon, RefreshIcon } from '@heroicons/vue/outline'
  import { acceptedWordList } from './assets/js/accepted_word_list.js'
  import { indexes } from './assets/js/indexes.js'
  import { wordList } from './assets/js/wordlist.js'

  let startDay = new Date(2022,4,19)
  let today = new Date()
  let gameNumber = Math.floor((today - startDay) / (1000 * 60 * 60 * 24)) + 1
  let word = wordList[indexes[gameNumber-1]]
  const wordLength = word.length > 0 ? word.length : 0
  let numberOfGuesses = 6
  let msg = computed(() => {
    if ( !finished.value ) {
      return ''
    }
    return correct.value ? 'Aye-Aye, Captain!' : 'Barnacles!'
  })
  let showWinModal = ref(false)
  let showHelpModal = ref(false)
  let showConfirmModal = ref(false)
  let showStatsModal = ref(false)
  let guesses = ref(Array())
  let currentGuess = ref(0)
  let playerGuessCount = ref(1)
  let givenWordCount = 0
  let gameResults = ref('')
  let finished = ref(false)
  let correct = ref(false)
  let notInDictionary = ref(false)
  let gaveUp = ref(false)
  let currentPosition = ref(0)
  let totalWins = ref(0)
  let totalGames = ref(0)
  let currentStreak = ref(0)
  let maxStreak = ref(0)
  let remainingHours = ref(0)
  let remainingMinutes = ref(0)
  let remainingSeconds = ref(0)
  let guessCounts = ref(Array())

  guessCounts.value = JSON.parse(window.localStorage.getItem('guessCounts'))
  if ( guessCounts.value === null ){
    guessCounts.value = [0,0,0,0,0,0,0]
  }

  if ( window.localStorage.getItem('totalGames') != undefined ){
    totalGames.value = window.localStorage.getItem('totalGames')
  }

  const xPercent = computed(() => {
    if ( totalGames.value == 0 ){
      return '10%'
    }
    return guessCounts.value[0] / totalGames.value * 100 + '%'
  })

  const onePercent = computed(() => {
    if ( totalGames.value == 0 ){
      return '10%'
    }
    return guessCounts.value[1] / totalGames.value * 100 + '%'
  })

  const twoPercent = computed(() => {
    if ( totalGames.value == 0 ){
      return '10%'
    }
    return guessCounts.value[2] / totalGames.value * 100 + '%'
  })

  const threePercent = computed(() => {
    if ( totalGames.value == 0 ){
      return '10%'
    }
    return guessCounts.value[3] / totalGames.value * 100 + '%'
  })

  const fourPercent = computed(() => {
    if ( totalGames.value == 0 ){
      return '10%'
    }
    return guessCounts.value[4] / totalGames.value * 100 + '%'
  })

  const fivePercent = computed(() => {
    if ( totalGames.value == 0 ){
      return '10%'
    }
    return guessCounts.value[5] / totalGames.value * 100 + '%'
  })

  const sixPercent = computed(() => {
    if ( totalGames.value == 0 ){
      return '10%'
    }
    return guessCounts.value[6] / totalGames.value * 100 + '%'
  })

  if ( window.localStorage.getItem('totalWins') != undefined ){
    totalWins.value = window.localStorage.getItem('totalWins')
  }


  if ( window.localStorage.getItem('currentStreak') != undefined ){
    currentStreak.value = window.localStorage.getItem('currentStreak')
  }
 if ( window.localStorage.getItem('maxStreak') != undefined ){
    maxStreak.value = window.localStorage.getItem('maxStreak')
  }

  const completeRow = function(skipAnimation) {
    let skip = skipAnimation ? 0 : 1
    let guess = guesses.value[currentGuess.value]
    for ( let i = 0; i < guess.length; i++){
      if ( guess[i]['letter'] === '' ) {
        return
      }
    }

    let answerLetters = word.split('')
    let playerAnswer = guess.map((e) => e['letter']).join('')
    correct.value = ( playerAnswer === word )

    if ( !correct.value && !acceptedWordList.includes(playerAnswer.toLowerCase()) && !skipAnimation ){
      showWordMissingMessage()
      return
    }

    if ( !skipAnimation ){
      window.localStorage.setItem('guesses',JSON.stringify(guesses.value))
    }

    let changedLetters = []
    let keyboardUpdates = []
    for ( let i=0; i < guess.length; i++ ) {
      if ( guess[i]['letter'] === answerLetters[i] ) {
        answerLetters[i] = null
        changedLetters.push(i)
        keyboardUpdates.push([guess[i]['letter'],4])
        setTimeout( () => {
          guess[i]['state'] = 4
        }, 150 * (i) * skip)
        setTimeout( () => {
          guess[i]['colored'] = true
        }, (450 + 150 * (i)) * skip)
      }
    }

    for ( let i=0; i < guess.length; i++ ) {
      if ( !changedLetters.includes(i) && answerLetters.includes(guess[i]['letter']) ){
        answerLetters[answerLetters.indexOf(guess[i]['letter'])] = null
        changedLetters.push(i)
        keyboardUpdates.push([guess[i]['letter'],3])
        setTimeout( () => {
          guess[i]['state'] = 3
        }, 150 * (i) * skip)
        setTimeout( () => {
          guess[i]['colored'] = true
        }, (450 + 150 * (i)) * skip)
      }
    }

    for ( let i=0; i < guess.length; i++ ) {
      if ( !changedLetters.includes(i) ) {
        keyboardUpdates.push([guess[i]['letter'],2])
        setTimeout( () => {
          guess[i]['state'] = 2
        }, 150 * (i) * skip)
        setTimeout( () => {
          guess[i]['colored'] = true
        }, (450 + 150 * (i)) * skip)
      }
    }

    setTimeout( () => {
      for ( let i=0; i < keyboardUpdates.length; i++ ) {
        updateKeyboard(keyboardUpdates[i][0],keyboardUpdates[i][1])
      }
      if ( correct.value || finished.value ) {
        genGameResults()
        if ( !skipAnimation ){
          storeResults()
        }
        showWinModal.value = true
      }
    }, 300 * guess.length * skip)

    if ( correct.value ) {
      finished.value = true
      currentPosition.value = -1
      currentGuess.value = -1
      return
    }

    currentGuess.value++
    currentPosition.value = 0
    if (playerGuessCount.value < numberOfGuesses ){
      playerGuessCount.value++
    } else {
      finished.value = true
    }
  }

  const lastDate = window.localStorage.getItem('lastDate')
  const todayString = "" + today.getFullYear() + today.getMonth() + today.getDate()
  window.localStorage.setItem('lastDate',todayString)
  if ( window.localStorage.getItem('guesses') != undefined && lastDate === todayString ) {
    guesses.value = JSON.parse(window.localStorage.getItem('guesses'))
    for ( let i=0; i < guesses.value.length; i++ ){
      if ( guesses.value[i][0]['letter'] === '' ){
        break;
      }
      completeRow(true)
      if ( i == guesses.value.length - 1){
        finished.value = true
      }
    }
  } else {
    window.localStorage.setItem('guesses',JSON.stringify([]))
  }

  function roundToTwo(value) {
    return(Math.round(value * 100) / 100);
  }
  const winPercentage = computed(() => {
    if ( totalGames.value == 0 ){
      return '0.00'
    }
    return (100 * parseFloat(totalWins.value) / totalGames.value).toFixed(2)
  })

  let tomorrow = new Date();
  tomorrow.setDate(tomorrow.getDate()+1);
  tomorrow.setHours(0)
  tomorrow.setMinutes(0)
  tomorrow.setSeconds(0)
  tomorrow.setMilliseconds(0)
  let milliseconds = tomorrow - today
  let seconds = Math.floor(milliseconds / 1000);
  let minutes = Math.floor(seconds / 60);
  let hours = Math.floor(minutes / 60);

  seconds = seconds % 60;
  minutes = minutes % 60;

  // 👇️ If you don't want to roll hours over, e.g. 24 to 00
  // 👇️ comment (or remove) the line below
  // commenting next line gets you `24:00:00` instead of `00:00:00`
  // or `36:15:31` instead of `12:15:31`, etc.
  hours = hours % 24;

  remainingHours.value = hours
  remainingMinutes.value = minutes
  remainingSeconds.value = seconds

  const timeUntilNew = computed(() => {
    let tomorrow = new Date();
    tomorrow.setDate(tomorrow.getDate()+1);
    tomorrow.setHours(0)
    tomorrow.setMinutes(0)
    tomorrow.setSeconds(0)
    tomorrow.setMilliseconds(0)
    //return tomorrow
    return convertMsToTime(tomorrow - today.value)
  })

  const addEmptyRow = function() {
    let initialGuess = []
    for ( let i=0; i < wordLength; i++ ) {
      initialGuess.push({ 'letter': '', 'state': 0, 'initialized': false, 'colored': false, 'completed': false })
    }
    guesses.value.push(initialGuess)
  }

  for ( let i=guesses.value.length; i < numberOfGuesses; i++ ){
    addEmptyRow()
  }

  let keyboardRows = ref([
    [
      {
        'letter': 'q',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'w',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'e',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'r',
        'state': 1,
        'colored': true
      },
      {
        'letter': 't',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'y',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'u',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'i',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'o',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'p',
        'state': 1,
        'colored': true
      }
    ],
    [
      {
        'letter': 'a',
        'state': 1,
        'colored': true
      },
      {
        'letter': 's',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'd',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'f',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'g',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'h',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'j',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'k',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'l',
        'state': 1,
        'colored': true
      },
      {
        'letter': '_',
        'state': 1,
        'colored': true
      }
    ],
    [
      {
        'letter': 'enter',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'z',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'x',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'c',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'v',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'b',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'n',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'm',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'del',
        'state': 1,
        'colored': true
      }
    ]
  ])

  const onKeyup = (e) => onKey(e.key)
  const tileClick = function(e) {
    if ( finished.value ){
      return
    }
    currentPosition.value = e.detail
  }
  window.addEventListener('keyup', onKeyup)
  const tileClickEvent = new Event('tile-click');
  window.addEventListener('tile-click',tileClick)
  onUnmounted(() => {
    window.removeEventListener('keyup', onKeyup)
    window.removeEventListener('tile-click',tileClick)
  })

  window.onload = () => {
    setInterval(() => {
      if ( remainingSeconds.value == 0 ){
        if (remainingMinutes.value > 0 || remainingHours.value > 0 ) {
         remainingSeconds.value = 59
        }
      } else {
        remainingSeconds.value--
      }
      if ( remainingSeconds.value == 59 ) {
        if ( remainingMinutes.value == 0 ) {
          if ( remainingHours.value > 0 ) {
            remainingMinutes.value == 59
            remainingHours.value--
          }
        } else {
          remainingMinutes.value--
        }
      }
    }, 1000)
  }

  onMounted(() => {
    var tooltipTriggerList = [].slice.call(document.querySelectorAll('[data-bs-toggle="tooltip"]'))
    var tooltipList = tooltipTriggerList.map(
      function (tooltipTriggerEl) {
        return new bootstrap.Tooltip(tooltipTriggerEl)
    })
  })

  const onKey = function(key) {
    if (showHelpModal.value || showWinModal.value) {
      return
    }
    if (/^[a-zA-Z_\-]$/.test(key)) {
      if (key == '-') {
        key = '_'
      }
      fillTile(key.toLowerCase())
    } else if (key === 'Backspace' || key === 'Delete') {
      clearTile()
    } else if (key === 'Enter') {
      completeRow(false)
    } else if ( key == 'ArrowLeft' && currentPosition.value > 0 ) {
      currentPosition.value--
    } else if ( ( key == 'ArrowRight' || key == ' ' ) && currentPosition.value < wordLength - 1 ) {
      currentPosition.value++
    }
  }

  const guessNotInDictionary = computed(() => {
    if ( guesses.value.length == 0 || currentGuess.value == guesses.value.length || currentGuess.value < 0 ){
      return false
    }

    let playerAnswer = guesses.value[currentGuess.value].map((e) => e['letter']).join('')
    if ( word == playerAnswer ){
      return false
    }

    if ( playerAnswer.length != wordLength || playerAnswer.match(/_/) ) {
      return false;
    }

    return !acceptedWordList.includes(playerAnswer.toLowerCase())
  })

  const showWordMissingMessage = function() {
    notInDictionary.value = true
    setTimeout(() => {
      notInDictionary.value = false
    },1500)
  }

  const fillTile = function(key){
    let tile = {}
    if ( currentPosition.value == wordLength ){
      tile = guesses.value[currentGuess.value][currentPosition.value - 1]
    } else {
      tile = guesses.value[currentGuess.value][currentPosition.value]
    }
    tile['letter'] = key
    if ( currentPosition.value < wordLength - 1  ){
      currentPosition.value++
    }
  }

  const clearTile = function() {
    if ( currentPosition.value > 0 && guesses.value[currentGuess.value][currentPosition.value]['letter'] == '' ){
      currentPosition.value--
    }
    let tile = guesses.value[currentGuess.value][currentPosition.value]
    tile['letter'] = ''
  }

  const updateKeyboard = function(letter,state) {
    let keyPosition = findKey(letter)
    if ( keyboardRows.value[keyPosition[0]][keyPosition[1]]['state'] < state ){
      keyboardRows.value[keyPosition[0]][keyPosition[1]]['state'] = state
    }
  }

  const findKey = function(letter) {
    for ( let i=0; i < 3; i++ ) {
      for ( let x=0; x < keyboardRows.value[i].length; x++ ){
        if ( letter === keyboardRows.value[i][x]['letter'] ) {
          return [i,x]
        }
      }
    }
  }

  const genGameResults = function() {
    let emoji = ['','','⚓','🧽','🦠']
    let results = "Squidwordle #" + gameNumber + " " + (correct.value ? playerGuessCount.value : 'X' ) + '/' +  numberOfGuesses + "\n"
    for ( let i=0; i < guesses.value.length; i++ ){
      let row = []
      for ( let x=0; x < wordLength; x++ ) {
        if ( guesses.value[i][x]['state'] == 0 ){
          break
        }
        row.push(emoji[guesses.value[i][x]['state']])
      }
      if ( row.length == word.length ){
        results += row.join('')
        results += "\n"
      }
    }
    if (gaveUp.value) {
      for ( let x=0; x < wordLength; x++ ) {
        results += '🦀'
      }
      results += "\n"
    }
    results += "https://theasylm.github.io/squidwordle"
    gameResults.value = results
  }

  const storeResults = function() {
    if ( correct.value ) {
      totalWins.value++
      window.localStorage.setItem('totalWins',totalWins.value)
      currentStreak.value++
      window.localStorage.setItem('currentStreak',currentStreak.value)
      if ( currentStreak.value > maxStreak.value ){
        maxStreak.value++
        window.localStorage.setItem('maxStreak',maxStreak.value)
      }
    } else {
      currentStreak.value = 0
      window.localStorage.setItem('currentStreak',currentStreak.value)
    }
    if ( correct.value ){
      guessCounts.value[playerGuessCount.value]++
    } else {
      guessCounts.value[0]++
    }
    window.localStorage.setItem('guessCounts',JSON.stringify(guessCounts.value))
    totalGames.value++
    window.localStorage.setItem('totalGames',totalGames.value)
  }
  const copyResults = function() {
    navigator.clipboard.writeText(gameResults.value)
    let span = document.getElementById('copiedResultsMessage')
    let classes = span.className
    span.className += 'shown'
    setTimeout(() => {
      span.className = classes
    }, 2000)
  }

  let giveUp = function() {
    showConfirmModal.value = false;
    finished.value = true
    gaveUp.value = true
    genGameResults()
    showWinModal.value = true
  }
</script>

<template>
  <div class="container wrap">
    <div class="header row">
      <div class="col-md-3">
      </div>
      <div class="col-md-6">
        <span class="title">Squidwordle #{{gameNumber}} </span>
      </div>
      <div class="col-md-3 help">
        <XIcon class="give-up-icon" @click="showConfirmModal = (true && !finished)"></XIcon>
        <ChartBarIcon @click="showWinModal = true"></ChartBarIcon>
        <QuestionMarkCircleIcon @click="showHelpModal = true"></QuestionMarkCircleIcon>
      </div>
    </div>
    <div class="info">
      <h5 class="warning-message" :class="{'shown': notInDictionary}">Word not in dictionary.</h5>
    </div>
    <Board :guesses="guesses" :guessNotInDictionary="guessNotInDictionary" :currentGuess="currentGuess" :currentPosition="currentPosition" :wordLength="wordLength"></Board>
    <Keyboard :rows="keyboardRows"></Keyboard>
    <div class="footer">
      Created by theasylm
    </div>
    <vue-final-modal
      name="confirmModal"
      classes="modal-container"
      :click-to-close="false"
      :esc-to-close="true"
      v-model="showConfirmModal"
      content-class="modal-content"
      :max-width="500"
    >
      <div class="close-modal-div">
        <XIcon @click="showConfirmModal = false"></XIcon>
      </div>
      <div class="modal__content confirm">
        <div class="hint-div">
          <div class="warning">Are you sure you wish to give up?</div>
        </div>
      </div>
      <div class="modal__action">
        <div class="mb-3 row">
          <div class="col-sm-4">
            <button @click="showConfirmModal = false" class="btn btn-primary">Keep Thinking</button>
          </div>
          <div class="col-sm-4">
            <button @click="giveUp" class="btn btn-danger">Give Up</button><br/>
          </div>
        </div>
      </div>
    </vue-final-modal>
    <vue-final-modal
      name="winModal"
      classes="modal-container"
      :click-to-close="false"
      :esc-to-close="true"
      v-model="showWinModal"
      content-class="modal-content"
      :max-width="500"
    >
      <div class="close-modal-div">
        <XIcon @click="showWinModal = false"></XIcon>
      </div>
      <span class="modal__title" id="win-msg">{{msg}}</span>
      <div class="modal__content">
        <div class="solution" v-if="finished && !correct">Solution: {{word.toUpperCase()}}</div>
        <div class="row stats">
          <div class="col-4">
            <span class="number">{{totalGames}}</span>
            <span class="stat-label">Total Games</span>
          </div>
          <div class="col-4">
            <span class="number">{{totalWins}}</span>
            <span class="stat-label">Total Wins</span>
          </div>
          <div class="col-4">
            <span class="number">{{winPercentage}}</span>
            <span class="stat-label">Win %</span>
          </div>
        </div>
        <div class="row stats">
          <div class="col-6">
            <span class="number">{{currentStreak}}</span>
            <span class="stat-label">Current Streak</span>
          </div>
          <div class="col-6">
            <span class="number">{{maxStreak}}</span>
            <span class="stat-label">Max Streak</span>
          </div>
        </div>
        <div class="row">
          <div class="col-12">Guess Distribution</div>
        </div>
        <div class="row">
          <div class="col-4">
            <div class="bar-label">X:</div>
            <div class="bar-label">1:</div>
            <div class="bar-label">2:</div>
            <div class="bar-label">3:</div>
            <div class="bar-label">4:</div>
            <div class="bar-label">5:</div>
            <div class="bar-label">6:</div>
          </div>
          <div class="col-8">
            <div class="bar" :style="{'width': xPercent}">{{guessCounts[0]}}</div>
            <div class="bar" :style="{'width': onePercent}">{{guessCounts[1]}}</div>
            <div class="bar" :style="{'width': twoPercent}">{{guessCounts[2]}}</div>
            <div class="bar" :style="{'width': threePercent}">{{guessCounts[3]}}</div>
            <div class="bar" :style="{'width': fourPercent}">{{guessCounts[4]}}</div>
            <div class="bar" :style="{'width': fivePercent}">{{guessCounts[5]}}</div>
            <div class="bar" :style="{'width': sixPercent}">{{guessCounts[6]}}</div>
          </div>
        </div>
        <div class="row">
          <div class="col-6">
            <span class="timer">{{remainingHours.toString().padStart(2,'0')}}:{{remainingMinutes.toString().padStart(2,'0')}}:{{remainingSeconds.toString().padStart(2,'0')}}</span>
            <span class="next">Next Squidwordle</span>

          </div>
          <div class="col-6">
            <button class="btn btn-primary share-button" @click="copyResults" :disabled="!finished">Share results</button><br/>
            <span id="copiedResultsMessage">Copied!</span>
          </div>
        </div>
      </div>
    </vue-final-modal>
    <vue-final-modal
      name="helpModal"
      classes="modal-container help-modal"
      :click-to-close="false"
      :esc-to-close="true"
      v-model="showHelpModal"
      content-class="modal-content"
      :max-width="600"
    >
      <div class="close-modal-div">
        <XIcon @click="showHelpModal = false"></XIcon>
      </div>
      <div class="modal__content">
        <h2>How to Play</h2>
        <div class="mb-3 row">
          <div class="col-sm-12">
            Guess the Wordle in the given number of tries. After each guess, the tiles will be colored to indicate how close to the target word your guess was.
            <img src="./assets/green_clue.png"/>
            Green indicates the Y is in the correct spot.
            <img src="./assets/yellow_clue.png"/>
            Yellow indicates the S is in the word, but in another position.
            <p><img src="./assets/grey_clue.png"/>
            Grey indicates the E is not in the word.</p>
          </div>
        </div>
      </div>
    </vue-final-modal>
  </div>
</template>

<style>
  body, html {
    height: 100%;
  }
  body {
    overflow: hidden;
  }
  #app {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    font-family: 'spongeboyregular';
    text-align: center;
    background-color: #b9d7cd;
    height: 100%;
    padding-top: 1em;
  }
  .wrap:before {
    content: ' ';
    display: block;
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    opacity: 0.25;
    background-image: url('../squidward.svg');
    background-repeat: no-repeat;
    background-position: center center;
    background-size: calc(100vh - 8rem);
  }

  .demo-content {
    position: relative;
  }
  .header,.info, .board {
    position: relative;
  }
  .title {
    font-size: 2em;
  }
  .warning-message {
    visibility: hidden;
    margin: 0;
  }
  .warning-message.shown {
    visibility: visible;
  }
  #copiedMessage, #copiedResultsMessage, #copiedJustMessage {
    visibility: hidden;
  }
  #copiedMessage.shown, #copiedResultsMessage.shown, #copiedJustMessage.shown {
    visibility: visible;
    opacity: 0;
    animation: fade 2s linear forwards;
  }
  .share-button {
    margin-top: 2rem;
  }
  .close-modal-div, .reset-modal {
    position: absolute;
    cursor: pointer;
  }

  .close-modal-div {
    margin-top: -.5rem;
    width: 38px;
    right: .5rem;
  }

  .reset-modal {
    margin-top: -.45rem;
    width: 32px;
    right:  42px;
  }

  .left-align {
    text-align: left;
  }
  .help svg {
    width: 36px;
    margin: 0 .25rem;
    cursor: pointer;
  }
  .help svg.inactive {
    opacity: 50%;
    cursor:  default;
  }
  .help-modal img {
    width: 100%;
  }
  .help-modal h2 {
    text-align: center;
  }
  .help-modal hr {
    height: 2px;
    background-color: #efefef;
    border: none;
    opacity: 1;
    margin-bottom: 0;
  }
  .solution {
    font-size: 1.5rem;
    font-weight: 500;
    margin-top: 1.5rem;
  }
  @keyframes fade {
    0%,100% { opacity: 0 }
    50% { opacity: 1 }
  }
  .add-link {
    cursor:  pointer;
    color: #efefef;
    text-decoration: none;
  }
  .add-link:hover {
    color: #efefef;
  }

  .info span {
    margin: 0 .5rem;
    font-size:  1.25rem;
  }
  table.table {
    color: #efefef;
    border-top: 2px solid;
    border-bottom: 2px solid;
  }
  .has-error {
    border-color: #842029 !important;
    border-width: 4px;
  }
  #win-msg {
    font-size:  2.25rem;
  }
  .give-up-icon {
    color:  #842029;
    width:  46px !important;
  }
  .warning {
    font-size: 2rem;
  }
  .slider {
    display: flex;
    align-items: center;
  }
  .left {
    justify-content: left;
    display: flex;
  }
  .footer {
    position: absolute;
    bottom: 0.5rem;
    font-size: smaller;
    width: 100%;
    left: 0;
  }
  span.number, span.timer {
    display: block;
    font-size: 2rem;
  }
  .timer {
    margin-top: .75rem;
  }
  .bar {
    background-color: #4192cf;
  }
  .bar-label {
    text-align: right;
  }
</style>
<style scoped>
  ::v-deep .modal-content {
    position: relative;
    display: flex;
    flex-direction: column;
    max-height: 90%;
    margin: 0 1rem;
    padding: 1rem;
    border: 1px solid #000;
    border-radius: 0.25rem;
    background: #b9d7cd;
    width: 700px;
    min-height: 10rem;
  }
  .modal__content {
    scrollbar-color: white;
  }
  .modal__content::-webkit-scrollbar {
    scrollbar-color: white;
  }
  ::v-deep .help-modal .modal-content {
    width:  650px;
    text-align: left;
  }
  .help-modal ul {
    list-style:  none;
  }
  .modal__title {
    font-size: 1.5rem;
    font-weight: 700;
  }
  ::v-deep .modal-container {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .modal__content {
    flex-grow: 1;
    overflow-y: auto;
  }
  .modal__close {
    position: absolute;
    top: 0.5rem;
    right: 1.5rem;
  }
  .modal__action {
    padding: 1rem 0 0;
  }
  .new-button {
    width: 10rem;
  }
  .new-button svg {
    float: right;
    width: 24px;
  }
  @media (max-width: 584px){
    .newModal .btn {
      margin: .5rem 0;
      height: 4rem;
    }
    .left .tooltip-icon {
      margin-top: 1.75rem;
    }
    .left button {
      width: 90%;
    }
  }
</style>