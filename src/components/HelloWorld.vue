<template>
  <div class="hello">
    <p>Hello World!</p>
    <p>Hello World Again! New message. Hot Reload works well</p>

    <form>
      <div class="control-container">
        <label></label>
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
      <div v-show="this.loaded" class="search-container">
        <!-- <label></label> -->
        <input type="text" v-model="input" />
        <button
          type="button"
          class="btn btn-outline-warning"
          v-on:click="search()"
        >
          Искать
        </button>
      </div>

      <div v-if="this.error" class="error-container">
        <span> {{ error }}</span>
      </div>
    </form>

    <div v-show="this.loaded" class="add-container">
      <form>
        <button
          v-show="!visible"
          type="button"
          v-on:click="visible = true"
          class="btn btn-outline-danger"
        >
          Добавить
        </button>
        <ul v-show="visible">
          <li>
            <label for="">id</label>
            <input type="text" v-model="row.id" />
          </li>
          <li>
            <label for="">firstName</label>
            <input type="text" v-model="row.firstName" />
          </li>
          <li>
            <label for="">lastName</label>
            <input type="text" v-model="row.lastName" />
          </li>
          <li>
            <label for="">email</label>
            <input type="text" v-model="row.email" />
          </li>
          <li>
            <label for="">phone</label>
            <input type="text" v-model="row.phone" />
          </li>
        </ul>
        <button
          v-show="isAddFormReady"
          type="button"
          v-on:click="addRow()"
          class="btn btn-outline-warning"
        >
          Добавить в таблицу
        </button>
      </form>
    </div>

    <div v-if="loading" class="loading-container">
      <p class="table-success">Загрузка...</p>
    </div>

    <!-- <div class="loading-container" v-if="loading">LOADING ...</div> -->
    <div v-if="this.loaded" class="result-container">
      <!-- <span>pagination table</span> -->
      <pagination
        v-bind:currentPage.sync="currentPage"
        v-bind:totalPages="totalPages"
      ></pagination>

      <table class="table table-hover">
        <thead>
          <tr>
            <th
              scope="col"
              v-for="(header, index) in headers"
              v-bind:key="header.id"
              v-on:click="changeFilterProperty(header.property, index)"
            >
              {{ header.symbol }} {{ header.title }}
            </th>
          </tr>
        </thead>
        <tbody>
          <!-- <tr v-if="loading">
            <td class="table-success" colspan="7">загрузка...</td>
          </tr> -->
          <table-element
            v-for="(person, index) in selectedPeople"
            v-bind:key="index"
            v-bind:person="person"
            v-on:click.native="selectPerson(index)"
          ></table-element>
        </tbody>
      </table>

      <div class="custom-element-container" v-if="isIndexSelected">
        <pre> {{ selectedPeople[index] }} </pre>
      </div>
    </div>
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
      headers: [
        {
          id: 1,
          title: '№',
          property: 'id'
        },
        {
          id: 2,
          title: 'Имя',
          property: 'firstName'
        },
        {
          id: 3,
          title: 'Фамилия',
          property: 'lastName'
        },
        {
          id: 4,
          title: 'Электропочта',
          property: 'email'
        },
        {
          id: 5,
          title: 'Телефон',
          property: 'phone'
        },
        {
          id: 6,
          title: 'Адрес',
          property: 'address'
        },
        {
          id: 7,
          title: 'Описание',
          property: 'description'
        }
      ],
      //settings
      size: '',
      url: '',
      loading: false,
      loaded: false,
      error: '',

      people: [],
      // items: [],

      // select person
      index: null,

      // filter
      property: '',
      order: '',
      query: '',

      //search
      input: '',

      // pagination
      totalPages: null,
      currentPage: null,
      perPage: null,

      // add
      row: {
        id: '',
        firstName: '',
        lastName: '',
        email: '',
        phone: '',
        description: '',
        address: {
          city: '',
          state: '',
          streetAddress: '',
          zip: ''
        }
      },
      visible: false
      // status: false
    }
  },
  computed: {
    isAddFormReady() {
      return this.row.id &&
        this.row.firstName &&
        this.row.lastName &&
        this.row.email &&
        this.row.phone
        ? true
        : false
    },
    isIndexSelected() {
      return this.index !== null
    },
    selectedPeople() {
      let filteredPeople = [...this.selectedItems]

      if (this.property) {
        // console.log('property')

        let ascending, descending, expression

        if (this.property === 'id') {
          ascending = (a, b) => a.id - b.id
          descending = (a, b) => b.id - a.id
        } else if (this.property === 'address') {
          ascending = (a, b) =>
            Object.values(a.address)
              .toString()
              .localeCompare(Object.values(b.address).toString())
          descending = (a, b) =>
            Object.values(b.address)
              .toString()
              .localeCompare(Object.values(a.address).toString())
        } else {
          ascending = (a, b) =>
            a[`${this.property}`].localeCompare(b[`${this.property}`])
          descending = (a, b) =>
            b[`${this.property}`].localeCompare(a[`${this.property}`])
        }

        if (this.order === 'возростание') {
          expression = ascending
        } else if (this.order === 'убывание') {
          expression = descending
        }

        filteredPeople.sort(expression)
      }

      let selectedPeople

      // debugger

      if (this.query) {
        selectedPeople = filteredPeople.filter(person => {
          return Object.values(person)
            .toString()
            .includes(this.query)
        })
      } else {
        this.clearFilterProperty()
        selectedPeople = [...filteredPeople]
      }
      console.log('selectedPeople')
      console.log(selectedPeople)

      return selectedPeople
    },
    selectedItems() {
      this.clearFilterProperty()

      let temp = []

      let start = (this.currentPage - 1) * this.perPage
      let end = start + this.perPage - 1

      for (let i = start; i <= end; i++) {
        temp.push(this.people[i])
      }

      // this.$set(this, 'items', temp)
      return temp
    }
  },
  watch: {
    size: async function() {
      this.clearPaginationSetup()
      //setup url and download data
      this.selectURLtoFetch()
      await this.fetchData()

      if (this.loaded) {
        this.paginationSetup()
      }
      // check length of array and add pagination
    }
    // currentPage: function() {}
  },
  methods: {
    addAdditionForm() {
      this.visible = true
    },
    addRow() {
      let obj = {}
      Object.assign(obj, this.row)
      this.people.unshift(obj)

      this.clearFilterProperty()
      // this.clearPaginationSetup()
      // this.paginationSetup()
      this.visible = false
      this.clearPersonData()
    },
    clearPersonData() {
      Object.keys(this.row).map(el => {
        this.$set(this.row, el, '')
      })
    },
    search() {
      this.query = this.input
      this.input = ''
    },
    paginationSetup() {
      if (this.size === 'маленький') {
        this.perPage = 10
      } else if (this.size === 'большой') {
        this.perPage = 50
      }

      this.totalPages = Math.floor(this.people.length / this.perPage)
      this.currentPage = 1
    },
    clearTableSymbols() {
      this.headers.forEach(el => {
        this.$set(el, 'symbol', '')
      })
    },
    clearFilterProperty() {
      this.query = ''
      this.property = ''

      this.clearTableSymbols()
    },
    clearPaginationSetup() {
      this.currentPage = null
      this.totalPages = null
      this.perPage = null
    },
    changeFilterProperty(property, index) {
      if (this.loaded) {
        this.property = property
        this.clearTableSymbols()

        if (this.order === 'возростание') {
          this.order = 'убывание'
          this.$set(this.headers[index], 'symbol', '↑')
        } else if (this.order === 'убывание' || this.order === '') {
          this.order = 'возростание'
          this.$set(this.headers[index], 'symbol', '↓')
        }
      }
    },
    selectPerson(index) {
      this.index = index
    },
    selectVolumeData(size) {
      if (!this.loading) {
        // default values for filter
        this.clearFilterProperty()
        // and setting size of future loaded data
        this.size = size
      }
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
        this.loaded = true
      } catch (error) {
        console.log(error)
        this.error = error
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

table th {
  cursor: pointer;
}
</style>
