<template>
  <div id="app" class="main">
    <div class="header">
      <div class="wrapper wrapper-header">
        <div class="header__title">Lesson 3</div>
      </div>
    </div>
    <div class="wrapper wrapper-main">
      <div class="search__wrp">
        <h2>Поиск</h2>
        <h3>Фильтры:</h3>
        <div class="checkboxes">
          <div
            class="checkbox"
            :class="{ active: isUpper }"
          >
            <input
              v-model="isUpper"
              id="isUpperCheck"
              type="checkbox"
              @click="isLower = false"
            />
            <label for="isUpperCheck">Uppercase</label>
          </div>
          <div
            class="checkbox"
            :class="{ active: isLower }"
          >
            <input
              v-model="isLower"
              id="isLowerCheck"
              type="checkbox"
              @click="isUpper = false"
            />
            <label for="isLowerCheck">Lowercase</label>
          </div>
          <div
            class="checkbox"
            :class="{ active: isReverse }"
          >
            <input
              v-model="isReverse"
              id="isReverseCheck"
              type="checkbox"
            />
            <label for="isReverseCheck">Reverse</label>
          </div>
          <div
            class="checkbox"
            :class="{ active: isTranslite }"
          >
            <input
              v-model="isTranslite"
              id="isTransliteCheck"
              type="checkbox"
            />
            <label for="isTransliteCheck">Translite</label>
          </div>
        </div>
        <div class="search__content">
          <input
            v-model="searchRow"
            placeholder="Введите имя и фамилию"
            class="textInput"
          />
          <div
            class="search_list"
            :class="{ active: searchRow.length }"
          >
            <div
              class="search__close"
              @click="searchRow = ''"
            />
            <div v-if="searchRow.length <= 2" v-text="'Введите больше символов'" class="search_list__result" />
            <div v-else>
              <div 
                v-for="item in searchResults"
                :key="item.id"
                class="search_list__result"
              >{{
                item
                | filterUppercase(isUpper)
                | filterLowercase(isLower)
                | filterReverse(isReverse)
                | filterTranslite(isTranslite)
              }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import russian_names from '@/data/russian_names.json'
import russian_surnames from '@/data/russian_surnames.json'
import api from '@/mixins/api'

export default {
  name: 'app',
  mixins: [ api ],
  data: () => ({
    loading: true,
    name: '',
    surname: '',
    names: [],
    surnames: [],
    searchRow: '',
    searchResults: [],
    isUpper: false,
    isLower: false,
    isReverse: false,
    isTranslite: false
  }),

  /* eslint-disable no-console */
  watch: {
    searchRow(newVal, oldVal) {
      if (!/^[А-я\s]*$/.test(newVal)) {
        this.searchRow = oldVal
      } else if (newVal == ' ') {
        this.searchRow = oldVal
      } else {
        this.startSearch()
      }
    }
  },

  filters: {
    filterUppercase: function (value, isUpper) {
      return isUpper ? value.toUpperCase() : value
    },

    filterLowercase: function (value, isLower) {
      return isLower ? value.toLowerCase() : value
    },

    filterReverse: function (value, isReverse) {
      return isReverse ? value.split("").reverse().join("") : value
    },

    filterTranslite: function (value, isTranslite) {
      if (isTranslite) {
        let ru = {
            'а': 'a', 'б': 'b', 'в': 'v', 'г': 'g', 'д': 'd', 
            'е': 'e', 'ё': 'e', 'ж': 'j', 'з': 'z', 'и': 'i', 
            'к': 'k', 'л': 'l', 'м': 'm', 'н': 'n', 'о': 'o', 
            'п': 'p', 'р': 'r', 'с': 's', 'т': 't', 'у': 'u', 
            'ф': 'f', 'х': 'h', 'ц': 'c', 'ч': 'ch', 'ш': 'sh', 
            'щ': 'shch', 'ы': 'y', 'э': 'e', 'ю': 'u', 'я': 'ya'
        },
        n_str = []
        
        value = value.replace(/[ъь]+/g, '').replace(/й/g, 'i')
        
        for ( var i = 0; i < value.length; ++i ) {
          n_str.push(
              ru[ value[i] ]
              || ru[ value[i].toLowerCase() ] == undefined && value[i]
              || ru[ value[i].toLowerCase() ].replace(/^(.)/, function ( match ) { return match.toUpperCase() })
          );
        }
        
        value = n_str.join('')
      }
      return value
    }
  },

  methods: {
    startSearch() {
      if (this.loading) {
        this.loading = false

        setTimeout(() => {
          if (this.name != this.searchRow.split(" ")[0]) {
            this.name = this.searchRow.split(" ")[0]

            if (this.name.length > 2) {
              this.getDataNamesFromAPI(this.name)
            } else {
              this.names = []
            }

            this.formresArray()
          }

          if (this.surname != this.searchRow.split(" ")[1]) {
            this.surname = this.searchRow.split(" ")[1]

            if (this.surname && this.surname.length > 2) {
              this.getDataSureNamesFromAPI(this.surname)
            } else {
              this.surnames = []
            }

            this.formresArray()
          }

          this.loading = true
        }, 1000)
      }
    },

    formresArray() {
      const self = this

      self.searchResults = []
    
      if (self.surnames.length) {
        self.surnames.forEach(function(itemSurname) {
          self.searchResults.push(self.capitalize(self.name) + " " + itemSurname)
        });
      } else {
        self.names.forEach(function(itemName) {
          self.searchResults.push(itemName + " ...")
        });
      }
    },

    getDataNamesFromAPI() {
      const self = this

      self.names = []

      this.dataNamesFromDB().forEach(function(em) {
        if (em.Name.toLowerCase().indexOf(self.name.toLowerCase()) == 0) {
          self.names.push(em.Name)
        }
      })
    },

    getDataSureNamesFromAPI() {
      const self = this
      
      self.surnames = []
  
      this.dataSurenamesFromDB().forEach(function(em) {
        if (em.Surname.toLowerCase().indexOf(self.surname.toLowerCase()) == 0) {
           self.surnames.push(em.Surname)
        }
      })
    },

    dataNamesFromDB() {
      return russian_names
    },

    dataSurenamesFromDB() {
      return russian_surnames
    }
  }
}
</script>

<style lang="sass">
.textInput
  width: 100%
  border: 2px solid #322f31
  font-size: 14px
  padding: 10px 14px
  transition: border .3s ease-out

.search__wrp
  width: 600px
  max-width: 100%
  padding: 40px 0

.checkboxes
  display: flex
  flex-wrap: wrap

.checkbox
  min-height: 24px
  margin: 0 34px 14px 0
  &:last-childe
    margin: 0 0 14px
  &.active
    label
      &:before
        background: url(assets/img/check.svg) center no-repeat
        background-size: 65%
  input
    display: none
  label
    font-size: 14px
    cursor: pointer
    position: relative
    padding: 4px 0 0 36px
    &:before
      content: ""
      width: 20px
      height: 20px
      border: 2px solid #322f31
      position: absolute
      left: 0
      top: 0

.search__content
  position: relative

.search_list
  width: 100%
  min-height: 40px
  max-height: 400px
  background: rgba(0, 0, 0, .85)
  overflow-y: auto
  position: absolute
  left: 0
  top: 40px
  padding: 10px 16px
  transform: scale(0)
  transform-origin: left top
  transition: transform .3s ease-out
  &.active
    transform: scale(1)

.search_list__result
  font-size: 14px
  color: #fff
  margin: 0 0 6px
  &:last-child
    margin: 0

.search__close
  width: 20px
  height: 20px
  background: url(assets/img/close.svg) center no-repeat
  background-size: 65%
  cursor: pointer
  position: absolute
  right: 16px
  top: 10px
  transition: opacity .3s ease-out
  &:hover
    opacity: .7
</style>
