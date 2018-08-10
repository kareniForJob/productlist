<template>
<div :id="product.productId" class="product">
  <div class="product-container product__product-container">
    <div class="product-photo product-container__product-photo"> 
      <a class="product-photo__url" href="#">
        <img class="product-photo__img" :src="product.primaryImageUrl">
      </a>
    </div>
    <div class="product-description product-container__product-description"> 
      <div class="product-description__top-information">
        <div class="product-code__container">
          <span class="product-code">Код: {{ getCodeWithoutZeros(product.code) }}</span>
        </div>
        <div class="product-status__container">
          <a class="product-status" href="#">Наличие</a>
        </div>
      </div>
      <div class="product-description__title">
        <div class="product-link__container">
          <a class="product-link" href="#">{{ product.title }}</a>
        </div>
      </div>
      <div class="product-description__main-information-switch">
        <div class="main-information-switch__container">
          <p @click="showDescription()" class="main-information-switch"
          >{{isShowDescription?'Скрыть описание':'Показать описание'}}</p>
        </div>
      </div>
      <div class="product-description__main-information">
        <transition
          :css="false"
          @before-enter="descriptionBeforeEnter"
          @enter="descriptionEnter"
          @after-enter="descriptionAfterEnter"
          @leave="descriptionLeave">
            <div class="main-information__container" v-if="isShowDescription">
              <p class="main-information"
              > {{ getBasicDescription(product.description) }} </p>
            </div>
        </transition>
      </div>
      <div class="product-description__additional-information">
        <div class="additional-information__container">
          <p class="additional-information__title">Могут понадобиться:</p>
          <span class="additional-information__tag" 
            v-for="(assocProduct, index) in assocProducts" 
            :key="index"
            ><a class="additional-information__link" href="#">{{assocProduct}}
            </a>{{(index === assocProducts.length - 1 ? '' : ', ')}}
          </span>
        </div>
      </div>
    </div>
    <div class="purchase-information product-container__purchase-information">
      <div class="purchase-information__top">
        <div class="product-price purchase-information__product-price"> 
          <span class="club-card__text">По карте<br>клуба</span>
          <div class="product-price__container">
            <p class="club-card">
              <span class="club-card__goldPrice"
              >{{animatedPriceGold}} р.</span>
            </p>
            <p class="price-default">
              <span class="club-card__retailPrice"
              >{{animatedPriceDefault}} р.</span>
            </p>
          </div>
        </div>
        <div class="bonus-title__container" v-if="product.bonusAmount>0">
          <p class="bonus-title">Можно купить за {{product.bonusAmount}} балла</p>
        </div>
        <div class="product-units__container">
          <div :class="'product-unit ' + (unit.isActive?'product-unit_active':'')"
            v-for="(unit,index) in units" :key="index"
            @click="setActive(index)">
            <p class="product-unit__title">За {{unit.title}}</p>
          </div>
        </div>
      </div>
      <div class="purchase-information__bottom">
        <div class="unit-info__container" v-if="units.length>1">
          <div class="unit-info">
            <div class="unit-info__img"></div>
            <div class="unit-info__text">
              <p>
                <span class="unit-info__title">Продается упаковками:</span>
                <span class="unit-info__calc">{{product.unitRatio}} {{product.unit}} = {{product.unitRatioAlt.toFixed(2)}} {{product.unitAlt}} </span>
              </p>
            </div>
          </div>
        </div>
        <div class="shopping-cart-form__container">
          <div class="shopping-cart-form">
            <div class="product-count__container">
              <div class="stepper">
                <input class="stepper__input" v-model="productCounter">
                <div class="stepper__arrows">
                  <span class="stepper__arrow stepper__arrow_up" @click="productCounter++"></span>
                  <span class="stepper__arrow stepper__arrow_down" @click="productCounter--"></span>
                </div>
              </div>
            </div>
            <span class="btn btn-shopping-cart shopping-cart-form__btn" @click="addToCart" 
                  data-url="/cart/" :data-product-id="product.productId">
              <svg class="icon icon-cart">
                <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#cart"></use>
              </svg>
              <span class="btn-shopping-cart__text">В корзину</span>
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
</template>

<script>
export default {
  name: 'ProductTemplate',
  data () {
    return {
      isShowDescription: false,
      assocProducts: [],
      units: [],
      productCounter: 1,
      priceGold: 0,
      tweenedPriceGold: 0,
      priceDefault: 0,
      tweenedPriceDefault: 0
    }
  },
  props: ['product'],
  watch: {
    productCounter: function (value) {
      this.productCounter = parseInt(value);
      if (isNaN(this.productCounter) || this.productCounter < 1) {
        this.productCounter = 1;
      }
      else if (this.productCounter > 1000) {
        this.productCounter = 1000;
      }
    },
    priceGold: function(newPriceGold) {
      TweenLite.to(this.$data, 0.3, { tweenedPriceGold: newPriceGold });
    },
    priceDefault: function(newPriceDefault) {
      TweenLite.to(this.$data, 0.3, { tweenedPriceDefault: newPriceDefault });
    }
  },
  computed: {
    animatedPriceGold: function() {
      return this.tweenedPriceGold.toFixed(0);
    },
    animatedPriceDefault: function() {
      return this.tweenedPriceDefault.toFixed(0);
    }
  },
  beforeMount: function () {
    this.setAssocProducts();
    this.setUnits();
    this.tweenedPriceGold = this.product.priceGold;
    this.tweenedPriceDefault = this.product.priceRetail;
  },
  methods: {
    setAssocProducts () {
      this.assocProducts = this.product.assocProducts.split(';')
      .filter(function (str) {
        return str.trim() !== '';
      });
    },
    setUnits () {
      let arr = [];
      arr[0] = {
        title: this.product.unit,
        isActive: true
      };
      if (arr[0].title !== this.product.unitAlt) {
        arr[1] = {
          title: this.product.unitAlt,
          isActive: false
        };
      }
      this.units = arr;
    },
    getCodeWithoutZeros (code) {
      return code.slice(code.search(/[^0]/i))
    },
    getBasicDescription (description) {
      return description.trim()
    },
    showDescription () {
      this.isShowDescription = !this.isShowDescription;
    },
    setActive (i) {
      if (this.units[i].isActive === false) {
        [this.units[i].isActive, this.units[(i+1)%2].isActive] = 
        [this.units[(i+1)%2].isActive, this.units[i].isActive];
        this.priceGold = i==0 ? this.product.priceGold:this.product.priceGoldAlt;
        this.priceDefault = i==0 ? this.product.priceRetail:this.product.priceRetailAlt;
      }
    },
    descriptionBeforeEnter: function (el) {
      el.style.opacity = 0
      el.style.height = 0
    },
    descriptionEnter: function (el, done) {
      let height = document.getElementById(this.product.productId)
                           .getElementsByClassName('main-information__container')[0]
                           .scrollHeight;
      Velocity(
        el,
        { opacity: 1, height: `${height} + px`, marginBottom: `6px` },
        { complete: done }
      )
    },
    descriptionAfterEnter: function (el,done) {
      Velocity(
        el,
        { height: `100%` },
        { complete: done }
      )
    },
    descriptionLeave: function (el, done) {
      Velocity(
        el,
        { opacity: 0, height: 0, marginBottom: 0 },
        { complete: done }
      )
    },
    addToCart () {
      this.productCounter = 1;
      alert('done');
    }
  }
}
</script>
<style scoped>
a {
  text-decoration: none;
  color: #000;
}
p {
  margin: 0;
}
.product {
  box-shadow: 0 0 1px rgba(0,0,0,0.2);
  border-radius: 2px;
}
.product-container {

}
.product__product-container {
  padding: 16px;
  display: flex;
}
.product-photo {
  width: 160px;
}
.product-container__product-photo{
  display: inline-block;
}
.product-photo__url {
  
}
.product-photo__img {  
  box-shadow: 0 0 1px rgba(0,0,0,0.2);
  border-radius: 2px;
  width: 160px;
}
.product-description {

  width: 560px;
}
.product-container__product-description {
  display: inline-block;
  padding-left: 16px;
  padding-right: 40px;
}
.product-description__top-information {


}
.product-code__container {
  display: inline-block;
}
.product-code {
  
}
.product-status__container {  
  display: inline-block;
  padding-left: 16px;
}
.product-status {
  color: #2a842a;
  border-bottom: 1px dotted #2a842a;
}
.product-status:hover {
  text-decoration: none;
  border-bottom: 1px solid #2a842a;
}
.product-description__title {
  font-size: 16px;
  line-height: 24px;
  font-weight: 700;
  max-width: 440px;
  padding-top: 16px;
}
.product-link__container {

}
.product-link {
  
}
.product-description__main-information-switch {
  display: inline-block;
  color: #f29c26;
  cursor: pointer;
  -moz-user-select: none;
  -khtml-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
.main-information-switch__container {

}
.main-information-switch {
  padding-top: 16px;
}
.product-description__main-information {
  text-align: justify;
  overflow: hidden;
  white-space: pre-line;
}
.main-information__container{
}
.main-information{
  padding-top: 16px;
}
.product-description__additional-information {

}
.additional-information__container {
  padding-top: 16px;
}
.additional-information__title {
  font-size: 13px;
  line-height: 16px;
  color: #545454;
  font-weight: 700;
  display: inline-block;
}
.additional-information__tag {
  font-size: 13px;
  line-height: 16px;
  color: #666;
  display: inline-flex;
}
.additional-information__link {
  color: #666;
}
.additional-information__link:hover {
  text-decoration: underline; 
}
.purchase-information {
  text-align: right;
}
.product-container__purchase-information {
  display: inline-block;

}
.purchase-information__top {

}
.product-price {
  
}
.purchase-information__product-price {
  display: inline-flex;
}
.product-price__container {
  padding-top: 6px;
  text-align: center;
}
.club-card {
}
.club-card__text {
  font-weight: 600;
  font-size: 14px;
  line-height: 17px;
  padding-right: 8px;
}
.club-card__goldPrice {
  font-size: 25px;
  font-weight: 700;
  color: #000;
}
.price-default {
  font-size: 25px;
  color: #a7a7a7;
}
.bonus-title__container {
  padding-top: 6px;
}
.bonus-title {
  font-size: 12px;
  line-height: 17px;
  color: #999;
}
.product-units__container {
  padding-top: 6px;
}
.product-unit {
  display: inline-flex;
  padding: 3px 5px;
  margin: 0 0 0 5px;
  cursor: pointer;
  font-size: 13px;
  line-height: 13px;
  color: #707070;
}
.product-unit_active {
  background: #000;
  color: #fff;
  cursor: default;
}
.product-unit__title {
  
}
.purchase-information__bottom {

}
.unit-info__container {
  position: relative;
  display: inline-block;
  padding-top: 16px;
}
.unit-info {
  width: 222px;
  text-align: left;
  border: 1px solid #ccc;
  padding: 5px 10px;
  border-radius: 2px;
}
.unit-info:before {
  content: '';
  position: absolute;
  bottom: -7px;
  border-top: 8px solid;
  margin-bottom: -1px;
  border-color: #ccc;
  color: #ccc;
  left: 13px;
  border-left: 8px solid transparent!important;
  border-right: 8px solid transparent!important;
}
.unit-info:after {
  content: '';
  position: absolute;
  bottom: -7px;
  border-top: 8px solid;
  border-color: #fff!important;
  left: 14px;
  border-left: 7px solid transparent!important;
  border-right: 7px solid transparent!important;
}
.unit-info__img {
  background: url(/static/images/icons/product/unit--i.png) 50% no-repeat;
  background-size: cover;
  width: 20px;
  height: 20px;
  display: inline-block;
  vertical-align: middle;
}
.unit-info__text {
  font-size: 14px;
  line-height: 1.4;
  padding-left: 8px;
  display: inline-block;
  vertical-align: middle;
}
.unit-info__title {
  display: block;
}
.unit-info__calc {
  display: block;
}
.shopping-cart-form__container {
  margin-top: 12px;
}
.shopping-cart-form {
  display: inline-flex;
  text-align: left;
  width: 244px;
}
.product-count__container {
  display: inline-block;
  height: 41px;
  width: 67px;
  border: 1px solid #ccc;
}
.stepper {
  display: inline-flex;
}
.stepper__input {
  text-align: center;
  border: none;
  color: #333;
  font-size: 13px;
  line-height: 1.2;
  overflow: hidden;
  width: 44px;
  z-index: 49;
  outline:none;
}
.stepper__arrows {
}
.stepper__arrow {
  display: block;
  background: url(/static/images/templates/jquery.fs.stepper-arrows.png) no-repeat #fff;
  cursor: pointer;
  height: 20px;
  width: 22px;
  z-index: 50;
}
.stepper__arrow:hover {
  background-color: #666;
}
.stepper__arrow_up {
  background-position: -29px 0;
  border-left: 1px solid #ccc;
}
.stepper__arrow_down {
  background-position: -29px -18px;
  border-left: 1px solid #ccc;
  border-top: 1px solid #ccc;
}
.btn-shopping-cart {
  display: inline-block;
  cursor: pointer;
  width: 148px;
  height: 43px;
  color: #fff;
  text-transform: uppercase;
  background: #f90;
  border-radius: 2px;
}
.btn-shopping-cart:hover {
  background: #555;
}
.shopping-cart-form__btn {
  margin-left: 8px;
  padding-left: 20px;
}
.icon-cart {
  width: 20px;
  height: 20px;
  vertical-align: middle;
  background-image: url(/static/images/templates/el.png);
  background-position: -125px -65px;
}
.btn-shopping-cart__text {
  font: 14px/40px Arial,Helvetica,sans-serif;
  padding-left: 10px;
}
</style>
