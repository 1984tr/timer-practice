<template>
  <div class="settings">
    <div class="container">
      <button class="btn-back" @click="saveRound">BACK</button>
      <input class="main-title" type="text" :value="title" @input="title = $event.target.value">
      <div class="row-header">
        <div class="label-header"></div>
        <input class="label" type="text" value="SMALL BLIND" disabled>
        <input class="label" type="text" value="BIG BLIND" disabled>
        <input class="label" type="text" value="ANTE" disabled>
        <input class="label" type="text" value="DURATION" disabled>
        <div class="label-footer"></div>
      </div>
      <div v-for="round in rounds" :key="round.no" class="row-item">
        <div class="label">{{ `LEVEL ${(round.no + 1)}` }}</div>
        <input class="input-label" type="text" :value="round.sb" @input="round.sb = $event.target.value">
        <input class="input-label" type="text" :value="round.bb" @input="round.bb = $event.target.value">
        <input class="input-label" type="text" :value="round.ante" @input="round.ante = $event.target.value">
        <input class="input-label" type="text" :value="round.duration" @input="round.duration = $event.target.value">
        <button class="delete" @click="deleteRound(round)"><i class="material-icons" style="font-weight: 700; text-align:center; font-size: 22px; color: white;">clear</i></button>
      </div>
      <div class="bottom-btns">
        <button class="btn-add" @click="insertRound">ADD NEW LEVEL</button>
        <button class="btn-back2" @click="saveRound">BACK</button>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'Settings',
  data () {
    return {
      title: '',
      subtitle: '',
      rounds: [{ no: 0, sb: 25, bb: 50, ante: 0, duration: 20 }]
    }
  },
  computed: {
    defaultRounds () {
      const rounds = []
      for (let i = 0; i < 20; i++) {
        rounds.push({ no: i, sb: 25, bb: 50, ante: 25, duration: 20 })
      }
      return rounds
    }
  },
  mounted () {
    const saved = localStorage.getItem('rounds')
    if (saved) {
      this.rounds = JSON.parse(saved)
    } else {
      const json = JSON.stringify(this.defaultRounds)
      localStorage.setItem('rounds', json)
      localStorage.setItem('lastRound', 0)
      this.rounds = this.defaultRounds
    }
    this.title = localStorage.getItem('main-title') || 'Hold\'em timer'
  },
  methods: {
    insertRound () {
      this.rounds.push({ no: this.rounds.length, sb: 25, bb: 50, ante: 0, duration: 20 })
    },
    deleteRound (round) {
      const idx = this.rounds.indexOf(round)
      if (idx > -1) {
        this.rounds.splice(idx, 1)
      }
      for (let i = 0; i < this.rounds.length; i++) {
        this.rounds[i].no = i
      }
    },
    saveRound () {
      const json = JSON.stringify(this.rounds)
      localStorage.setItem('rounds', json)
      localStorage.setItem('lastRound', 0)
      localStorage.setItem('main-title', this.title)
      this.$router.go(-1)
    }
  }
}
</script>
<style lang="scss" scoped>
.settings {
  display: flex;
  flex-direction: column;
  color: white;
  align-items: flex-start;
  background-color: #111111;

  .container {
    display: flex;
    flex-direction: column;
    padding: 0px;
    background-color: #222222;
    align-items: stretch;

    .btn-back {
      margin: 8px;
      border-radius: 5px;
      padding-top: 8px;
      padding-bottom: 8px;
      font-size: 24px;
      font-weight: 700;
      background-color: #0194FD;

      &:hover {
        background-color: #1da1ff;
      }
    }
    .main-title {
      color: white;
      font-size: 48px;
      font-size: 24px;
      text-align: center;
      font-weight: 600;
      border-radius: 5px;
      padding-top: 16px;
      padding-bottom: 16px;
      margin-left: 8px;
      margin-right: 8px;
      margin-bottom: 48px;
      margin-top: 16px;
      background-color: #363636;
    }
    .row-header {
      display: flex;
      flex-direction: row;
      align-self: stretch;
      padding-left: 8px;
      padding-right: 8px;

      .label-header {
        flex-shrink: 0;
        flex-basis: 180px;
      }
      .label {
        flex-grow: 1;
        min-width: 0px;
        font-size: 16px;
        font-weight: 500;
        text-align: center;
        background-color: transparent;
        color: white;
      }
      .label-footer {
        flex-shrink: 0;
        flex-basis: 48px;
        margin-left: 8px;
      }
    }
    .row-item {
      display: flex;
      flex-direction: row;
      align-self: stretch;
      margin-top: 4px;
      margin-bottom: 4px;
      padding-left: 8px;
      padding-right: 8px;

      .label {
        flex-shrink: 0;
        flex-basis: 180px;
        font-size: 22px;
        padding: 8px;
        font-weight: 600;
        text-align: center;
        border-radius: 5px;
        background-color: #0379FB;
      }
      .input-label {
        flex-grow: 1;
        min-width: 0px;
        font-size: 24px;
        text-align: center;
        font-weight: 600;
        border-radius: 5px;
        margin-left: 8px;
        background-color: #363636;
        color: white;
      }
      .delete {
        flex-shrink: 0;
        flex-basis: 48px;
        margin-left: 8px;
        padding: 8px;
        font-weight: 600;
        text-align: center;
        border-radius: 5px;
        background-color: #989898;
        &:hover {
          background-color: #bbbbbb;
        }
      }
    }

    .bottom-btns {
      display: flex;
      flex-direction: row;
      align-self: stretch;

      .btn-add {
        margin: 8px;
        border-radius: 5px;
        padding-top: 8px;
        padding-bottom: 8px;
        font-size: 24px;
        font-weight: 700;
        background-color: #333333;
        flex-grow: 2;

        &:hover {
          background-color: #535353;
        }
      }
      .btn-back2 {
        margin-top: 8px;
        margin-right: 8px;
        margin-bottom: 8px;
        border-radius: 5px;
        padding-top: 8px;
        padding-bottom: 8px;
        font-size: 24px;
        font-weight: 700;
        background-color: #0194FD;
        flex-grow: 1;

        &:hover {
          background-color: #1da1ff;
        }
      }
    }
  }
}
</style>
