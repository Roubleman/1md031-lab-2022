<template>
  <div id="orders">
    <div id="orderList">
      <button v-on:click="clearQueue">Clear Queue</button>
    </div>
    <div
      id="dots"
      v-bind:style="{
        background: 'url(' + require('../../public/img/polacks.jpg') + ')',
      }"
    >
      <div
        class="dåligFantasiNu hoverable"
        v-for="(order, key) in orders"
        v-bind:style="{
          left: order.details.x + 'px',
          top: order.details.y + 'px',
        }"
        v-bind:key="'dots' + key"
        v-on:click="toggleShowCustomerInfo(key)"
      >
        {{ key }}

        <div class="snyggareCustomerInfo" v-if="order.showCustomerInfo">
          <span style="font-size: medium; font-weight: 500"
            >Customer info:</span
          >
          {{ order.customer.name }} ({{ order.customer.email }},
          {{ order.customer.payment }}, {{ order.customer.gender }})
          <span style="font-size: medium; font-weight: 500">Order:</span>

          <div
            v-for="(number, foodItem) in order.orderItems"
            v-bind:key="'number' + foodItem"
          >
            {{ number }} x {{ foodItem }}

            <br />
          </div>

          Order is currently being
          <span style="text-decoration: underline">{{ order.status }}</span>
          <br />
          <button @click="changeButtonText(key)">
            {{ order.status }} order
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import io from "socket.io-client";
const socket = io();

export default {
  name: "DispatcherView",
  data: function () {
    return {
      orders: null,
    };
  },
  created: function () {
    socket.on("currentQueue", (data) => {
      this.orders = data.orders;
    });
  },
  methods: {
    clearQueue: function () {
      socket.emit("clearQueue");
    },

    changeButtonText: function (key) {
      if (this.orders[key].status == "Preparing") {
        socket.emit("setStatus", { orderId: key, status: "Cooking" });
      } else if (this.orders[key].status == "Cooking") {
        socket.emit("setStatus", { orderId: key, status: "Dispatching" });
      } else {
        socket.emit("setStatus", { orderId: key, status: "Delivered" });

        // TA BORT ORDER NÄR DELIVERED.
      }
    },

    toggleShowCustomerInfo: function (key) {
      this.orders[key].showCustomerInfo = !this.orders[key].showCustomerInfo;
    },
  },
};
</script>
<style>
#orderList {
  top: 1em;
  left: 1em;
  position: absolute;
  z-index: 2;
  color: black;
  background: rgba(255, 255, 255, 0.5);
  padding: 1em;
  overflow: scroll;
  height: 20px;
}

.snyggareCustomerInfo {
  position: absolute;
  z-index: 2;
  background-color: black;
  width: 300px;
  height: auto;
}

#dots {
  position: relative;
  margin: 0;
  padding: 0;
  background-repeat: no-repeat;
  width: 1920px;
  height: 1078px;
  cursor: crosshair;
}

.dåligFantasiNu {
  position: absolute;
  background: black;
  color: white;
  border-radius: 10px;
  width: 20px;
  height: 20px;
  text-align: center;
}

.hoverable :hover {
  cursor: pointer;
}
</style>
