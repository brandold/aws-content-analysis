<template>
  <div>
    <div v-if="noTranscript === true">
      No transcript found for this asset
    </div>
    <div v-if="isBusy">
      <b-spinner
        variant="secondary"
        label="Loading..."
      />
      <p class="text-muted">
        (Loading...)
      </p>
    </div>
    <div v-else>
      <div v-if="isProfane">
        <span style="color:red">WARNING: Transcript contains potentially offensive words.</span>
        <br>
        <br>
      </div>
      {{ transcript }}
    </div>
    <b-button
      type="button"
      @click="saveFile()"
    >
      Download Data
    </b-button>
  </div>
</template>

<script>
export default {
  name: "Transcript",
  data() {
    return {
      transcript: "",
      isBusy: false,
      operator: "transcript",
      noTranscript: false,
      elasticsearch_data: []
    }
  },
  computed: {
    isProfane() {
      const Filter = require('bad-words');
      const profanityFilter = new Filter({ placeHolder: '_' });
      return profanityFilter.isProfane(this.transcript);
    },
  },
  deactivated: function () {
    this.transcript = ""
    console.log('deactivated component:', this.operator)
  },
  activated: function () {
    console.log('activated component:', this.operator);
    this.fetchAssetData();
  },
  beforeDestroy: function () {
      this.transcript = ''
  },
  methods: {
    async fetchAssetData () {
      let query = 'AssetId:'+this.$route.params.asset_id+ ' _index:mievideotranscript';
      let apiName = 'contentAnalysisElasticsearch';
      let path = '/_search';
      let apiParams = {
        headers: {'Content-Type': 'application/json'},
        queryStringParameters: {'q': query, 'default_operator': 'AND', 'size': 10000}
      };
      let response = await this.$Amplify.API.get(apiName, path, apiParams);
      if (!response) {
        this.showElasticSearchAlert = true
      }
      else {
        let result = await response;
        let data = result.hits.hits;
        if (data.length === 0) {
          this.noTranscript = true
        }
        else {
          for (let i = 0, len = data.length; i < len; i++) {
            if ('transcript' in data[i]._source) {
              this.elasticsearch_data.push(data[i]._source)
              this.transcript = this.transcript.concat(data[i]._source.transcript + " ")
              this.noTranscript = false;
            }
          }
        }
        this.isBusy = false
      }
    },
    saveFile() {
      const elasticsearch_data = JSON.stringify(this.elasticsearch_data);
      const blob = new Blob([elasticsearch_data], {type: 'text/plain'});
      const e = document.createEvent('MouseEvents'),
        a = document.createElement('a');
      a.download = "data.json";
      a.href = window.URL.createObjectURL(blob);
      a.dataset.downloadurl = ['text/json', a.download, a.href].join(':');
      e.initEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
      a.dispatchEvent(e);
    }
  }
}
</script>
