<template>
  <div>
    <header>
      <section id="headline">
        <img
          class="headerImage"
          src="https://img.freepik.com/free-photo/exploding-burger-with-vegetables-melted-cheese-black-background-generative-ai_157027-1734.jpg?size=626&ext=jpg&ga=GA1.1.1826414947.1699142400&semt=ais"
          alt="Image can't be loaded"
        />
        <h1 id="absolute">Welcome to BurgerHeaven Online</h1>
      </section>
    </header>

    <main>
      <section class="selectBurger">
        <h2>Select Burger</h2>

        <p>This is where you execute burger selection</p>

        <div class="burgerGrid">
          <Burger
            v-for="burger in burgers"
            v-bind:burger="burger"
            v-bind:key="burger.name"
            v-on:orderedBurger="burgersToBeOrdered($event)"
          ></Burger>
        </div>
      </section>

      <section id="customerInformation">
        <h2>Customer information</h2>

        <p>This is where you provide necessary information</p>

        <h3>Delivery information</h3>

        <form>
          <p>
            <label for="Fullname">Fullname</label><br />
            <input
              type="text"
              v-model="customerInformation.Fullname"
              id="Fullname"
              required="required"
              placeholder="First-and Last name"
            />
          </p>

          <p>
            <label for="Email">Email</label><br />
            <input
              type="text"
              v-model="customerInformation.Email"
              id="Email"
              required="required"
              placeholder="Email adress"
            />
          </p>

          <p>
            <label for="Payment"><b>Payment</b></label>

            <br />

            <select v-model="customerInformation.Payment" id="payment">
              <option>Klarna</option>
              <option>Paypal</option>
              <option>Credit Card</option>
              <option>Check</option>
            </select>
          </p>

          <p>
            <label for="Gender" style="display: inline-block"
              ><b>Gender:</b></label
            >

            <br />

            <input
              type="radio"
              v-model="customerInformation.Gender"
              id="Do not wish to provide"
              value="Do not wish to provide"
              checked="checked"
            />
            <label for="Do not wish to provide">Do not wish to provide</label>

            <br />

            <input
              type="radio"
              v-model="customerInformation.Gender"
              id="Man"
              value="Man"
            />
            <label for="Man">Man</label>

            <br />

            <input
              type="radio"
              v-model="customerInformation.Gender"
              id="Woman"
              value="Woman"
            />
            <label for="Woman">Woman</label>
          </p>
        </form>
      </section>

      <h3>Press where you want the burgers delivered.</h3>

      <section id="mapScroll">
        <section id="map" v-on:click="setLocation">
          <div
            v-bind:style="{
              left: this.location.x + 'px',
              top: this.location.y + 'px',
            }"
          >
            T
          </div>
        </section>
      </section>

      <section style="margin: 1% 20px">
        <button v-on:click="addToOrder" type="hover">
          Place order
          <img
            src="https://media.istockphoto.com/id/1079725292/vector/green-tick-checkmark-vector-icon-for-checkbox-marker-symbol.jpg?s=612x612&w=0&k=20&c=OvOpxX8ZFuc5NufZTJDbpwGKvgFUmfZjY68MICmEzX4="
            alt=""
            style="width: 14px; height: 14px"
          />
        </button>
      </section>

      <div v-if="showReceipt">
        <section class="receipt">
          <h4>Your Receipt:</h4>

          <p
            v-for="(number, foodItems) in orderedBurgers"
            v-bind:key="'number' + foodItems"
          >
            {{ number }} x {{ foodItems }}
          </p>

          <hr />

          <span style="font-weight: 700"> Payment Method:</span>
          {{ customerInformation.Payment }}
        </section>

        <p>
          <span v-if="order != null"> STATUS: {{ order.status }}</span>

          <span v-else> STATUS: Preparing </span>
        </p>
      </div>
    </main>

    <br />
    <hr />

    <footer>
      <p>COPYRIGHT 2023</p>
    </footer>
  </div>
</template>

<script>
import Burger from "../components/OneBurger.vue";
import io from "socket.io-client";
import menu from "../assets/menu.json";

const socket = io();
/*
function menuItem(nameOfBurger, URL, calories, glutenOrNot, lactoseOrNot, cursedOrNot) {
  this.name = nameOfBurger;
  this.URl = URL;
  this.kCal = calories;
  this.gluten = glutenOrNot;
  this.lactose = lactoseOrNot;
  this.cursed = cursedOrNot;
}
*/

function emailCheck(str) {
  const atIndex = str.search("@");

  if (atIndex < 0) {
    return true;
  }

  for (let i = atIndex + 1; i < str.length; i++) {
    if (str[i] == ".") {
      return false;
    }
  }
  return true;
}

export default {
  name: "HomeView",
  components: {
    Burger,
  },
  data: function () {
    return {
      burgers: menu,

      customerInformation: {
        Fullname: "",
        Email: "",
        Payment: "Klarna",
        Gender: "Do not wish to provide",
      },

      orderedBurgers: {},

      location: {
        x: 0,
        y: 0,
      },

      showReceipt: false,

      order: null,
    };
  },

  created: function () {
    socket.on("customerStatus", (data) => {
      this.order = data.order;
    });
  },

  methods: {
    setLocation: function (event) {
      var offset = {
        x: event.currentTarget.getBoundingClientRect().left,
        y: event.currentTarget.getBoundingClientRect().top,
      };

      this.location.x = event.clientX - offset.x - 10;
      this.location.y = event.clientY - offset.y - 10;
    },

    deleteBurger: function (key) {
      delete this.orderedBurgers[key];
    },

    ifBurgersAreOrdered: function () {
      var keys = Object.keys(this.orderedBurgers);
      let bool = false;
      let counter = 0;

      if (keys.length < 1) {
        return true;
      }

      keys.forEach((element) => {
        if (this.orderedBurgers[element] < 1) {
          this.deleteBurger(element);
          counter++;
        }
      });

      if (counter == keys.length) {
        bool = true;
      }

      return bool;
    },

    addToOrder: function () {
      if (this.location.x == 0 && this.location.y == 0) {
        alert("You haven't chosen a location yet");
        return;
      }

      if (this.customerInformation.Fullname.length < 1) {
        alert("Name is required");
        return;
      }

      if (emailCheck(this.customerInformation.Email)) {
        alert("Correct email is required");
        return;
      }

      if (this.ifBurgersAreOrdered()) {
        alert("You have to order a minimum of a single burger");
        return;
      }

      socket.emit("addOrder", {
        orderId: this.getOrderNumber(),

        customer: {
          name: this.customerInformation.Fullname,
          email: this.customerInformation.Email,
          payment: this.customerInformation.Payment,
          gender: this.customerInformation.Gender,
        },

        details: {
          x: this.location.x,
          y: this.location.y,
        },

        orderItems: this.orderedBurgers,

        status: "Preparing",

        showCustomerInfo: false,
      });

      this.showReceipt = true;
    },

    burgersToBeOrdered: function (event) {
      this.orderedBurgers[event.name] = event.amount;
      var keys = Object.keys(this.orderedBurgers);
      keys.forEach((element) => {
        if (this.orderedBurgers[element] < 1) {
          this.deleteBurger(element);
        }
      });
    },

    getOrderNumber: function () {
      return Math.floor(Math.random() * 100000);
    },
  },
};
</script>

<style>
#map {
  position: relative;
  width: 1920px;
  height: 1078px;
  cursor: crosshair;
  background: url("../../public/img/polacks.jpg");
}

h1 {
  font-family: fantasy;
  font-weight: bold;
  text-align: center;
  margin: 5% 0;
}

body {
  font-family: arial;
}

#customerInformation {
  background-color: black;
  color: whitesmoke;
  border: 2px dotted whitesmoke;
  padding-left: 30px;
}

button:hover {
  color: darkgray;
  cursor: pointer;
}

.headerImage {
  opacity: 0.77;
  width: 100%;
  height: auto;
}

#absolute {
  position: absolute;
  padding-left: 35%;
  top: 8%;
  text-align: center;
}

#headline {
  height: 300px;
  overflow: hidden;
}

.burgerGrid {
  display: grid;
  width: 100%;
  grid-template-columns: repeat(auto-fill, 20em);
}

#map div {
  position: absolute;
  background: black;
  color: white;
  border-radius: 10px;
  width: 20px;
  height: 20px;
  text-align: center;
}

#mapScroll {
  overflow: scroll;
  height: 400px;
  width: auto;
}

.receipt {
  width: 300px;
  padding-left: 15px;
  padding-bottom: 5px;
  border: dimgray;
  border-style: solid;
  border-radius: 15px;
}
</style>
