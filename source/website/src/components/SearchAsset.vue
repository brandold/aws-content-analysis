<template>
    <div>
          <input
            v-model="user_defined_query"
            type="text"
            placeholder="Enter a keyword"
            @keyup.enter="searchCollection"
          >

          <ul>
            <li v-for="item in searchResults" :key="item.id">
              <search-result :data="item"></search-result>
            </li>
          </ul>
          <!-- <b-table striped hover fixed :items="searchResults"></b-table> -->
    </div>
</template>

<script>
import SearchResult from '@/components/SearchResult.vue'
export default {
  name: "Search",
  components: {
    SearchResult
  },
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
      let apiParams = {
        headers: {'Content-Type': 'application/json'},
        body: {
          "query": {
            "bool": {
              "must": [
                {
                  "query_string": {
                    "query" : query
                  }
                }
              ],
              "filter": [
                {
                    "match": {
                    "AssetId": this.$route.params.asset_id
                    }
                }
              ]
            }
          }
        }
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


<style scoped>
li {list-style-type: none;}
</style>