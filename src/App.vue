<script setup>
import { ref } from 'vue';
import restAudio from './assets/rest.mp3'
import hangAudio from './assets/hang.mp3'
import halfCrimpAudio from './assets/halfcrimp.mp3'
import threeAudio from './assets/threeFingerDrag.mp3'
import frontTwoAudio from './assets/frontTwo.mp3'
import middleTwoAudio from './assets/middleTwo.mp3'
import frontTwoCrimpAudio from './assets/frontTwoCrimp.mp3'
import middleTwoCrimpAudio from './assets/middleTwoCrimp.mp3'
import readyAudio from './assets/ready.mp3'
  
  const streak = ref("initializing")
  const timer = ref(10);
  var holdType = 0
  const hangStrings = ["Half Crimp", "Three Finger Drag", "Front Two Drag", "Middle Two Drag", "Front Two Crimp", "Middle Two Crimp"]
  const name = ref(hangStrings[holdType])
  var hold = false
  var announce = false
  var amount = 6
  var visible = ref(true)
  const getReady = new Audio(readyAudio)
  const rest = new Audio(restAudio)
  const hang = new Audio(hangAudio)
  const hangTypes = [new Audio(halfCrimpAudio), new Audio(threeAudio), new Audio(frontTwoAudio),
    new Audio(middleTwoAudio), new Audio(frontTwoCrimpAudio), new Audio(middleTwoCrimpAudio)
  ]
  const interval = ref(undefined)

  function getCookie(name) {
    let cookies = document.cookie.split(";");
    for(let i = 0; i < cookies.length; i++){
      let cookie = cookies[i]
      while(cookie.charAt(0) == ' '){
        cookie = cookie.substring(1);
      }
      if (cookie.indexOf(name) == 0) {
        return cookie.substring(name.length, cookie.length)
      }
    }
    return ""
  }

  function cookieUpdater() {
    const d = new Date();
    const d2 = new Date();
    const d3 = new Date();
    d3.setHours(0, 0, 0, 0)
    let lastTime = getCookie("lastTime=")
    lastTime =  new Date(Date.parse(lastTime))
    lastTime.setHours(0, 0, 0, 0)
    if ((lastTime.getTime() + 24*60*60*1000) !== d3.getTime() ) {
      console.log("day hasnt changed");
      return
    }
    const newStreak = parseInt(streak.value) + 1
    d.setTime(d.getTime() + (365*24*60*60*1000))
    document.cookie = "streak=" + newStreak + ";expires=" + d.toUTCString() + ";path=/"
    document.cookie = "lastTime=" + d2.toUTCString() + ";expires=" + d.toUTCString() + ";path=/"
    streak.value = newStreak

  }
  function init() {
    let expired = getCookie("lastTime=")
    const d = new Date();
    const d2 = new Date();
    if (expired !== "") {
      expired = Date.parse(expired)
      if ( (expired + 30*60*60*1000) < d.getTime() ) {
        d.setTime(d.getTime() + (365*24*60*60*1000))
        document.cookie = "streak=0;expires=" + d.toUTCString() + ";path=/"
        document.cookie = "lastTime=" + d2.toUTCString() + ";expires=" + d.toUTCString() + ";path=/"

      }
    } else {
      d.setTime(d.getTime() + (365*24*60*60*1000))
      document.cookie = "streak=0;expires=" + d.toUTCString() + ";path=/"
      document.cookie = "lastTime=" + d2.toUTCString() + ";expires=" + d.toUTCString() + ";path=/"
    }
    
    let num = getCookie("streak=")
    if (num !== "") {
      streak.value = num
    } else {
      streak.value = 0
    }
  }
  init()
  function start() {
    interval.value = setInterval(() => {
      if (hangTypes === 7) {
        cookieUpdater()
        clearInterval(interval.value)
        return
      }
      else if (!announce) {
        hangTypes[holdType].play()
        announce = !announce
      }
      else if ( amount === 0) {
        holdType++
        name.value = hangStrings[holdType]
        announce = !announce
        if (holdType === 1){
          amount = 6
        } else {
          amount = 2
        }
      }
      if ( timer.value <= 0 && !hold ) {
          timer.value = 10;
          hold = !hold;
          hang.play();

      } else if ( timer.value <= 0 && hold){
        timer.value = 20;
        hold = !hold;
        rest.play();
        amount--;
      } else if ( timer.value === 5 && !hold) {
        getReady.play();
        timer.value--;
      } else {
        timer.value--;
      }
    }, 1000)
  }
  function pause() {
    clearInterval(interval.value)
  }
  function reset() {
    amount = 6
    holdType = 0
    timer.value = 10
    clearInterval(interval.value)
  }
  function changeVisibility() {
    visible.value = !visible.value
  }
</script>


<template>
  <div>
    <h1>Hangboard stuff</h1>
    <h2>Current hang type: {{ name }}</h2>
    <h1 v-if="hold">HANG</h1>
    <h1 v-else>REST</h1>
    <p>
      {{timer}}
    </p>
    <p>Streak: {{ streak }} day(s)</p>
    <button v-if="visible" @click="start(); changeVisibility()">Begin</button>
    
    <button v-if="!visible" @click="pause(); changeVisibility();">Pause</button>
    <button @click="reset">Reset</button>
    <button @click="cookieUpdater">test</button>
  </div>
</template>

<style>

</style>