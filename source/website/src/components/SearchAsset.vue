<template>
    <div>
          <input
            v-model="user_defined_query"
            type="text"
            placeholder="Enter a keyword"
            @keyup.enter="searchCollection"
          >
          <b-table striped hover fixed :items="searchResults"></b-table>
    </div>
</template>

<script>
export default {
  name: "Search",
  data() {
    return {
      isBusy: false,
      operator: "",
      user_defined_query: "",
      searchResults: null
    }
  },
  deactivated: function () {
    console.log('deactivated component:', 'search')
  },
  activated: function () {
    console.log('activated component:', 'search');
  },
  methods: {
    async elasticsearchQuery (query) {
      this.isBusy = true
      let apiName = 'contentAnalysisElasticsearch';
      let path = '/_search';
      let completeQuery = 'AssetId:'+this.$route.params.asset_id + query;
      let apiParams = {
        headers: {'Content-Type': 'application/json'},
        queryStringParameters: {'q': completeQuery}
      };
      let es_data = [];
      let response = await this.$Amplify.API.post(apiName, path, apiParams);
      let data = response.hits.hits;
      for (let i = 0, len = data.length; i < len; i++) {
                console.log(data[i]._source)
                es_data.push(data[i]._source)
              }
      this.isBusy = false
      return es_data;
    },
    async searchCollection () {
      let results = await this.elasticsearchQuery(this.user_defined_query)
      this.searchResults = results
    }
  }
}
</script>
