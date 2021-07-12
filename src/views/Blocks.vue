<template>
  <search></search>
  <loader :loading="isLoading"></loader>
  <div v-if="!isLoading">
    <template v-if="block">
      <block-summary :block="block"></block-summary>
    </template>
    <template v-if="blocks.length > 0">
      <h1 class="text-lg font-bold">Blocks</h1>
      <blocks :blocks="blocks"></blocks>
    </template>
  </div>
</template>

<script>
import axios from "axios";
import Blocks from "../components/LatestBlocks"
import BlockSummary from "../components/BlockSummary"
import Search from "../components/Search"
import Loader from "../components/Loader";

export default {
  name: "Blocks",
  components: {Loader, Search, BlockSummary, Blocks},
  data() {
    return {
      baseUrl: process.env.VUE_APP_EXPLORER_URL,
      blocks: [],
      block: null,
      isLoading: true
    }
  },
  watch: {
    $route(to, from) {
      if (to.params.hash && to.params.hash !== from.params.hash) {
        this.search(to.params.hash)
      }
    }
  },
  methods: {
    search(hash) {
      axios.post(`${this.baseUrl}`, JSON.stringify({
        method: 'search',
        params: [hash]
      }), {
        headers: {
          'Content-Type': 'application/json'
        }
      }).then(response => {
        this.block = response.data.result
        this.isLoading = false
      }).catch(error => {
        this.isLoading = false
        this.block = null
        console.log(error)
      })
    }
  },
  mounted() {
    if (this.$route.params.hash) {
      this.search(this.$route.params.hash)
    } else {
      // Get paginated blocks
    }
  }
}
</script>

<style scoped>

</style>