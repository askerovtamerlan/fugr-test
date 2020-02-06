<template>
  <tr>
    <th scope="row">{{ person.id }}</th>
    <td>{{ person.firstName }}</td>
    <td>{{ person.lastName }}</td>
    <td>{{ person.email }}</td>
    <td>{{ person.phone }}</td>
    <td>{{ fullAddress }}</td>
    <td>{{ person.description }}</td>
  </tr>
</template>

<script>
export default {
  name: 'TableElement',
  props: {
    person: {
      type: Object,
      default() {
        return {
          id: '1',
          firstName: 'Stuart',
          lastName: 'Little',
          email: 'kinder@surprise.net',
          phone: '+1(344)855-33-25',
          description: 'blank',
          address: {
            streetAddress: '80 Ave P Apart B12',
            city: 'Brooklyn',
            state: 'NY',
            zip: '11705'
          }
        }
      }
    }
  },
  data() {
    return {}
  },
  computed: {
    fullAddress() {
      return Object.values(this.person.address).join(', ')
    }
  }
}
</script>

<style scoped></style>
