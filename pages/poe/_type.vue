<template>
  <div>
    <nuxt-child />
    <StoreThingy :page-title="type" @setSelectedItem="searchTradeItem ($event)" />
  </div>
</template>

<script>
import StoreThingy from '../../components/StoreThingy'

export default {
  name: 'Type',

  transition: 't2',

  components: {
    StoreThingy
  },

  middleware: 'poeItems',

  data () {
    return {
      type: this.$route.params.type,
      search: false
    }
  },

  // idealmente isto seria com validate ou assim. se fosse enviado por parametros ou assim e quisessemos rejeitar
  beforeCreate () {
    if (this.$store.state.selection === null || this.$store.state.items.length === 0) {
      // this.$router.push({
      //   path: '/poe'
      // })
      return this.$nuxt.error({ statusCode: 404, message: 'Invalid URL' })
    }
  },

  methods: {

    getItemStringParams (item) {
      let ret = ''
      if (item.isRelic) {
        ret += '&rarity=uniquefoil'
      }
      // yes, the api is retarded enough to give map tiers to things that aren't maps
      if (item.mapTier > 0 && !item.isEssence) {
        ret += `&maptier=${item.mapTier}`
      }
      if (item.variant !== null) {
        ret += `&variant=${item.variant}`
      }
      if (item.links > 0) {
        ret += `&socketlinks=${item.links}`
      }
      return ret
    },

    searchTradeItem (obj) {
      if (obj.baseType === undefined) {
        return
      }
      this.search = true
      let queryString = `/poe/${this.type}/${obj.name}`
      if (obj.baseType !== null && !obj.name.includes(obj.baseType)) {
        queryString += `&${obj.baseType}`
      }
      // more black magics
      if (this.type === 'prophecy') {
        queryString += `&Prophecy`
      }
      this.$router.push({
        path: queryString + this.getItemStringParams(obj.usefulInfo)
      })
    }

  }
}
</script>

<style slot-scope="component">
  img:hover, img.nuxt-link-active {
    cursor: pointer;
  }
  img:active {
    cursor: wait;
  }
</style>
