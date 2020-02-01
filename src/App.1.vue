<template>
  <div id="app" class="main">
    <div class="header">
      <div class="wrapper wrapper-header">
        <div class="header__title">Lesson 3</div>
      </div>
    </div>
    <div class="wrapper wrapper-main">
      <div class="search__wrp">
        <input v-model="searchRow">
        <div class="search_list">
          <div 
            v-for="item in searchResults"
            :key="item.id"
            v-text="item"
            class="search_list__result"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import russian_names from '@/data/russian_names.json'
import russian_surnames from '@/data/russian_surnames.json'

export default {
  name: 'app',
  data: () => ({
    loading: true,
    name: '',
    surname: '',
    names: [],
    surnames: [],
    searchRow: "Иван",
    searchResults: []
  }),
  /* eslint-disable no-console */
  watch: {
    searchRow(newVal, oldVal) {
      if (!/^[А-я/s]*$/.test(newVal)) {
        this.searchRow = oldVal
      } else {
        this.startSearch()
      }
    }
  },
  methods: {
    startSearch() {
      if (this.name != this.searchRow.split(" ")[0]) {
        this.name = this.searchRow.split(" ")[0]

        if (this.name.length > 2) {
          this.getDataNamesFromAPI(this.name)
          console.log(this.names)
          this.formresArray()
        }
      }

      if (this.name != this.searchRow.split(" ")[0]) {
        this.surnames = this.searchRow.split(" ")[1]

        if (this.surname.length > 2) {
          console.log(this.surname)
          console.log(this.surnames)
          this.getDataNamesFromAPI(this.surname)
        }
      }
    },
    formresArray() {
      const self = this
      self.searchResults = []

      self.names.forEach(function(itemName) {
        if (self.surnames) {
          self.surnames.forEach(function(itemSurname) {
            self.searchResults.push(itemName + " " + itemSurname)
          });
        } else {
          self.searchResults.push(itemName + " ...")
        }
      });
    },
    getDataNamesFromAPI() {
      const self = this
      self.names = []

      console.log(this.name)
      Object.values(this.dataNamesFromDB()).forEach(function(em) {
        if (em.Name.toLowerCase().indexOf(self.name.toLowerCase()) == 0) {
           self.names.push(em.Name)
        }
      })
    },
    getDataSureNamesFromAPI() {
      const self = this
      self.surnames = []

      Object.values(this.dataSurenamesFromDB()).forEach(function(em) {
        if (em.Name.toLowerCase().indexOf(self.surname.toLowerCase()) == 0) {
           self.surnames.push(em.Name)
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
label
  display: inline-block
  font-size: 14px
  position: relative
  margin: 0 0 6px
  span
    font-size: 18px
    color: #f36f6f
    font-weight: 700
    position: absolute
    right: -15px
    top: -5px

input,
textarea
  width: 100%
  border: 2px solid #322f31
  font-size: 14px
  padding: 10px 14px
  transition: border .3s ease-out

input
  margin: 0 0 20px

textarea
  height: 100px

.search__wrp
  width: 600px
  max-width: 100%
  padding: 40px 0

.search_list__result
  font-size: 14px
</style>
