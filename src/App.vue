<template>
  <header>
    <div class="site-name">{{ sitename }}</div>
    <div class="cart">
      <button
        class="cart__button"
        type="button"
        @:click="isProductPage = !isProductPage"
      >
        <span class="material-icons md-24">shopping_cart</span>
        <div>{{ cartItemCount }} Checkout</div>
      </button>
    </div>
  </header>
  <main v-if="isProductPage">
    <h1>Menu</h1>
    {{ product }}
    <div class="product-page">
      <div class="product__item" v-for="product in products" :key="product.id">
        <div>
          <img :src="product.image" width="867" height="1300" />
        </div>
        <div class="product-page__description">
          <h1 :class="{ oos: product.outOfStock }">{{ product.title }}</h1>
          <p v-html="product.description"></p>
          <p>{{ formatPrice(product.price) }}</p>
          <button
            class="button"
            :disabled="product.outOfStock"
            v-on:click="addToCart(product)"
          >
            {{ product.outOfStock ? "Out of Stock" : "Add to Cart" }}
          </button>
          <p v-if="product.stock <= 1" class="low-in-stock">Low in Stock</p>
        </div>
      </div>
    </div>
  </main>

  <main class="" v-else>
    <div v-if="cartItemCount > 0">
      <h1>Cart Page</h1>
      <ul class="checkout-page">
        <!--VARIABLES NEED TO HAVE DIFFERENT NAME! OTHERWISE IT WILL
         SHOW ALL ITEMS INSTEAD OF SELECTED ONES.-->
        <div
          v-for="cartItem in cartWithCounts"
          :key="cartItem.id + '-' + cartItem.count"
          class="flex-reverse-colum"
        >
          <button class="button" @click="removeFromCart(cartItem)">
            Remove
          </button>
          <li>{{ formatPrice(cartItem.price) }} EACH</li>
          <li>
            {{ cartItem.title }}
            <span v-if="cartItem.count > 1"> ({{ cartItem.count }})</span>
          </li>
          <img :src="cartItem.image" width="50" height="50" />
        </div>
      </ul>
      <p class="total-price">Total: {{ formatPrice(totalPrice) }}</p>
    </div>
    <div v-else>
      <h1 class="cart-empty">Your cart is empty</h1>
    </div>
  </main>
</template>

<script>
import axios from "axios";
export default {
  data: function () {
    return {
      sitename: "Nestor Bowl",
      products: [],
      cart: [],
      isProductPage: true,
    };
  },
  computed: {
    cartItemCount() {
      return this.cart.length;
    },
    cartWithCounts() {
      const cartWithCounts = this.cart.reduce((acc, item) => {
        const existingItem = acc.find((i) => i.id === item.id);
        if (existingItem) {
          existingItem.count++;
        } else {
          acc.push({ ...item, count: 1 });
        }
        return acc;
      }, []);
      return cartWithCounts;
    },

    totalPrice: function () {
      return this.cart.reduce((total, item) => total + item.price, 0);
    },
  },
  created: function () {
    const app = this;
    axios.get("data.json").then(function (response) {
      console.log(response.data); // Add this line
      app.products = response.data.products;
    });
  },

  methods: {
    formatPrice: function (price) {
      const options = { style: "currency", currency: "USD" };
      const dollars = price / 100;
      return dollars.toLocaleString("en-US", options);
    },
    addToCart(product) {
      if (product.stock <= 0) {
        product.outOfStock = true;
        return;
      }
      this.cart.push(product);
      product.stock--;
    },
    removeFromCart(product) {
      const index = this.cart.findIndex((item) => item.id === product.id);
      if (index !== -1) {
        this.cart.splice(index, 1);
      }
    },
  },
};
</script>

<style>
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial,
    sans-serif, "Apple Color Emoji", "Segoe UI Emoji";
  margin: 0;
}

li {
  display: flex;
}

button:hover {
  opacity: 0.7;
  cursor: pointer;
}

h3,
h1 {
  text-align: center;
}

.total-price {
  background: black;
  color: white;
  padding: 1%;
  margin: 2% auto;
  text-align: center;
  width: 16%;
}

.checkout-page {
  display: flex;
  margin: 0 auto;
  justify-content: center;
  gap: 5%;
}

.cart-empty {
  text-align: center;
}

img {
  width: 300px;
  height: auto;
}

header {
  background-color: black;
  color: #fff;
  padding: 20px;
  display: flex;
  border-bottom: 1px solid #ccc;
  font-weight: bold;
  font-size: 30px;
  justify-content: space-around;
  align-items: center;
}

.product__item {
  display: flex;
}
.flex-reverse-colum {
  display: flex;
  flex-direction: column-reverse;
  align-items: center;
  gap: 2%;
  grid-gap: 15px;
}

.product-page__description {
  padding-left: 20px;
  justify-content: center;
  align-items: center;
  display: grid;
  text-align: center;
}

.product-page {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(3, 1fr);
  grid-gap: 10px;
}

.oos {
  text-decoration: line-through;
  color: #ccc;
}

.cart__button {
  display: flex;
  align-items: center;
}

.button {
  font-size: 20px;
  background-color: #f14129;
  color: white;
  padding: 12px 30px;
  border: 1px solid #407c16;
  border-radius: 9px;
}

.button:disabled {
  background-color: #ccc;
  border-color: #ccc;
  color: #ddd;
}
</style>