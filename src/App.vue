<script setup>
  import { $vfm, VueFinalModal, ModalsContainer } from 'vue-final-modal'
  import { ref, onUnmounted, onMounted, computed } from 'vue'
  import Board from './components/Board.vue'
  import Keyboard from './components/Keyboard.vue'
  import { PencilIcon, QuestionMarkCircleIcon, LightBulbIcon, XIcon, ChartBarIcon, RefreshIcon } from '@heroicons/vue/outline'
  import { allWords } from './assets/js/allWords.js'

  let word = 'krabs'
  const wordLength = word.length > 0 ? word.length : 0
  const allPossibleWords = allWords[wordLength]
  let numberOfGuesses = 6
  let gameNumber = 1
  let msg = ref("Aye-aye, captain!")
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

  const addEmptyRow = function() {
    let initialGuess = []
    for ( let i=0; i < wordLength; i++ ) {
      initialGuess.push({ 'letter': '', 'state': 0, 'initialized': false, 'colored': false, 'completed': false })
    }
    guesses.value.push(initialGuess)
  }

  for ( let i=0; i < numberOfGuesses; i++ ){
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

    return !allWords[wordLength].includes(playerAnswer.toUpperCase())
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

    if ( !correct.value && !allPossibleWords.includes(playerAnswer.toUpperCase()) && !skipAnimation ){
      showWordMissingMessage()
      return
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
    if (!skipAnimation ){
      if (playerGuessCount.value < numberOfGuesses ){
        playerGuessCount.value++
      } else {
        finished.value = true
      }
    }
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
    if (!correct.value){
      msg.value = 'Barnacles!'
    }
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
  <div class="container">
    <div class="header row">
      <div class="col-md-4">
      </div>
      <div class="col-md-4">
        <span class="title">Squidwordle</span>
      </div>
      <div class="col-md-4 help">
        <XIcon class="give-up-icon" @click="showConfirmModal = (true && !finished)"></XIcon>
        <ChartBarIcon :class="{ inactive: !finished }" @click="showWinModal = (true && finished)"></ChartBarIcon>
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
        <button class="btn btn-primary share-button" @click="copyResults">Share results</button><br/>
        <span id="copiedResultsMessage">Copied!</span>
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
  @font-face {
    font-family: 'spongeboyregular';
    src: url('spongeboyregular-gx2n6-webfont.woff2') format('woff2'),
         url('spongeboyregular-gx2n6-webfont.woff') format('woff');
    font-weight: normal;
    font-style: normal;

  }
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
  .header {
  }
  .title {
    font-size: 2em;
  }
  .info {
    position: relative;
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