<script setup>
import { getCurrentInstance, onBeforeMount, ref, onMounted, watch } from "vue";
import Papa from "papaparse";
import axios from "axios";
let blockValues = ref([1, 2, 3]);
const activeCode = ref();
const loading = ref(true);
const error = ref(false);

axios
  .get(
    "https://script.google.com/macros/s/AKfycbyPcbm2nt-cnA2zse5KHEIHFPi674wQx6WWdZwvf_CQJomge-7NC_RellTzhlX7eLjHXg/exec"
  )
  .then((res) => {
    const data = JSON.parse(res.data);
    blockValues.value = Object.keys(data).map((i) => {
      return {
        answer: i,
        keyCode: data[i],
        result: null,
      };
    });

    blockValues.value.sort((start, next) => {
      const order = [
        49, 50, 51, 52, 81, 87, 69, 82, 65, 83, 68, 70, 90, 88, 67, 86,
      ];
      return order.indexOf(start.keyCode) - order.indexOf(next.keyCode);
    });

    loading.value = false;
  });

const clearValues = () => {
  blockValues.value.forEach((i) => (i.result = null));
};

const waiting = ref(true);
const cancelWaiting = () => {
  waiting.value = false;
};

onBeforeMount(() => {
  window.addEventListener("keydown", (e) => {
    if (e.keyCode === 27) {
      window.close();
      return;
    }

    // backspace
    if (e.keyCode === 8) {
      blockValues.value[activeCode.value].result = null;
    }

    // // space
    if (e.keyCode === 32) {
      waiting.value = true;
      clearValues();
      return;
    }
    // if (e.keyCode === 32) {
    //   error.value = true;
    //   setTimeout(() => {
    //     error.value = false;
    //   }, 2000);
    //   return;
    // }

    // enter
    if (e.keyCode === 13) {
      if (waiting.value) {
        cancelWaiting();
        return;
      }
      clearValues();
      return;
    }

    const activeIndex = blockValues.value.findIndex(
      (i) => i.keyCode === e.keyCode
    );

    if (event.shiftKey && activeIndex !== -1) {
      activeCode.value = activeIndex;
      blockValues.value[activeIndex].result = false;
      setTimeout(() => {
        blockValues.value[activeIndex].result = null;
      }, 2000);
      return;
    }
    if (activeIndex !== -1) {
      activeCode.value = activeIndex;
      blockValues.value[activeIndex].result = true;
    }
  });
});
</script>

<template>
  <div v-if="waiting" class="welcome">
    <img src="./assets/banner.png" alt="" />
  </div>
  <div v-if="!loading && !waiting" class="container">
    <div
      class="block"
      :class="
        item.result === true ? 'correct' : item.result === false ? 'wrong' : ''
      "
      v-for="(item, index) in blockValues"
      :key="`${item.answer}-${index}`"
    >
      <h1 v-html="item.answer"></h1>
      <div v-show="item.result === false" class="wrong-container">
        <img class="wrong" src="./assets/wrong.png" />
      </div>
    </div>
  </div>
</template>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}

body {
  background: url("assets/bg.png"); /* fallback for old browsers */
  background-size: cover;
  // width: 100%;
  height: 100vh;
  background-position: center;
  background-repeat: no-repeat;
  margin: 0;
}

.container {
  margin: auto;
  width: 90vw;
  height: auto;
  display: grid;
  grid-template-columns: repeat(4, minmax(max-content, 300px));
  grid-template-rows: repeat(4, minmax(max-content, 180px));
  // grid-template-rows: repeat(4, 300px);
  gap: 8px;
  justify-content: center;
}
.block {
  background: url("assets/block.png");
  background-size: contain;
  background-position: center;
  background-repeat: no-repeat;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  position: relative;
  padding: 4px;
  p {
    margin: 0;
  }
  &.correct {
    position: relative;
    &::before {
      position: absolute;
      content: "";
      background: url("./assets/border.png");
      background-size: contain;
      background-position: center;
      background-repeat: no-repeat;
      width: 100%;
      height: 100%;
    }
  }
  h1 {
    margin: 0;
    font-size: 36px;
    color: #593328;
  }
}

.wrong-container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 100;
  .wrong {
    width: 100%;
    animation: rubberBand 0.8s;
    animation-iteration-count: 3;
  }
}

@keyframes rubberBand {
  from {
    transform: scale(1);
  }

  50% {
    transform: scale(1.5);
  }

  to {
    transform: scale(1);
  }
}

.rubberBand {
  -webkit-animation-name: rubberBand;
  animation-name: rubberBand;
}

.welcome {
  background: url("assets/welcome.jpg");
  background-size: cover;
  width: 100%;
  height: 100%;
  background-position: center;
  background-repeat: no-repeat;
  img {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    width: 50%;
    top: 50%;
    animation: disappear 1.5s infinite;
  }
}
@keyframes disappear {
  0% {
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    opacity: 1;
  }
}
</style>
