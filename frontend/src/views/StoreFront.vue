<template>
  <div>
    <!-- SEARCH BAR start -->
    <div class="pt-4 px-5">
      <b-row class="search-bar d-flex align-items-center mb-4">
        <b-col cols="8">
          <b-input-group>
            <b-form-input type="search" placeholder="Search"></b-form-input>
            <b-button>
              <span class="d-flex align-self-center material-icons md-24"
                >search</span
              >
            </b-button>
          </b-input-group>
        </b-col>
        <div>Filter:</div>

        <b-col>
          <b-form-select
            v-model="filtering"
            :options="
              categoryOption.map((e) => ({ value: e.id, text: e.name }))
            "
            >{{ filtering.name }}</b-form-select
          >
        </b-col>
      </b-row>
    </div>
    <!-- SEARCH BAR end -->
    <b-row id="body" class="mx-5">
      <div class="item-menu d-flex justify-content-start flex-wrap">
        <b-overlay
          v-for="item in items"
          :key="item.id"
          :show="item.quantity < 1"
          spinner-variant="primary"
          spinner-type="grow"
          spinner-small
          rounded="sm"
          style="max-width: 320px"
          class="m-1"
        >
          <template #overlay>
            <b-icon
              icon="stopwatch"
              variant="info"
              scale="2"
              shift-v="8"
              shift-h="8"
              class="position-absolute"
              style="top: 0; right: 0"
            ></b-icon>
          </template>
          <b-card
            :img-src="item.pic"
            :img-alt="item.pic"
            img-height="190"
            img-width="350"
            img-top
            tag="article"
            class="item-card"
            @click="addToCart(item)"
          >
            <b-card-title>{{ item.name }}</b-card-title>
            <!-- <div class="card--price">
                <span class="me-2">Price:</span>
                <span class="bold bigger">{{ item.price }}</span>
                <span class="mx-2">/</span>
                <span>{{ item.unit }}</span>
              </div>
              <div class="my-1">
                <span class="me-2">Category:</span>
                <b-button
                  v-for="cate in item.category"
                  :key="cate.id"
                  class="me-1"
                  variant="outline-info"
                  size="sm"
                  pill
                  >{{ cate.name }}</b-button
                >
              </div>
              <div>
                <span class="me-2">In-stock:</span>
                <span>{{ item.quantity }}</span>
              </div> -->
            <b-row>
              <b-col cols="4">
                <div>Price:</div>
                <div>In-stock:</div>
                <div>Category:</div>
              </b-col>
              <b-col cols="8">
                <div>THB {{ item.price }} / {{ item.unit }}</div>
                <div>{{ item.quantity }}</div>
                <div
                  v-for="cate in item.category"
                  :key="cate.id"
                  variant="outline-info"
                  size="sm"
                  pill
                >
                  {{ cate.name }}
                </div>
              </b-col>
            </b-row>
          </b-card>
        </b-overlay>
      </div>
      <b-col cols="4" class="item-cart">
        <b-list-group class="item-list mb-3">
          <b-list-group-item
            v-for="car in cart"
            :key="car.id"
            class="d-flex align-items-center justify-content-between"
          >
            <span class="d-flex align-items-center">
              <b-button
                variant="outline-light"
                style="outline: none !important"
                @click="removeFromCart(car.id)"
                ><b-icon
                  variant="danger"
                  icon="trash-fill"
                  font-scale="1"
                ></b-icon
              ></b-button>

              <b-form-spinbutton
                id="demo-sb"
                v-model="car.amount"
                :min="1"
                :max="car.quantity"
                inline
              >
              </b-form-spinbutton>
            </span>
            <span class="mx-2 text-truncate">{{ car.name }}</span>
            <span>{{ car.price }}</span>
          </b-list-group-item>
        </b-list-group>
        <b-card class="p-3">
          <b-row>
            <b-col cols="33">
              <div class="mt-1 mb-3">Discount:</div>
              <div class="">Note:</div>
            </b-col>
            <b-col cols="">
              <b-form-input
                class="mb-1"
                placeholder="add discount of this order"
                type="number"
                v-model="discount"
              ></b-form-input>
              <b-form-textarea
                v-model="noteOrder"
                placeholder="Add item desccription ..."
              ></b-form-textarea>
            </b-col>
          </b-row>
        </b-card>
        <b-button
          class="buttons--submit mt-3 d-flex align-items-center justify-content-between"
          variant="success"
          @click="calculateModelOrder()"
        >
          <span>Total</span>
          <span class="bold bigger">{{
            cart.reduce((previousValue, currentValue) => {
              return currentValue.price * currentValue.amount + previousValue;
            }, 0) - this.discount
          }}</span>
          <span>SUBMIT ORDER</span>
        </b-button>
        <div class="error-submit error mt-1" v-if="errorSubmit">
          Please select an item before submitting an order.
        </div>
      </b-col>
    </b-row>
    <b-modal title="Submit order" v-model="confirmOrderModal" @ok="createOrder"
      >Are you sure you want to submit this order?</b-modal
    >
  </div>
</template>

<script>
import api from "../apis";

export default {
  data() {
    return {
      confirmOrderModal: false,
      items: [],
      amount: 50,
      cart: [],
      paidStatus: false,
      checked: false,
      discount: 0,
      noteOrder: "",
      filtering: {},
      categoryOption: [],
      errorSubmit: false,
      count: 0,
    };
  },
  methods: {
    async initPage() {
      await api
        .getReadyItems()
        .then((result) => {
          console.log(result);
          this.items = [...result.data];
        })
        .catch((err) => {
          console.log(err);
        });
      await api
        .getCategory()
        .then((result) => {
          this.categoryOption = [...result.data];
        })
        .catch((err) => {
          console.log(err);
        });
    },
    addToCart(target) {
      const initCartBody = {
        amount: 1,
        quantity: target.quantity,
        name: target.name,
        id: target.id,
        price: target.price,
      };
      const index = this.cart.findIndex((e) => e.id === target.id);
      console.log(index);
      if (index >= 0) {
        this.cart[index].amount < this.cart[index].quantity
          ? (this.cart[index].amount += 1)
          : this.cart[index].amount;
      } else {
        target.quantity > 0 ? this.cart.push(initCartBody) : this.cart;
      }
      console.log(this.cart);
    },
    removeFromCart(target) {
      this.cart = this.cart.filter((value) => value.id !== target);
    },
    reload() {
      this.$router.go(this.$router.currentRoute);
    },

    calculateModelOrder() {
      this.confirmOrderModal = this.cart.length > 0 ? true : false;
      this.count = 1;
      if (this.confirmOrderModal == false && this.count != 0) {
        this.errorSubmit = true;
      } else {
        this.errorSubmit = false;
      }
      this.count = 0;
    },

    async createOrder() {
      console.log(this.noteOrder);
      const stt = this.cart.reduce((a, b) => b.amount * b.price + a, 0);
      console.log(stt);
      await api
        .createOrder({
          price: stt,
          paidAt: this.paidStatus ? new Date() : null,
          itemlist: this.cart,
          discount: this.discount,
          note: this.noteOrder,
        })
        .then((result) => {
          if (result.status === 200) {
            this.reload();
          } else {
            //error hander
          }
        })
        .catch((err) => {
          console.log(err);
        });
    },
  },
  created() {
    this.initPage();
  },
};
</script>

<style>
.menu {
  z-index: 1;
}
.menu--b-card {
  width: 320px;
}
.m-r-10 {
  margin-right: 10px;
}
.m-r-5 {
  margin-right: 5px;
}
.col--2 {
  position: fixed;
  right: 50px;
}

.buttons--submit {
  height: 60px;
  width: 100%;
}
.bigger {
  font-size: 1.25em;
}

.input-group {
  margin-right: 30px;
}
.input-group button {
  border: none;
}
.error {
  color: red;
}
.item-menu {
  width: 70vw;
}
.item-cart {
  width: 30vw;
  position: fixed;
  top: 20vh;
  right: 2vw;
}
.item-list {
  height: 350px;
  background-color: white;
  position: sticky;
  top: 190px;
  overflow-y: auto;
}
.item-list--child {
  padding: 10px;
  width: 100%;
}
</style>
