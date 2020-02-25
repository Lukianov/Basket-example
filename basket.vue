<script>
  import {mapState} from 'vuex'
  import Button from '../../components/Button/Button.vue'
  import Select from '../../components/Select/Select.vue'

  export default {
    name: 'basket',
    components: {
      'c-button': Button
    },
    data () {
      return {
        isFavorite: false,
        deleteFavorite: false
      }
    },
    computed: {
      total () {
        return Math.round(this.products.reduce((acc, i) => acc + Number(i.price), 0))
      },
      ...mapState({
        products: state => state.cart.products
      })
    },
    methods: {
      async deleteItem (index) {
        const option = this.products[index].option,
          cart_id = this.$cookies.get('_cs_cart_id')

        if (this.$store.getters.isAuthorized) {
          const {data: removed} = await this.$axios.delete('cart', {option})

          if (removed) {
            const {data: cartData} = await this.$axios.get('cart')

            this.$store.commit('cart/SET_CART_PRODUCTS', cartData.data.products)
          }
        }
        else {
          const {data: removed} = await this.$axios.delete('cart', {params: {cart_id, option}})

          if (removed) {
            const {data: cartData} = await this.$axios.get('cart', {params: {cart_id}})

            this.$store.commit('cart/SET_CART_PRODUCTS', cartData.data.products)
          }
        }
      },
      async addToFavorites (index) {
        const {colors, sku, favorite} = this.products[index]

        console.log(colors, sku, favorite)

        if (!this.$store.getters['user/isAuthorized']) {
          if (this.$cookies.get('_cs_fav_list')) {
            const fav_list = this.$cookies.get('_cs_fav_list')

            await this.$axios.$post('/favorites', {
              sku,
              colors,
              fav_list
            })
            this.isFavorite = true
          }
          else {
            const {data} = await this.$axios.$post('/favorites', {
                sku,
                colors
              }),
              {list} = data

            this.isFavorite = true

            this.$cookies.set('_cs_fav_list', list, {
              path: '/',
              expires: new Date(Date.now() * 2),
              sameSite: true
            })
          }
          if (favorite === true) {
            const fav_list = this.$cookies.get('_cs_fav_list')

            await this.$axios.delete('favorites', {params: {fav_list, sku, colors}})
            this.deleteFavorite = true
          }
        }
        else {
          const fav_list = null

          await this.$axios.$post('/favorites', {
            sku,
            colors,
            fav_list
          })
          this.isFavorite = true

          if (favorite === true) {
            await this.$axios.delete('favorites', {params: {fav_list, sku, colors}})
            this.deleteFavorite = true
          }
        }
      }
    }
  }
</script>

<template>
  <div class="g-wrapper page">
    <main>
      <div class="g-container">
        <div class="g-row">
          <div class="g-offset-md-1 g-col-md-11 t-cap">
            <h1 class="t-h1">
              Корзина
            </h1>
          </div>
        </div>
      </div>
      <div class="g-container">
        <div v-for="(product, index) in products"
             :key="index"
             class="p-basket-wrap">
          <div class="p-basket-wrap_item">
            <div class="g-row">
              <div class="g-col-xl-2 g-col-lg-3 g-col-md-3">
                <div class="p-basket-wrap_item__img">
                  <img :src="imgNoBack(product.image)"
                       :alt="product.brands">
                </div>
              </div>
              <div class="g-col-xl-5 g-col-lg-4 g-col-md-12">
                <div class="p-basket-wrap_item__name">
                  <span class="t-p-semibold">
                    {{ product.brand }}
                  </span>
                  <p class="t-p">
                    {{ product.name }}
                  </p>
                  <p class="t-p">
                    Артикул
                    <span class="t-p-semibold">
                      {{ product.sku }}
                    </span>
                  </p>
                </div>
              </div>
              <div class="g-col-13 g-col-lg-4 g-col-md-10 g-col-sm-13">
                <div class="p-basket-wrap_item__characteristics">
                  <div class="p-basket__size-wrp">
                    <span class="p-basket-wrap_item__characteristics-item__name t-p-semibold">
                      Размер:
                    </span>
                    <p class="t-p">
                      {{ product.size[0] }}
                    </p>
                  </div>
                </div>
              </div>
              <div class="g-col-lg-2 g-col-md-4">
                <div class="p-basket-wrap_item__price">
                  <span>
                    {{ mFmt(product.price, true) }}
                  </span>
                </div>
              </div>
              <div class="p-basket__block">
                <div class="p-basket__delete icon-delete"
                     @click="deleteItem(index)"/>
                <div class="p-basket__fav"
                     @click="addToFavorites(index)">
                  <span class="icon-ilem-liked">
                    <span class="path1"/>
                    <span class="path2"
                          :class="{'icon-ilem-liked-done': isFavorite || product.favorite === true, 'icon-ilem-liked-delete': deleteFavorite}"/>
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="p-basket__total mt-50">
          <div class="p-basket__total-el">
            <div class="p-basket__total-price">
              <h2 class="t-h2">
                Итого
              </h2>
              <div class="p-basket__total-price-title">
                {{ mFmt(total, true) }}
              </div>
            </div>
            <c-button v-if="products.length"
                      color="black"
                      to="/order">
              Оформить заказ
            </c-button>
          </div>
          <div class="p-basket__total-el t-cap">
            <div class="p-basket__total-title mb-30">
              City Seller сотрудничает с ритейлерами, надежность которых не подвергается сомнению. Партнеры маркетплейса — официальные региональные дистрибьюторы мировых брендов. Большинство из них представлено в рейтинге «100 лучших бутиков России».
            </div>
            <span class="g-link g-link_underline">
              Подробнее
            </span>
          </div>
        </div>
        <div class="p-basket__return">
          <h2 class="t-h2 p-basket__return-title">
            Бесплатный возврат покупок
          </h2>
          <div class="p-basket__return-content">
            <div class="p-basket__return-el p-basket__return-imgs">
              <svg xmlns="http://www.w3.org/2000/svg"
                   xmlns:xlink="http://www.w3.org/1999/xlink"
                   width="66px"
                   height="32px">
                <image x="0px"
                       y="0px"
                       width="66px"
                       height="32px"
                       xlink:href="data:img/png;base64,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"/>
              </svg>
              <img src="../../assets/img/basket/boxberry.png">
            </div>
            <div class="p-basket__return-el">
              <p class="t-p-semibold p-basket__return-info">
                Возврат через пункты выдачи BoxBerry или Почтой России
              </p>
              <p class="t-p">
                Более 8 500 отделений по всей стране. Вы можете сделать возврат на нашем сайте, с помощью
                формы для возврата. Доставка, обработка и возврат денег заимает примерно 21 день.
              </p>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>


<style lang="scss" src="./basket.scss" scoped/>