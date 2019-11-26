<template>
  <div id="app">
    <section class="hero">
      <div class="hero-body">
        <div class="container">
          <h1 class="title has-text-centered">
            GC AWARDS
          </h1>
        </div>
      </div>
    </section>
    <div class="spinner-container">
      <vue-winwheel :segments="options"> </vue-winwheel>
    </div>
  </div>
</template>

<script>
import VueWinwheel from "vue-winwheel/vue-winwheel";
export default {
  name: "app",
  components: {
    VueWinwheel
  },
  data() {
    return {
      options: [
        {
          textFillStyle: "#fff",
          fillStyle: "#333",
          textFontSize: 20,
          textFontWeight: 100,
          text: "👻👻"
        },
        {
          textFillStyle: "#fff",
          fillStyle: "#6A4C93",
          textFontSize: 20,
          textFontWeight: 100,
          text: "Sam"
        },
        {
          textFillStyle: "#fff",
          fillStyle: "#8AC926",
          textFontSize: 20,
          textFontWeight: 100,
          text: "Jany"
        },
        {
          textFillStyle: "#fff",
          fillStyle: "#1982C4",
          textFontSize: 20,
          textFontWeight: 100,
          text: "Nazmi"
        },
        {
          textFillStyle: "#fff",
          fillStyle: "#C3423F",
          textFontSize: 20,
          textFontWeight: 100,
          text: "Celine"
        },
        {
          textFillStyle: "#fff",
          fillStyle: "#048BA8",
          textFontSize: 18,
          textFontWeight: 100,
          text: "Thiam Hock"
        },
        {
          textFillStyle: "#fff",
          fillStyle: "#5887FF",
          textFontSize: 20,
          textFontWeight: 100,
          text: "Jarrett"
        },
        {
          textFillStyle: "#fff",
          fillStyle: "#2C2A4A",
          textFontSize: 20,
          textFontWeight: 100,
          text: "Vincent"
        },
        {
          textFillStyle: "#fff",
          fillStyle: "#9CC976",
          textFontSize: 20,
          textFontWeight: 100,
          text: "Zek"
        },
        {
          textFillStyle: "#fff",
          fillStyle: "#333",
          textFontSize: 20,
          textFontWeight: 100,
          text: "👻👻"
        },
        {
          textFillStyle: "#fff",
          fillStyle: "#E01A4F",
          textFontSize: 20,
          textFontWeight: 100,
          text: "Justin"
        },

        {
          textFillStyle: "#fff",
          fillStyle: "#C3423F",
          textFontSize: 20,
          textFontWeight: 100,
          text: "YaoJie"
        },

        {
          textFillStyle: "#fff",
          fillStyle: "#6A4C93",
          textFontSize: 20,
          textFontWeight: 100,
          text: "Jaslyn"
        },
        {
          textFillStyle: "#fff",
          fillStyle: "#048BA8",
          textFontSize: 20,
          textFontWeight: 100,
          text: "Jason"
        }
      ]
    };
  },

  methods: {},

  mounted() {}
};
</script>

<style>
@import "~bulma/css/bulma.css";
body {
  padding: none;
  margin: none;
  height: 100vh;
  background: url(https://images.unsplash.com/photo-1495195129352-aeb325a55b65?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1655&q=80);
}
.wheel-wrapper {
  zoom: 1.3;
}
.vue-winwheel div h1:first-child {
  display: none !important;
}
</style>
