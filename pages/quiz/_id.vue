<template>
  <div>
    <h1>{{ title }}</h1>
    <v-text-field v-if="!gaveUp" @input="handleInput" label="Enter place name" v-model="name"></v-text-field>
    <h2>{{ collectedNames.length }} / {{ names.length }}</h2>
    <h2 v-if="started && !gaveUp">{{ duration }} s</h2>
    <v-btn v-if="started && !gaveUp" color="warning" @click="giveUp">Give up</v-btn>
    <v-row v-if="started && !gaveUp && sortedCollectedNames.length > 0">
      <v-col
        v-for="cn in sortedCollectedNames"
        :key="cn"
        sm="4"
        lg="2"
        md="3"
        cols="6"><v-alert>{{ cn }}</v-alert></v-col>
    </v-row>
    <v-btn v-if="gaveUp" @click="retry">Retry</v-btn>
    <v-list v-if="gaveUp">
      <v-list-item v-for="cn in sortedNames" :key="cn" :class="{ missed: !lowerCaseCollectedNames.includes(cn.toLowerCase())}">
        {{cn}} <v-icon v-if="lowerCaseCollectedNames.includes(cn.toLowerCase())">mdi-check-bold</v-icon>
      </v-list-item>
    </v-list>
  </div>
</template>
<style>
.missed {
  color: red !important;
  font-weight: bold
}
</style>
<script>

export default {
  name: 'Quiz',
  data: function() {
    return {
      timer: 0,
      started: false,
      startTime: 0,
      quizid: '',
      title: '',
      name: '',
      collectedNames: [],
      names: [],
      gaveUp: false
    }
  },
  asyncData: function(req) {
    const data = require(`assets/data/${req.params.id}.json`)
    return { quizid: req.params.id, names: data.data, title: data.title }
  },
  methods: {
    handleInput: function() {
      if (!this.started) {
        this.started = true
        this.startTime = new Date().getTime()
      }
      const n = this.name.toLowerCase().trim()
      if (this.lowerCaseNames.includes(n) && 
          !this.lowerCaseCollectedNames.includes(n)) {
        this.collectedNames.push(this.name)
        const self = this
        setTimeout(() => {
          self.name = ''
        }, 10)
      }
    },
    giveUp: function() {
      this.gaveUp = true
    },
    retry: function() {
      this.started = false
      this.collectedNames = []
      this.gaveUp = false
    }
  },
  mounted: function () {
    // increment the timer every second - to force the computed function to recalculate
    setInterval(() => { this.timer++ }, 1000)
  },
  computed: {
    lowerCaseNames: function() {
      return this.names.map((n) => { return n.toLowerCase()} )
    },
    lowerCaseCollectedNames: function() {
      return this.collectedNames.map((n) => { return n.toLowerCase()})
    },
    sortedNames: function() {
      return this.names.sort()
    },
    sortedCollectedNames: function() {
      return this.collectedNames.sort()
    },
    duration: function() {
      this.timer
      const now = new Date().getTime()
      return Math.floor((now - this.startTime)/1000)
    }
  }
}
</script>