<template>
  <button-full @click.native="addToCartWrapper(product)" :disabled="isProductDisabled" data-testid="addToCart" class="w-full" :class="{'bg-primary': isProductDisabled || added}">
    <div class="flex items-center justify-center">
      <span>{{ failed ? $t('Error while adding') : added ? $t('Added to cart') : $t('Add to cart') }}</span>
      <div v-show="isAddingToCart" class="loader ml-1"/>
      <svg v-show="added" viewBox="0 0 17.333 9.333" class="vt-icon--sm ml-1">
        <use xlink:href="#success"/>
      </svg>
      <svg v-show="failed" viewBox="0 0 17.313 17.311" class="vt-icon--sm ml-1 fill-white">
        <use xlink:href="#error"/>
      </svg>
    </div>
  </button-full>
</template>

<script>
import { formatProductMessages } from '@vue-storefront/core/filters/product-messages'
import focusClean from 'theme/components/theme/directives/focusClean'
import ButtonFull from 'theme/components/theme/ButtonFull.vue'
import { AddToCart } from '@vue-storefront/core/modules/cart/components/AddToCart'
import { setTimeout } from 'timers'

export default {
  mixins: [AddToCart],
  directives: { focusClean },
  components: { ButtonFull },
  data () {
    return {
      added: false,
      failed: false
    }
  },
  computed: {
    isProductDisabled () {
      return this.disabled || formatProductMessages(this.product.errors) !== '' || this.isAddingToCart
    }
  },
  methods: {
    onAfterRemovedVariant () {
      this.$forceUpdate()
    },
    notifyUser (notificationData) {
      this.$store.dispatch('notification/spawnNotification', notificationData, { root: true })

      if (notificationData.type === 'success') {
        this.added = true
      } else {
        this.failed = true
      }

      setTimeout(() => {
        this.added = false
        this.failed = false
      }, 2000)
    },
    addToCartWrapper (product) {
      this.added = false
      this.failed = false
      this.addToCart(product)
    }
  },
  beforeMount () {
    this.$bus.$on('product-after-removevariant', this.onAfterRemovedVariant)
  },
  beforeDestroy () {
    this.$bus.$off('product-after-removevariant')
  }
}
</script>

<style lang="scss" scoped>
.loader {
  display: inline-block;
  border: 3px solid #fff;
  border-top: 3px solid theme('colors.primary');
  border-radius: 50%;
  width: 16px;
  height: 16px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
