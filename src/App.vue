<template>
  <div id="app">
    <section class="hero">
      <div class="title-bar">
        <b-icon icon="settings" size="is-large" type="is-dark" @click.native="openModal()" />
        <h1 class="page-title">
          {{ title }}
        </h1>
      </div>
    </section>

    <div class="spinner-container">
      <section class="vue-winwheel">
        <div class="mobile-container">
          <div class="wheel-wrapper">
            <div class="canvas-wrapper">
              <canvas id="canvas" width="310" height="310">
                <p style="{color: white}" align="center">
                  Sorry, your browser doesn't support canvas. Please try Google Chrome.
                </p>
              </canvas>
            </div>
            <div class="button-wrapper">
              <a
                class="btn btn-play"
                href="#"
                @click.prevent="startSpin()"
                v-if="!loadingPrize && !wheelSpinning"
                >SPIN!</a
              >
            </div>
          </div>
        </div>
        <b-modal :active.sync="modalPrize" custom-class="prize">
          <canvas id="winner-canvas"></canvas>
          <section class="">
            <div class="winner">
              <h2>
                Yay you got the prize!!
              </h2>
              <h1>{{ prizeName }}</h1>
            </div>
          </section>
        </b-modal>
      </section>
    </div>
    <b-modal :active.sync="modalActive" has-modal-card full-screen :can-cancel="false">
      <div class="modal-card" style="width: auto">
        <header class="modal-card-head">
          <p class="modal-card-title">CONFIGURATIONS</p>
          <b-icon icon="close" size="is-medium" type="is-dark" @click.native="closeModal()" />
        </header>
        <section class="modal-card-body">
          <b-field label="Title" :type="titleType" custom-class="title-label">
            <b-input
              name="title"
              expanded
              size="is-medium"
              v-model="title"
              @change.native="setTitleType()"
            ></b-input>
          </b-field>
          <label class="label title-label">Options</label>
          <b-field label="" custom-class="title-label">
            <b-input
              name="option"
              expanded
              size="is-medium"
              v-model="option"
              v-on:keyup.enter.native="addOption()"
            ></b-input>
            <b-button
              class="button is-primary"
              size="is-medium"
              icon-left="plus"
              @click="addOption()"
              >Add</b-button
            >
          </b-field>
          <b-field grouped group-multiline class="options-section">
            <div class="control" v-bind:key="index" v-for="(item, index) in segments">
              <div class="control">
                <div class="tags has-addons">
                  <span
                    class="tag is-large"
                    :style="{ background: item.fillStyle, color: 'white' }"
                  >
                    <span> {{ item.text }} </span>
                  </span>
                  <a
                    @click="removeOption(index)"
                    role="button"
                    aria-label="Close tag"
                    tabindex="0"
                    class="tag is-delete is-large"
                  ></a>
                </div>
              </div>
            </div>
          </b-field>
        </section>
      </div>
    </b-modal>
  </div>
</template>

<script>
import * as Winwheel from "vue-winwheel/Winwheel";
import ConfettiGenerator from "confetti-js";

export default {
  name: "app",
  data() {
    const colors = [
      "#333",
      "#6A4C93",
      "#8AC926",
      "#1982C4",
      "#C3423F",
      "#048BA8",
      "#5887FF",
      "#2C2A4A",
      "#9CC976",
      "#E01A4F"
    ];
    const segments = [
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
    ];
    return {
      titleType: "",
      title: "GC AWARDS",
      option: null,
      modalActive: false,
      fillColorsLength: colors.length,
      fillColors: colors,
      segments: segments,
      loadingPrize: false,
      theWheel: null,
      modalPrize: false,
      wheelSpinning: false,
      prizeName: "",
      WinWheelOptions: {
        responsive: true,
        textAlignment: "center",
        textFontSize: 14,
        outterRadius: 410,
        innerRadius: 25,
        lineWidth: 8,
        animation: {
          type: "spinOngoing",
          duration: 0.5
        }
      }
    };
  },

  methods: {
    shuffleArray: function(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    },
    closeModal: function() {
      this.modalActive = false;
      this.titleType = "";
      this.option = null;
      this.initSpin();
    },
    openModal: function() {
      this.modalActive = true;
    },
    setTitleType: function() {
      this.titleType = "is-success";
    },
    addOption: function() {
      if (this.option) {
        const fillColor = this.getFillColor();
        const newOption = {
          textFillStyle: "#fff",
          fillStyle: fillColor,
          textFontSize: 20,
          textFontWeight: 100,
          text: this.option
        };
        this.segments.push(newOption);
        this.option = null;
      }
    },
    getFillColor: function() {
      return this.fillColors[(this.segments.length + 1) % this.fillColorsLength];
    },
    removeOption: function(index) {
      this.segments.splice(index, 1);
      this.segments = this.segments;
    },
    showPrize: function() {
      this.modalPrize = true;
    },
    hidePrize: function() {
      this.modalPrize = false;
    },
    startSpin: function() {
      if (this.wheelSpinning === false) {
        this.theWheel.startAnimation();
        this.wheelSpinning = true;
        this.theWheel = new Winwheel.Winwheel({
          ...this.WinWheelOptions,
          numSegments: this.segments.length,
          segments: this.shuffleArray(this.segments),
          animation: {
            type: "spinToStop",
            duration: Math.ceil(Math.random() * 10),
            spins: Math.ceil(Math.random() * 20),
            callbackFinished: this.onFinishSpin
          }
        });

        const prizeNumber = Math.floor(Math.random() * this.segments.length);
        const stopAt = (360 / this.segments.length) * prizeNumber - Math.floor(Math.random() * 60); //random location
        this.theWheel.animation.stopAngle = stopAt;
        this.theWheel.startAnimation();
        this.wheelSpinning = false;
      }
    },
    resetWheel: function() {
      if (this.wheelSpinning) {
        this.theWheel.stopAnimation(false);
      }
      this.theWheel = new Winwheel.Winwheel({
        ...this.WinWheelOptions,
        numSegments: this.segments.length,
        segments: this.shuffleArray(this.segments),
        clearTheCanvas: true,
        rotationAngle: 0
      });

      this.theWheel.draw();
      this.wheelSpinning = false;
    },
    initSpin: function() {
      this.loadingPrize = true;
      this.resetWheel();
      this.loadingPrize = false;
    },
    onFinishSpin: function(indicatedSegment) {
      this.prizeName = indicatedSegment.text;
      this.showPrize();

      setTimeout(() => {
        const confettiSettings = {
          target: "winner-canvas"
        };
        const confetti = new ConfettiGenerator(confettiSettings);
        confetti.render();
      }, 200);
    }
  },
  mounted() {
    this.initSpin();
  }
};
</script>

<style>
body {
  padding: none;
  margin: none;
  height: 100vh;
  background-repeat: no-repeat;
  background-size: cover;
  background: url(https://images.unsplash.com/photo-1495195129352-aeb325a55b65?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1655&q=80);
  overflow: scroll;
}
.wheel-wrapper {
  zoom: 1.2;
}
.vue-winwheel div h1:first-child {
  display: none !important;
}
.title-bar {
  justify-content: space-between;
  align-items: center;
  display: flex;
  padding: 1rem 1rem !important;
  -webkit-box-flex: 1;
  -ms-flex-positive: 1;
  flex-grow: 1;
  -ms-flex-negative: 0;
  flex-shrink: 0;
}

.page-title {
  color: white;
  font-family: "Exo", sans-serif;
  font-size: 6vh;
  font-weight: 900;
  text-shadow: 0 4px 24px rgba(0, 0, 0, 0.3);
  letter-spacing: 2px;
  margin: 0;
}

.modal {
  z-index: 9999999 !important;
}

.modal-content {
  background-color: white;
}

.title-label {
  font-size: 1.2rem !important;
}

.input:focus {
  border-color: #dbdbdb !important;
  box-shadow: none !important;
  -webkit-box-shadow: none !important;
}

.options-section {
  margin-top: 25px;
}
.vue-winwheel {
  text-align: center;
  background-image: url("/static/img/obstacle-run/bg-spinner-mobile.svg");
  background-size: cover;
  background-position: center bottom;
  background-repeat: no-repeat;
}
.vue-winwheel h1 {
  color: #b32656;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  font-size: 36px;
  line-height: 90px;
  letter-spacing: 4px;
  margin: 0;
}
.vue-winwheel h2 {
  margin: 0;
}

.vue-winwheel canvas#canvas {
  position: relative;
}
.vue-winwheel .canvas-wrapper {
  position: relative;
}
.vue-winwheel .canvas-wrapper:after {
  content: "";
  display: block;
  width: 42px;
  background: #c4376f;
  height: 42px;
  position: absolute;
  left: calc(50% - 25px);
  margin: auto;
  border-radius: 100%;
  top: calc(50% - 29px);
  border: 5px solid white;
  box-sizing: content-box;
}
.vue-winwheel .canvas-wrapper:before {
  content: "";
  display: block;
  width: 310px;
  background: #0f0f0f;
  height: 310px;
  position: absolute;
  left: 0;
  right: 0;
  margin: 0 auto;
  border-radius: 100%;
  top: 0;
}
.vue-winwheel .wheel-wrapper {
  position: relative;
}
.vue-winwheel .wheel-wrapper:before {
  content: "";
  width: 62px;
  height: 47px;
  position: absolute;
  top: -10px;
  left: calc(50% - 31px);
  right: 0;
  display: block;
  z-index: 99999;
  background-image: url("./spinner-marker.svg");
  background-repeat: no-repeat;
  background-size: contain;
  background-position: center;
}
.vue-winwheel .wheel-wrapper .button-wrapper {
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 231px;
  height: 118px;
}
.vue-winwheel .wheel-wrapper .btn.btn-play {
  padding: 0 58px !important;
  background: #c4376f;
  height: 40px;
  line-height: 40px;
  color: white;
  font-weight: bold;
  text-decoration: none;
  border-radius: 2px;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  letter-spacing: 2px;
}

.prize .modal-content {
  /* height: 75vh; */
  /* display: flex;
  flex-direction: column;
  justify-content: center; */
}

#winner-canvas {
  position: absolute;
  left: 0px;
  top: 0px;
}

.prize {
  overflow: hidden;
  display: flex;
  flex-direction: column;
  justify-items: center;
  justify-content: center;
  align-items: center;
}
.winner {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  align-self: center;
  height: 50vh;
}
</style>
