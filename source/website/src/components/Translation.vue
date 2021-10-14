<template>
  <div>
    <div v-if="noTranslation === true">
      No translation found for this asset
    </div>
    <div
      v-if="isBusy"
      class="wrapper"
    >
      <b-spinner
        variant="secondary"
        label="Loading..."
      />
      <p class="text-muted">
        (Loading...)
      </p>
    </div>
    <div
      v-else-if="noTranslation === false"
      class="wrapper"
    >
      <label>Source Language:</label> {{ source_language }}<br>
      <label>Target Language:</label> {{ target_language }}<br>
      {{ translated_text }}
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
  name: "Translation",
  data() {
    return {
      translated_text: "",
      source_language: "",
      target_language: "",
      isBusy: false,
      operator: "translation",
      noTranslation: false,
      elasticsearch_data: []
    }
  },
  deactivated: function () {
    console.log('deactivated component:', this.operator)
  },
  activated: function () {
    console.log('activated component:', this.operator);
    this.fetchAssetData();
  },
  beforeDestroy: function () {
      this.translated_text = "";
      this.source_language = "";
      this.target_language = "";
    },
  methods: {
    async fetchAssetData () {
      let query = 'AssetId:'+this.$route.params.asset_id+ ' _index:mietranslation';
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
          this.noTranslation = true
        }
        else {
          this.noTranslation = false;
          for (let i = 0, len = data.length; i < len; i++) {
            this.elasticsearch_data.push(data[i]._source)
            this.translated_text = data[i]._source.TranslatedText;
            this.source_language = data[i]._source.SourceLanguageCode;
            this.target_language = data[i]._source.TargetLanguageCode
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
