<template>
    <div>
        <!-- <b-alert show variant="success"> -->
            <!-- {{ formattedData }} -->
            <b-table small stacked outlined bordered table-variant="success" responsive :items="formattedData"></b-table>
        <!-- </b-alert> -->
    </div>
</template>

<script>
export default {
  name: "SearchResult",
  props: {
    data: {required: true, type: Object}
  },
  data() {
    return {

    }
  },
  computed: {
      formattedData () {
          return [this.data]
      }
  },
  deactivated: function () {
    console.log('deactivated component:', 'searchResult')
  },
  activated: function () {
    console.log('activated component:', 'searchResult');
  },
  methods: {

  }
}
</script>

<style scoped>

</style>