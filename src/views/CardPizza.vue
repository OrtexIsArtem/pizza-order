<template>
  <div class="col s12 m10 l8 card-wrapper card-pizza">
    <button v-if="admin" @click="deletePizza" class="close">&#10006;</button>
    <div class="card deep-purple lighten-2 bill-card card-pizza-content">
      <div class="card-content white-text">
        <div class="card-header">
          <span class="card-title">{{ name }}</span>
        </div>
        <table>
          <thead>
            <tr>
              <th>Цена</th>
              <th>
                <span>{{ price }}</span
                >грн
              </th>
            </tr>
          </thead>
          <thead v-if="stockDate">
            <tr>
              <th>Акция до</th>
              <th>
                <span>{{ stockDate }}</span>
              </th>
            </tr>
          </thead>
        </table>
      </div>
      <div class="card-content card-content__bottom">
        <div class="card-img">
          <img :src="imgUrl" alt="" />
        </div>
        <div class="card-info">
          <p class="card-text">{{ info }}</p>
          <div class="buttons">
            <button class="recipe btn" @click="addOrderUser">Заказать</button>
            <button @click="getPizzaComposition" class="recipe btn">
              Состав
            </button>
          </div>
          <ul
            v-if="composition.length"
            class="composition deep-purple lighten-4"
          >
            <li v-for="(item, i) in composition" :key="i">- {{ item }}</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'CardPizza',
  props: {
    name: String,
    info: String,
    price: Number,
    imgUrl: String,
    stockDate: String
  },
  computed: {
    ...mapGetters(['getName'])
  },
  data: () => ({
    composition: [],
    userId: null,
    admin: false
  }),
  async mounted() {
    this.userId = await this.$store.dispatch('getUserId')
    if (this.userId) {
      await this.$store.dispatch('getUserName', this.userId)
    }
    this.admin = this.userId === 'TzpIYdPOGrVFRwHK0YjJ2t3eVBz1'
  },
  methods: {
    async deletePizza() {
      try {
        await this.$store.dispatch('deletePizza', {
          collection: this.stockDate === null ? 'pizza' : 'stock',
          name: this.name
        })
        console.log('удалено', this.name)
        this.$emit('remove-refresh')
      } catch (e) {
        throw new Error(e)
      }
    },
    async getPizzaComposition() {
      await this.$store
        .dispatch('getPizzaComposition', {
          collection: this.stockDate === null ? 'pizza' : 'stock',
          name: this.name
        })
        .then(res => {
          this.composition = res.data().composition
        })
    },
    async addOrderUser() {
      const options = {
        day: 'numeric',
        month: 'numeric',
        year: 'numeric',
        hour: 'numeric',
        minute: 'numeric',
        second: 'numeric'
      }
      const orderData = [
        {
          userName: this.getName,
          userId: this.userId,
          orderName: this.name,
          price: this.price,
          imgUrl: this.imgUrl,
          date: new Intl.DateTimeFormat('ru-RU', options).format(new Date())
        }
      ]
      await this.$store.dispatch('addOrder', orderData)
      await this.$router.push('/history?message=addOrder')
    }
  }
}
</script>

<style scoped lang="scss">
.card {
  &-pizza {
    box-shadow: 21px 17px 42px -15px rgba(0, 0, 0, 0.75);
    padding: 0;

    &-content {
      margin: 0;
    }
  }

  &-wrapper {
    position: relative;
  }

  &-img {
    max-width: 270px;
    min-width: 100px;
  }

  &-content__bottom {
    display: flex;
    justify-content: space-between;
  }
}

.composition {
  padding: 5px 10px;
}

.buttons {
  display: flex;
  justify-content: space-between;
}

.card .card-info {
  margin-left: 10px;
  flex-basis: 50%;
}

.card .card-text {
  margin-bottom: 10px;
}

.bill-card {
  height: auto;
}

img {
  width: 100%;
}

.close {
  position: absolute;
  right: 10px;
  top: 10px;
  z-index: 9;
  border: none;
  background: transparent;
  font-size: 24px;
  line-height: 1;
  color: white;
  cursor: pointer;
}

@media (max-width: 992px) {
  .card-content__bottom {
    flex-direction: column;
  }
}
</style>
