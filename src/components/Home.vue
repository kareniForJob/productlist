<template>
  <div class="home">
    <block-view :products="products"></block-view>
  </div>
</template>

<script>
import BlockView from './blockView/BlockView'
import Jakson from '../products.json'

export default {
  name: 'Home',
  data () {
    return {
      products: Jakson
    }
  },
  components: {
    'block-view': BlockView
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.home {
  max-width: 1060px;
  min-width: 670px;
  margin: auto;
}
</style>
