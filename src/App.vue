<template>
  <div id="main">
    <div class="container">
      <h1>{{ gesture.name == 'load' ? 'Loading' : gesture.name == 'thumbs_up' ? '👍' : '✌️' }}</h1>
      <video autoplay="true" :srcObject="stream" ref="previewVideo"></video>
    </div>
  </div>
</template>

<style scoped>
  #main {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #121212;
    color: white;
    display: grid;
    place-items: center;
    text-align: center;
  }
  .container {
    /* Add styles for the container if needed */
  }
  video {
    border-radius: 12px;
    width: 11rem;
    height: 18rem;
    transform: scaleX(-1);
  }
  h1 {
    margin: 1rem 0;
  }
</style>

<script>
import * as tf from "@tensorflow/tfjs";
import * as handpose from "@tensorflow-models/handpose";
import * as fp from "fingerpose";

export default {
  data() {
    return {
      stream: null,
      gesture: {}
    };
  },
  async created() {
    await this.loadHandpose();
    this.recordVideo();
  },
  methods: {
    async loadHandpose() {
      this.gesture = { name: "load"};
      const net = await handpose.load();
      console.log("Handpose model loaded.");
      setInterval(() => {
        this.detect(net);
      }, 10);
    },
    async detect(net) {
      let video = this.$refs.previewVideo;
      if (video) {
        try {
          const hand = await net.estimateHands(video);
          if (hand.length > 0) {
        const GE = new fp.GestureEstimator([
          fp.Gestures.VictoryGesture,
          fp.Gestures.ThumbsUpGesture,
        ]);
        const gesture = await GE.estimate(hand[0].landmarks, 4);
        if (gesture.gestures !== undefined && gesture.gestures.length > 0) {
           console.log(gesture.gestures);
          
          
          let sorted = gesture.gestures.sort((x, y) => x.score - y.score)[1];
          this.gesture = sorted;
        
        }
          }
        } catch (err) {
          console.log("TFJS error:", err)
        }
      }
    },
    async recordVideo() {
      let constraints = {
        video: {
          facingMode: 'user',
          width: { ideal: 1080 },
          height: { ideal: 720 },
        },
      };
      if (navigator.mediaDevices.getUserMedia) {
        try {
          let stream = await navigator.mediaDevices.getUserMedia(constraints);
          this.stream = stream;
        } catch (error) {
          console.error("Error accessing the camera:", error);
        }
      } else {
        console.error("getUserMedia is not supported in this browser.");
      }
    },
  },
};
</script>