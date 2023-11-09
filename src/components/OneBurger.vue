<template>
  <div>
    <div>
      <h3>{{ burger.name }}</h3>

      <section>
        <h5>Amount ordered: {{ amountOrdered }}</h5>
        <button v-on:click="plusOne">
          <img
            class="orderedButton"
            src="https://cdn-icons-png.flaticon.com/512/149/149145.png"
            alt="Plus"
          />
        </button>

        <button v-on:click="minusOne">
          <img
            class="orderedButton"
            src="https://cdn-icons-png.flaticon.com/512/18/18657.png"
            alt="Minus"
          />
        </button>
      </section>

      <img v-bind:src="burger.URL" alt="" style="width: 200px" />

      <p>Calories: {{ burger.kCal }} kCal</p>

      <section class="allergies">
        <p>Allergies:</p>

        <ul>
          <li v-if="burger.gluten"><span class="allergyspan">Gluten</span></li>

          <li v-if="burger.lactose">
            <span class="allergyspan">Lactose</span>
          </li>
          <li v-if="burger.cursed"><span class="allergyspan">ALL</span></li>
        </ul>
      </section>
    </div>
  </div>
</template>

<script>
export default {
  name: "OneBurger",
  props: {
    burger: Object,
  },

  data: function () {
    return {
      amountOrdered: 0,
    };
  },

  methods: {
    plusOne: function () {
      this.amountOrdered += 1;
      this.$emit("orderedBurger", {
        name: this.burger.name,
        amount: this.amountOrdered,
      });
    },

    minusOne: function () {
      if (this.amountOrdered > 0) {
        this.amountOrdered -= 1;
        this.$emit("orderedBurger", {
          name: this.burger.name,
          amount: this.amountOrdered,
        });
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
section.allergies {
  color: #ff5500;
}
.allergyspan {
  text-transform: uppercase;
}
.orderedButton {
  width: 20px;
  height: auto;
}
</style>
