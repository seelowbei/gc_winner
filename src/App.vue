<template>
  <div id="app">
    <section class="hero">
      <div class="title-bar">
        <b-icon
          icon="settings"
          size="is-large"
          type="is-dark"
          @click.native="openModal()"
        />
        <h1 class="page-title">{{ title }}</h1>
      </div>
    </section>

    <div class="spinner-container">
      <section class="vue-winwheel">
        <div class="mobile-container">
          <div class="wheel-wrapper">
            <div class="canvas-wrapper" id="canvas-wrapper">
              <canvas id="canvas" width="310" height="310">
                <p
                  style="
                     {
                      color: white;
                    }
                  "
                  align="center"
                >
                  Sorry, your browser doesn't support canvas. Please try Google
                  Chrome.
                </p>
              </canvas>
            </div>
            <div class="button-wrapper">
              <b-button
                class="btn btn-play"
                type="is-danger"
                icon-left="star"
                icon-right="star"
                :disabled="wheelSpinning"
                @click.prevent="startSpin()"
                >SPIN</b-button
              >

              <b-button
                class="btn btn-shuffle"
                type="is-light"
                icon-left="shuffle"
                :disabled="wheelSpinning"
                @click.prevent="shuffle()"
                >SHUFFLE</b-button
              >
            </div>
          </div>
        </div>
        <b-modal
          :active.sync="modalPrize"
          custom-class="prize"
          @close="resetWheel()"
        >
          <canvas id="winner-canvas"></canvas>
          <section class>
            <div class="winner">
              <h2>Yay you got the prize!!</h2>
              <h1>{{ prizeName }}</h1>
            </div>
          </section>
        </b-modal>
      </section>
    </div>
    <b-modal
      :active.sync="modalActive"
      has-modal-card
      full-screen
      :can-cancel="false"
    >
      <div class="modal-card" style="width: auto">
        <header class="modal-card-head">
          <p class="modal-card-title">CONFIGURATIONS</p>
          <b-icon
            icon="close"
            size="is-medium"
            type="is-dark"
            @click.native="closeModal()"
          />
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
          <div class="options-section">
            <label class="label title-label">Options</label>
            <span>
              <b-button
                type="is-primary"
                inverted
                @click="applyDefaultOptions()"
              >
                <b-icon
                  icon="undo-variant"
                  size="is-small"
                  type="is-primary"
                />Use Default
              </b-button>
              <b-button type="is-primary" inverted @click="removeAllOptions()">
                <b-icon icon="delete" size="is-small" type="is-primary" />Delete
                All
              </b-button>
            </span>
          </div>

          <b-field label custom-class="title-label">
            <b-input
              name="option"
              expanded
              size="is-medium"
              v-model="option"
              v-on:keyup.enter.native="addOption()"
            ></b-input>
            <b-button
              class="button is-dark"
              size="is-medium"
              icon-left="plus"
              @click="addOption()"
              >Add</b-button
            >
          </b-field>
          <b-field grouped group-multiline class="options-section">
            <div
              class="control"
              v-bind:key="index"
              v-for="(item, index) in segments"
            >
              <div class="control">
                <div class="tags has-addons">
                  <span
                    class="tag is-large"
                    :style="{ background: item.fillStyle, color: 'white' }"
                  >
                    <span>{{ item.text }}</span>
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

          <b-field
            label="Discord Webhook"
            :type="webhookType"
            custom-class="webhook-label"
          >
            <b-input
              name="Discord Webhook"
              expanded
              size="is-medium"
              v-model="webhook"
              @change.native="setWebhookType()"
              >></b-input
            >
          </b-field>
        </section>
      </div>
    </b-modal>
  </div>
</template>

<script>
import * as Winwheel from "vue-winwheel/Winwheel";
import ConfettiGenerator from "confetti-js";
import { DEFAULT_OPTIONS, COLORS } from "./constants";
import axios from "axios";

export default {
  name: "app",
  data() {
    const segments =
      JSON.parse(localStorage.getItem("segments")) || DEFAULT_OPTIONS;
    const title = localStorage.getItem("title") || "GC AWARDS";
    const webhook = localStorage.getItem("webhook") || "";
    return {
      titleType: "",
      webhookType: "",
      webhook: webhook,
      title: title,
      option: null,
      modalActive: false,
      fillColorsLength: COLORS.length,
      fillColors: COLORS,
      segments: segments,
      theWheel: null,
      modalPrize: false,
      wheelSpinning: false,
      prizeName: "",
      spin: 0,
      WinWheelOptions: {
        responsive: true,
        textAlignment: "center",
        textFontSize: 14,
        outterRadius: 410,
        innerRadius: 25,
        lineWidth: 8,
        animation: {
          type: "spinOngoing",
          duration: 0.5,
        },
      },
    };
  },
  computed: {
    standupBotToken: function () {
      const urlParams = new URLSearchParams(window.location.search);
      return urlParams.get("token");
    },
  },
  methods: {
    validWebhookUrl: function () {
      const hook = this.webhook;
      if (hook) {
        try {
          new URL(hook);
        } catch (_) {
          return false;
        }
      }

      return true;
    },
    shuffleArray: function (array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    },
    closeModal: function () {
      this.modalActive = false;
      this.titleType = "";
      this.webhookType = "";
      this.option = null;
      this.shuffle();
    },
    openModal: function () {
      this.modalActive = true;
    },
    setTitleType: function () {
      localStorage.setItem("title", this.title);
      this.titleType = "is-success";
    },
    setWebhookType: function () {
      localStorage.setItem("webhook", this.webhook);
      this.webhookType = this.validWebhookUrl() ? "is-success" : "is-danger";
    },
    removeAllOptions: function () {
      this.segments = [];
      localStorage.clear();
    },
    applyDefaultOptions: function () {
      this.segments = DEFAULT_OPTIONS;
    },
    addOption: function () {
      if (this.option) {
        const fillColor = this.getFillColor();
        const newOption = {
          textFillStyle: "#fff",
          fillStyle: fillColor,
          textFontSize: 20,
          textFontWeight: 100,
          text: this.option,
        };
        this.segments.push(newOption);
        localStorage.setItem("segments", JSON.stringify(this.segments));
        this.option = null;
      }
    },
    getFillColor: function () {
      return this.fillColors[
        (this.segments.length + 1) % this.fillColorsLength
      ];
    },
    removeOption: function (index) {
      this.segments.splice(index, 1);
      this.segments = this.segments;
      localStorage.setItem("segments", JSON.stringify(this.segments));
    },
    showPrize: function () {
      this.modalPrize = true;
    },
    hidePrize: function () {
      this.modalPrize = false;
    },
    startSpin: function () {
      if (this.segments.length === 0) {
        this.$buefy.toast.open({
          duration: 3000,
          message: "Please add some options.",
          position: "is-top",
          type: "is-danger",
        });
        return;
      }
      if (this.wheelSpinning === false) {
        this.theWheel.startAnimation();
        this.wheelSpinning = true;
        const segmentsLength = this.segments.length;
        this.theWheel = new Winwheel.Winwheel(
          {
            ...this.WinWheelOptions,
            numSegments: segmentsLength,
            segments: this.shuffleArray(this.segments),
            pins: {
              number: segmentsLength * 2,
              outerRadius: 5,
              margin: 0,
              fillStyle: "black",
              strokeStyle: "rgb(239, 239, 239)",
            },
            animation: {
              type: "spinToStop",
              duration: Math.ceil(Math.random() * 10),
              spins: Math.ceil(Math.random() * 20),
              callbackFinished: this.onFinishSpin,
              soundTrigger: "pin",
              callbackSound: () => {
                let audio = new Audio("tick.mp3");
                audio.pause();
                audio.currentTime = 0;
                audio.play();
              },
            },
          },
          false
        );

        const prizeNumber = Math.floor(Math.random() * this.segments.length);
        const stopAt =
          (360 / this.segments.length) * prizeNumber -
          Math.floor(Math.random() * 60); //random location
        this.theWheel.animation.stopAngle = stopAt;
        this.theWheel.startAnimation();
      }
    },
    resetWheel: function () {
      if (this.wheelSpinning) {
        this.theWheel.stopAnimation(false);
        this.wheelSpinning = false;
      }

      this.theWheel = new Winwheel.Winwheel({
        ...this.WinWheelOptions,
        numSegments: this.segments.length,
        segments: this.shuffleArray(this.segments),
        clearTheCanvas: true,
        rotationAngle: 0,
      });

      this.theWheel.draw();
      this.theWheel.rotationAngle = 0;
    },
    onFinishSpin: function (indicatedSegment) {
      this.wheelSpinning = false;
      this.prizeName = indicatedSegment.text;
      this.showPrize();
      this.spin = 1;
      this.postToWebhook();
      setTimeout(() => {
        const confettiSettings = {
          target: "winner-canvas",
        };
        const confetti = new ConfettiGenerator(confettiSettings);
        confetti.render();
      }, 200);
    },
    shuffle: function () {
      this.spin > 0 ? window.location.reload() : this.resetWheel();
    },
    members() {
      return this.segments.reduce((list, item) => {
        if (item.text !== "👻👻") {
          list.push(item.text);
        }
        return list;
      }, []);
    },
    postToWebhook: function () {
      if (this.prizeName === "👻👻") {
        console.log("Invalid prize name or stand up bot token.");
        return;
      } else if (this.standupBotToken) {
        this.postToSJStandupBot();
      } else {
        this.postToInputWebhook();
      }
    },
    postToSJStandupBot: function () {
      const members = this.members().sort().join(", ");
      const sharing = `${this.prizeName} will host the next standup! Members: ${members}`;
      const body = {
        query: `
        mutation ($contributor: String!, $sharing: String!) {
          insert_shares (objects: {
            contributor: $contributor,
            sharing: $sharing
          }) {
            affected_rows
          }
        }
        `,
        variables: { contributor: "LUCKY SPIN", sharing },
      };

      axios
        .post("https://sj-stand-up-bot.herokuapp.com/v1/graphql", body, {
          headers: {
            "content-type": "application/json",
            Authorization: `Bearer ${this.standupBotToken}`,
          },
        })
        .then(function (response) {
          console.log(response);
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    postToInputWebhook: function () {
      if (this.webhook && this.validWebhookUrl()) {
        axios
          .post(this.webhook, {
            tts: true,
            content: `Yay, ${this.prizeName} won the prize!!!`,
            embeds: [
              {
                title: this.title,
                color: 261297,
                footer: {
                  text: `🥳🥳 ${this.prizeName} won! 🥳🥳`,
                },
                thumbnail: {
                  url:
                    "https://media.giphy.com/media/3o7WIOU62pfc2Ox76o/giphy.gif",
                },
                // image: {
                //   url: "https://picsum.photos/200",
                // },
                fields: [
                  {
                    name: "🥳🥳🥳🥳🥳🥳🥳🥳🥳🥳",
                    value: `Yay, ${this.prizeName} won the prize!!!`,
                  },
                  {
                    name: "Members",
                    value: this.members().sort().join(", "),
                  },
                ],
              },
            ],
          })
          .then(function (response) {
            console.log(response);
          })
          .catch(function (error) {
            console.log(error);
          });
      }
    },
  },
  mounted() {
    this.resetWheel();
  },
};
</script>

<style>
body {
  padding: none;
  margin: none;
  height: 100vh;
  background-repeat: no-repeat;
  background-size: cover;
  background-image: url(https://images.unsplash.com/photo-1495195129352-aeb325a55b65?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1655&q=80);
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

.vue-winwheel .wheel-wrapper .btn {
  height: 40px;
  line-height: 40px;
  color: white;
  font-weight: bold;
  text-decoration: none;
  border-radius: 2px;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  letter-spacing: 2px;
  margin-top: 15px;
}

.vue-winwheel .wheel-wrapper .btn.btn-play {
  background: #c4376f;
  padding: 0 58px !important;
}

.vue-winwheel .wheel-wrapper .btn.btn-shuffle {
  background: #d59eb4;
  padding: 0 48px !important;
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

.remove-all {
  margin-left: 5px;
}

.options-section {
  display: flex;
  justify-content: space-between;
}
</style>
