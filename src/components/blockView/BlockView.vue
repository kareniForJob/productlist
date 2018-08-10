<template>
<div class="product-list">
  <div class="product-list__product" 
    v-for="(product, index) in displayedProducts" 
    :key="index">
      <product :product="product"></product>
  </div>
</div>
</template>

<script>
import ProductTemplate from './ProductTemplate'

export default {
  name: 'BlockView',
  data () {
    return {
      numberOfDisplayedProducts: 6,
      displayedProductsCounter: 1,
      isAllProductsDownloaded: false
    }
  },
  props: ['products'],
  computed: {
    displayedProducts: function () {
      return this.products.slice(0,this.numberOfDisplayedProducts * this.displayedProductsCounter);
    }
  },
  beforeMount() {
    document.addEventListener('scroll', this.handleScroll);
  },
  beforeDestroy() {
    document.removeEventListener('scroll', this.handleScroll);
  },
  methods: {
    handleScroll: function(){
      if (this.isAllProductsDownloaded) return;
      let cHeight = document.documentElement.clientHeight;
      if(window.pageYOffset + cHeight >= document.getElementsByClassName("product-list")[0].clientHeight - cHeight) {
        this.displayedProductsCounter++;
      }
      this.isAllProductsDownloaded = this.numberOfDisplayedProducts * this.displayedProductsCounter >= this.products.length 
    }
  },
  components: {
    'product': ProductTemplate
  }
}
</script>
<style scoped>
.product-list__product {
  padding-top: 12px;
}
</style>
