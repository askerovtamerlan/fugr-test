<template>
  <div class="pagination-container">
    <!-- <button>first</button> -->
    <!-- <button>next</button> -->
    <!-- <button v-on:click="currentPage = ">{{ currentPage }}</button> -->
    <button
      v-for="item in totalPages"
      v-bind:key="item"
      v-on:click="changePage(item)"
    >
      {{ item }}
    </button>
    <!-- <button>previous</button> -->
    <!-- <button>last</button> -->
  </div>
</template>

<script>
export default {
  name: 'Pagination',
  props: {
    totalPages: {
      type: Number
    },
    currentPage: {
      type: Number
    }
  },
  data() {
    return {
      page: null
    }
  },
  watch: {
    page: function() {
      this.$emit('update:currentPage', this.page)
    }
  },
  mounted() {
    this.page = this.currentPage
  },
  methods: {
    changePage(item) {
      this.page = item
    }
  }
}
</script>

<style scoped>
.pagination-container {
  display: flex;
  justify-content: space-between;
}

button {
}
</style>
