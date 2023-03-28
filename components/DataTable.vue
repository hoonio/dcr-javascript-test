<template>
  <v-table>
    <thead>
      <tr>
        <th class="text-left">
          Name
        </th>
        <th class="text-left">
          {{label.charAt(0).toUpperCase() + label.slice(1)}}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="item in dataset"
        :key="item.alpha3Code"
      >
        <td>{{ item.name }}</td>
        <td>{{ item.display }}</td>
      </tr>
    </tbody>
  </v-table>
</template>

<script>
  export default {
  props: {
    dataset: Array,
    label: String
  },
    data () {
      return {
      }
    },
  }
</script>
