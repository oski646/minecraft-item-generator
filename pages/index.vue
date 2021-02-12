<template>
  <b-container class="mt-4">
    <b-row>
      <h2 class="mx-auto font-weight-bold mb-4">Generator przedmiotów</h2>
    </b-row>
    <b-row>
      <b-col xl="6" md="12">
        <h3 class="text-center">Podgląd przedmiotu</h3>
        <div class="d-flex justify-content-center align-items-center">
          <div id="minecraft-item" class="minecraft-item text-truncate my-4">
            <div
              class="minecraft-item-name"
              v-html="parseColoredLine(inputs.name + ' &f(#0000/0)')"
            />
            <div
              v-for="(line, index) in lore"
              :key="index"
              v-html="parseColoredLine(line.text)"
            />
            <div v-if="inputs.showMaterial" style="color: #555555">
              <div>minecraft:material</div>
              <div>NBT: 1 tag(s)</div>
            </div>
          </div>
        </div>
        <div class="d-flex justify-content-center mt-3">
          <b-btn variant="primary" @click="downloadFile()">
            <div class="d-flex align-items-center">
              Pobierz zawartość przedmiotu <BIconDownload class="ml-2" />
            </div>
            <div>
              <small>{{ getFixedName() }}.txt</small>
            </div>
          </b-btn>
        </div>
      </b-col>
      <b-col xl="6" md="12">
        <h3 class="text-center mt-5 mt-xl-0">Zarządzanie</h3>
        <b-form-group
          id="input-group-0"
          label="Nazwa przedmiotu"
          label-for="input-0"
        >
          <b-form-input
            id="input-0"
            v-model="inputs.name"
            type="text"
            required
          ></b-form-input>
        </b-form-group>
        <b-form-group>
          <b-checkbox v-model="inputs.showMaterial">
            Pokaż materiał przedmiotu
          </b-checkbox>
        </b-form-group>
        <b-form-group label="Opis przedmiotu">
          <div
            v-for="(line, index) in lore"
            :key="index"
            class="d-flex justify-content-center align-items-center mb-3"
          >
            <b-form-input
              v-model="line.text"
              type="text"
              required
            ></b-form-input>
            <b-btn
              class="ml-2"
              variant="dark"
              :disabled="lore.length === 1 || index === 0"
              @click="moveUp(index)"
            >
              <BIconArrowUp />
            </b-btn>
            <b-btn
              class="ml-2"
              variant="dark"
              :disabled="lore.length === 1 || lore.length === index + 1"
              @click="moveDown(index)"
            >
              <BIconArrowDown />
            </b-btn>
            <b-btn
              class="ml-2"
              variant="danger"
              :disabled="lore.length === 1"
              @click="removeLine(index)"
            >
              <BIconTrash />
            </b-btn>
          </div>
        </b-form-group>
        <b-btn variant="success" block @click="addLine()">Dodaj linijkę</b-btn>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import 'assets/global.scss'
import MinecraftTextJS from 'minecraft-text-js'
import FileSaver from 'file-saver'
import {
  BIconTrash,
  BIconArrowUp,
  BIconArrowDown,
  BIconDownload,
} from 'bootstrap-vue'
export default {
  components: {
    BIconTrash,
    BIconArrowDown,
    BIconArrowUp,
    BIconDownload,
  },
  data() {
    return {
      inputs: {
        name: '',
        showMaterial: false,
        showPolishCharacters: false,
      },
      lore: [],
    }
  },
  mounted() {
    this.lore.push({ text: '' })
  },
  methods: {
    parseColoredLine(text) {
      return MinecraftTextJS.toHTML(text)
    },
    moveInArray(arr, from, to) {
      const item = arr.splice(from, 1)
      if (item.length) {
        arr.splice(to, 0, item[0])
      }
    },
    addLine() {
      this.lore.push({ text: '' })
    },
    removeLine(index) {
      if (this.lore.length > 1) {
        this.lore.splice(index, 1)
      }
    },
    moveUp(index) {
      this.moveInArray(this.lore, index, index - 1)
    },
    moveDown(index) {
      this.moveInArray(this.lore, index, index + 1)
    },
    getFixedName() {
      const nameWords = this.inputs.name.split(' ')
      let fixedName = ''
      nameWords.forEach((word, index) => {
        if (word.includes('&')) {
          const chars = word.split('')
          chars.forEach((char, charIndex) => {
            if (char === '&') {
              const toRemove = char + chars[charIndex + 1]
              word = word.replace(toRemove, '')
            }
          })
          fixedName += word
        } else {
          fixedName += word
        }
        fixedName += index === nameWords.length ? '' : ' '
      })
      return fixedName
    },
    downloadFile() {
      const lore = []
      this.lore.forEach((line) => {
        lore.push(line.text + '\n')
      })
      const blob = new Blob(
        [this.inputs.name + '\n--------------\n', ...lore],
        {
          type: 'text/plain;charset=utf-8',
        }
      )
      FileSaver.saveAs(blob, this.getFixedName() + '.txt')
    },
  },
}
</script>

<style lang="scss">
.minecraft-item {
  font-family: 'Minecraft PL', sans-serif;
  background-color: #120312;
  border: 3px solid #250359;
  padding: 10px;
  color: white;
  border-radius: 5px;
  display: inline-block;
  max-width: 100%;
  user-select: none;
}
</style>
