<template>
  <div class="home">
    <div class="main-title">{{ title }}</div>
    <div class="body-cont">
      <div class="side-cont">
        <v-progress-circular class="progress" size="240" width="1" value="100"/>
        <div class="text-cont">
          <div style="color: #ffffff; font-size: 48px; font-weight: 700; ">{{ currentRound.sb }}</div>
          <div style="color: #989898; font-size: 14px; font-weight: 700; margin-top: 16px; margin-bottom: 16px;">BLIND</div>
          <div style="color: #ffffff; font-size: 48px; font-weight: 700; ">{{ currentRound.bb }}</div>
        </div>
        <div class="next-text-cont">
          <div class="label">NEXT</div>
          <div class="content">{{ nextBlind }}</div>
        </div>
      </div>
      <div class="time-cont">
        <v-progress-circular v-if="isPlaying" rotate="270" class="progress-second" size="500" width="2" indeterminate/>
        <v-progress-circular rotate="270" class="progress-round" size="480" width="8" :value="roundProgress" />
        <v-progress-circular rotate="270" class="progress-total" size="450" width="20" :value="totalProgress"/>
        <div class="text-cont">
          <div style="color: white; font-size: 38px; font-weight: 600;">{{ `LEVEL ${currentRound.no + 1}`}}</div>
          <div style="color: #989898; font-size: 20px; margin-top: 16px;">TIME REMAINING</div>
          <div class="time">{{remainingTime}}</div>
          <div style="color: #989898; font-size: 20px; margin-top: 12px;">TOTAL TIME ELAPSED</div>
          <div style="color: #989898; font-size: 32px;">{{elapsedTime}}</div>
        </div>
      </div>
      <div class="side-cont">
        <v-progress-circular class="progress" size="240" width="1" value="100"/>
        <div class="text-cont">
          <div style="color: #989898; font-size: 14px; font-weight: 700; margin-bottom: 16px;">BLIND</div>
          <div style="color: #ffffff; font-size: 60px; font-weight: 700; ">{{ currentRound.ante }}</div>
        </div>
        <div class="next-text-cont">
          <div class="label">NEXT</div>
          <div class="content">{{ nextAnte }}</div>
        </div>
      </div>
    </div>
    <div class="controller-cont">
      <div class="button-cont">
        <button v-if="isOffSound" class="btn-s ml8" @click="toggleSound"><i class="material-icons big">volume_off</i></button>
        <button v-else class="btn-s ml8" @click="toggleSound"><i class="material-icons">volume_up</i></button>
        <button class="btn-s ml8" @click="prevRound"><i class="material-icons">navigate_before</i></button>
        <button v-if="isPlaying" class="btn-b ml8" @click="stop()"><i class="material-icons" style="font-size: 36px;">pause</i></button>
        <button v-else class="btn-b ml8" @click="play()"><i class="material-icons" style="font-size: 36px;">play_arrow</i></button>
        <button class="btn-s ml8" @click="nextRound"><i class="material-icons">navigate_next</i></button>
        <button class="btn-s ml8" @click="reset"><i class="material-icons">sync</i></button>
      </div>
      <button class="settings" @click="$router.push('/settings')">Settings</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data () {
    return {
      title: localStorage.getItem('main-title'),
      isPlaying: false,
      isOffSound: false,
      rounds: [],
      lastRound: 0,
      totalElapsed: 0,
      roundElapsed: 0,
      audio: undefined
    }
  },
  computed: {
    currentRound () {
      return this.rounds[this.lastRound] || {}
    },
    nextBlind () {
      const round = this.rounds[Number(this.lastRound) + 1] || {}
      return `${round.sb || '-'} / ${round.bb || '-'}`
    },
    nextAnte () {
      const round = this.rounds[Number(this.lastRound) + 1] || {}
      return `${round.ante || '-'}`
    },
    roundProgress () {
      return parseInt(this.roundElapsed * 100 / (this.currentRound.duration * 60 * 1000))
    },
    totalProgress () {
      return parseInt(this.totalElapsed * 100 / this.totalTime)
    },
    totalTime () {
      return this.rounds
        .map(item => parseInt(item.duration))
        .reduce((acc, cur) => acc + cur, 0) * 60 * 1000
    },
    remainingTime () {
      const roundMillis = this.currentRound.duration * 60 * 1000
      const remaining = (roundMillis - this.roundElapsed) / 1000
      let min = parseInt(remaining / 60)
      if (min < 10) {
        min = '0' + min
      }
      let sec = remaining % 60
      if (sec < 10) {
        sec = '0' + sec
      }
      return `${min}:${sec}`
    },
    elapsedTime () {
      let hh = parseInt(this.totalElapsed / 1000 / 60 / 60)
      if (hh < 10) {
        hh = '0' + hh
      }
      let mm = parseInt(this.totalElapsed / 1000 / 60)
      if (mm < 10) {
        mm = '0' + mm
      }
      let ss = (this.totalElapsed / 1000) % 60
      if (ss < 10) {
        ss = '0' + ss
      }
      return `${hh}:${mm}:${ss}`
    }
  },
  watch: {
    lastRound (value) {
      localStorage.setItem('lastRound', value)
    }
  },
  mounted () {
    const saved = localStorage.getItem('rounds')
    if (saved) {
      this.rounds = JSON.parse(saved)
      const lastRound = localStorage.getItem('lastRound')
      if (lastRound) {
        this.lastRound = lastRound
      } else {
        this.lastRound = 0
      }
      for (let i = 0; i < this.lastRound; i++) {
        this.totalElapsed += this.rounds[i].duration * 60 * 1000
      }
    } else {
      this.$router.push('/settings')
    }
    this.audio = new Audio(require('../assets/countdown.mp3'))
  },
  methods: {
    toggleSound () {
      this.isOffSound = !this.isOffSound
    },
    playSound () {
      if (!this.isOffSound) {
        this.audio.play()
      }
    },
    play () {
      this.isPlaying = true
      this.interval = setInterval(() => {
        const roundMillis = this.currentRound.duration * 60 * 1000
        if (roundMillis > this.roundElapsed) {
          this.roundElapsed += 1000
          this.totalElapsed += 1000
          if (roundMillis - this.roundElapsed === 4000) {
            this.playSound()
          }
        } else {
          this.nextRound()
        }
      }, 1000)
    },
    stop () {
      this.isPlaying = false
      clearInterval(this.interval)
    },
    prevRound () {
      const round = parseInt(this.lastRound)
      this.lastRound = round === 0 ? 0 : (round - 1)
      this.totalElapsed = this.totalElapsed - this.roundElapsed - (this.currentRound.duration * 60 * 1000)
      if (this.lastRound === 0) {
        this.totalElapsed = 0
      }
      this.roundElapsed = 0
    },
    nextRound () {
      const round = parseInt(this.lastRound)
      const roundMillis = this.currentRound.duration * 60 * 1000
      this.lastRound = round === this.rounds.length - 1 ? 0 : (round + 1)
      this.totalElapsed = this.totalElapsed - this.roundElapsed
      this.totalElapsed = this.totalElapsed + roundMillis
      if (this.lastRound === 0) {
        this.totalElapsed = 0
      }
      this.roundElapsed = 0
    },
    reset () {
      this.stop()
      this.lastRound = 0
      this.totalElapsed = 0
      this.roundElapsed = 0
    }
  }
}
</script>
<style lang="scss" scoped>

$primary-color: #BEAC74;

.home {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100vh;
  background-color: #111111;
  color: white;
  .main-title {
    position: absolute;
    top: 0;
    color: white;
    font-size: 48px;
    font-weight: 900;
    margin-top: 24px;
  }
  .body-cont {
    width: 100%;
    display: flex;
    flex-direction: row;
    flex-grow: 1;
    justify-content: center;
    align-items: center;
    .side-cont {
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      flex-grow: 1;
      .progress {
        color: $primary-color;
        position: absolute;
      }
      .text-cont {
        display: flex;
        flex-direction: column;

        align-items: center;
        justify-content: center;
      }
      .next-text-cont {
        margin-top: 440px;
        position: absolute;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        .label {
          font-size: 12px;
          font-weight: 700;
          color: $primary-color;
        }
        .content {
          font-size: 46px;
          font-weight: 700;
          color: white;
        }
      }
    }
    .time-cont {
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      flex-grow: 1;

      .progress-second {
        color: $primary-color;
        position: absolute;
      }
      .progress-round {
        color: $primary-color;
        position: absolute;
      }
      .progress-total {
        color: $primary-color;
        position: absolute;
      }
      .text-cont {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;

        .time {
          width: 300px;
          color: $primary-color;
          font-size: 106px;
          font-weight: 600;
          text-align: center;
        }
      }
    }
  }

  .controller-cont {
    width: 100%;
    padding-top: 16px;
    padding-bottom: 16px;
    display: flex;
    flex-direction: column;
    align-items: center;
    background-color: #222222;

    .button-cont {
      display: flex;
      flex-direction: row;
      flex-grow: 1;
      align-items: center;
      justify-content: center;
    }
    .settings {
      margin-top: 16px;
      margin-left: 8px;
      font-size: 16px;
      text-decoration: underline;
    }
  }
}
.ml8 {
  margin-left: 8px;
}
.mr8 {
  margin-right: 8px;
}
.material-icons {
  color: white;
}

.btn-s {
  display: flex;
  align-items: center;
  justify-content: center;
  outline: none;
  height: 48px;
  width: 48px;
  border-radius: 50%;
  border: 1px solid #111111;
  background: #111111;
}

.btn-b {
  outline: none;
  height: 80px;
  width: 80px;
  border-radius: 50%;
  border: 1px solid #111111;
  background: #111111;
}

</style>
