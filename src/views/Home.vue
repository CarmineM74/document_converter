<template>
  <div class="flex">
    <div id="sidebar" class="w-1/5 min-h-screen border-r-2 border-grey-dark bg-grey p-2">
      <div class="border-grey-darker border-2 shadow-md hover:shadow-lg hover:cursor-pointer rounded-lg bg-grey-dark text-center text-white font-semibold mb-2 p-2"
          @click="openDialog">Carica file</div>
      <div v-if="xmlPath && (xmls.length != 0)"
          class="border-grey-darker border-2 shadow-md hover:shadow-lg hover:cursor-pointer rounded-lg bg-grey-dark text-center text-white font-semibold p-2"
          @click="convertFiles">Converti file</div>
      <div v-if="xmlPath && (xmls.length != 0)" class="mt-2 rounded-lg shadow w-full bg-grey-light">
        <div v-if="progress > 0" class="rounded-lg bg-green font-semibold text-xs leading-normal py-1 text-center text-white" :style="{width: progress + '%'}">{{ progress }}%</div>
      </div>
      <div v-if="converted"
          class="mt-2 border-grey-darker border-2 shadow-md hover:shadow-lg hover:cursor-pointer rounded-lg bg-grey-dark text-center text-white font-semibold p-2"
          @click="save">Salva XLS</div>
    </div>
    <div id="content" class="w-4/5 h-full ml-2 p-2">
      <h2 class="font-semibold tracking-wide text-center">Elenco file XML</h2>
      <h3 class="font-normal text-sm">File XML individuati: {{xmls.length}}</h3>
      <div class="border-b border-grey-dark mt-2"></div>
      <ul class="list-reset p-2 mt-4 flex flex-col">
        <li class="w-full border-b-2 mb-2 border-red" v-for="(msg, idx) in msgs" :key="idx">{{ msg }}</li>
      </ul>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src

import Vue from 'vue'

import fs from 'fs-extra'

// eslint-disable-next-line
const { dialog } = require('electron').remote
const convert = require('xml-js')
import XLSX from 'xlsx'

export default Vue.extend({
  name: 'home',
  data() {
    return {
      xmlPath: undefined,
      msgs: [],
      xmls: [],
      parsed: [],
      processed: []
    }
  },
  components: {
  },
  computed: {
    progress() {
      return ((this.parsed.length/this.xmls.length) * 100).toPrecision(3)
    },
    converted() {
      return this.parsed.length == this.xmls.length
    }
  },
  methods: {
    openDialog() {
      this.xmls = []
      this.xmlPath = dialog.showOpenDialog({
        title: 'Seleziona cartella XML',
        properties: ['openDirectory']})
      console.log('Choosen path', this.xmlPath)
      if (this.xmlPath) {
        fs.readdir(this.xmlPath[0], { withFileTypes: true }, (err, files) => {
          this.xmls = this._.filter(files, (f) => { return f.toLowerCase().endsWith('xml') })
        })
      }
    },
    convertFiles() {
      this.converted = false
      this.parsed = []
      console.log('Converting ...')
      this._.each(this.xmls, (f) => {
        fs.readFile(this.xmlPath + '/' + f, 'utf8', (e, d) => {
          this.msgs.push("Elaborazione di " + f +" in corso ...")
          const parsed = JSON.parse(convert.xml2json(d, {compact: true, spaces: 2}))
          this.parsed.push(parsed.FlussoMisure)
          Vue._.each(this.processFile(parsed.FlussoMisure), (r) => {
            this.processed.push(r)
            const opts = {compact: true, ignoreComment: true, spaces: 4}
            convert.json2xml(r, opts)
          })
        })
      })
      console.log('DONE CONVERTING TO JSON')
    },
    save() {
      this.msgs = ["Salvatggio in corso ..."]
      let rows = [[]]
      Vue._.each(this.processed[0], (v,k) => {
        rows[0].push(k)
      })
      Vue._.each(this.processed, (p) => {
        rows.push(Vue._.map(p, (v) => { return v }))
      })
      const ws = XLSX.utils.aoa_to_sheet(rows)
      const wb = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(wb, ws, "Dati")
      XLSX.writeFile(wb, "dati.xlsx")
      this.msgs.push("Salvataggio completato")
    },
    extractBase(j) {
      return {
        CodFlusso: j._attributes.CodFlusso,
        PIvaUtente: j.IdentificativiFlusso.PIvaUtente._text,
        PIvaDistributore: j.IdentificativiFlusso.PIvaDistributore._text,
        CodContrDisp: j.IdentificativiFlusso.CodContrDisp._text,
      }
    },
    extractPods(j) {
      const pods = j.DatiPod.length ? j.DatiPod : [ j.DatiPod]
        return Vue._.map(pods, (pod) => {
          return {
            Pod: pod.Pod._text,
            DataMisura: pod.DataMisura._text,
            Trattamento: pod.DatiPdp.Trattamento._text,
            Tensione: pod.DatiPdp.Tensione._text,
            PotContrImp: pod.DatiPdp.PotContrImp._text,
            PotDisp: pod.DatiPdp.PotDisp._text,
            Ka: pod.DatiPdp.Ka._text,
            Kr: pod.DatiPdp.Kr._text,
            Kp: pod.DatiPdp.Kp._text,
            MatrAtt: pod.DatiPdp.MatrAtt._text,
            MatrRea: pod.DatiPdp.MatrRea._text,
            MatrPot: pod.DatiPdp.MatrPot._text,
            DataInsMisAtt: pod.DatiPdp.DataInstMisAtt._text,
            DataInsMisRea: pod.DatiPdp.DataInstMisRea._text,
            DataInsMisPot: pod.DatiPdp.DataInstMisPot._text,
            GruppoMis: pod.DatiPdp.GruppoMis._text,
            Forfait: pod.DatiPdp.Forfait._text,
            Raccolta: pod.Misura.Raccolta._text,
            TipoDato: pod.Misura.TipoDato._text,
            Validato: pod.Misura.Validato._text,
            EaF1: pod.Misura.EaF1._text,
            EaF2: pod.Misura.EaF2._text,
            EaF3: pod.Misura.EaF3._text,
            ErF1: pod.Misura.ErF1._text,
            ErF2: pod.Misura.ErF2._text,
            ErF3: pod.Misura.ErF3._text,
            PotF1: pod.Misura.PotF1._text,
            PotF2: pod.Misura.PotF2._text,
            PotF3: pod.Misura.PotF3._text
          } 
        })
    },
    processFile(j) {
      console.log("J", j)
      const base = this.extractBase(j)
      const pods = this.extractPods(j)
      console.log(pods.length)
      if (pods.length > 1) {
        console.log("MULTIPLE PODS")
        return Vue._.map(pods, (p) => { return { ...base, ...p } })
      } else {
        console.log("SINGLE POD")
        return [{...base, ...pods[0]}]
      }
    }
  }
})

</script>
