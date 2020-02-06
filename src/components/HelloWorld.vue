<template>
  <div class="hello">
    <p>Hello World!</p>
    <p>Hello World Again! New message. Hot Reload works well</p>
    <div class="buttons-container">
      <button
        type="button"
        class="btn btn-outline-success"
        v-on:click="selectVolumeData('маленький')"
      >
        Маленький
      </button>
      <button
        type="button"
        class="btn btn-outline-danger"
        v-on:click="selectVolumeData('большой')"
      >
        Большой
      </button>
    </div>
    <!-- <div class="loading-container" v-if="loading">LOADING ...</div> -->
    <div v-show="isIndexSelected">
      <pre> {{ people[index] }} </pre>
    </div>
    <div class="search-container">
      <input type="text" v-model="query" />
      <button class="btn btn-outline-warning">Искать</button>
    </div>
    <table class="table table-hover">
      <thead>
        <tr>
          <th scope="col">ID<span v-on:click="property = 'id'">SORT</span></th>
          <th scope="col">Имя</th>
          <th scope="col">Фамилия</th>
          <th scope="col">Электропочта</th>
          <th scope="col">Телефон</th>
          <th scope="col">Адрес</th>
          <th scope="col">Описание</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="loading">
          <td class="table-success" colspan="7">загрузка...</td>
        </tr>
        <table-element
          v-for="(person, index) in selectedPeople"
          v-bind:key="index"
          v-bind:person="person"
          v-on:click.native="selectPerson(index)"
        ></table-element>
      </tbody>
    </table>
    <pagination></pagination>
  </div>
</template>

<script>
import TableElement from '@/components/TableElement.vue'
import Pagination from '@/components/Pagination.vue'

export default {
  name: 'HelloWorld',
  components: {
    TableElement,
    Pagination
  },
  props: {
    msg: String
  },
  data() {
    return {
      //settings
      size: '',
      url: '',
      loading: false,

      people: [],

      // select person
      index: null,

      // filter
      property: '',
      order: 'убывание',
      query: '',

      // pagination
      pages: null,
      currentPage: null,
      perPage: 5

      // add
    }
  },
  computed: {
    isIndexSelected() {
      return this.index !== null
    },
    selectedPeople() {
      let filteredPeople = [...this.people]

      if (this.property) {
        const ascending = (a, b) =>
          a[`${this.property}`] - b[`${this.property}`]
        const descending = (a, b) =>
          b[`${this.property}`] - a[`${this.property}`]
        let expression

        if (this.order === 'возрастание') {
          expression = ascending
        } else if (this.order === 'убывание') {
          expression = descending
        }

        filteredPeople.sort(expression)
      }

      if (this.query) {
        filteredPeople.filter(person => {
          console.log(JSON.stringify(person).includes(this.query))
          return JSON.stringify(person).includes(this.query)
        })
      }

      return filteredPeople
    }
  },
  watch: {
    size: async function() {
      this.selectURLtoFetch()
      await this.fetchData()
    }
  },
  methods: {
    selectPerson(index) {
      this.index = index
    },
    selectVolumeData(size) {
      this.size = size
    },
    selectURLtoFetch() {
      if (this.size === 'маленький') {
        this.url = `${process.env.VUE_APP_URL_32ROWS}`
      } else if (this.size === 'большой') {
        this.url = `${process.env.VUE_APP_URL_1000ROWS}`
      }
    },
    async fetchData() {
      this.loading = true

      try {
        const { data } = await this.$http.request({
          url: this.url
        })
        console.log('data')
        console.log(data)

        this.people = data
      } catch (error) {
        console.log(error)
      }
      this.loading = false
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.loading-row {
  /* text-align: center; */
}

h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
