<template>
  <div class="flex items-center justify-center h-screen bg-gray-900 relative">
    <img
      src="/img/perso1_decor.png"
      class="absolute bottom-0 right-0 w-80"
      alt=""
    />

    <div class="absolute bottom-4 left-0 right-0 flex justify-center">
      <img src="/img/titre_bde.png" class="w-60" alt="" />
    </div>

    <div
      id="container"
      class="h-3/5 aspect-square flex items-center justify-center relative"
      style="max-width: 900px; min-width: 700px"
    >
      <img id="mainImage" :src="'/img/' + imageList[index]" class="h-full" />
      <div class="absolute h-full aspect-square">
        <div
          @click="tryPrevious()"
          class="h-full w-1/2 inline-block prev-cursor"
          style="color: rgba(255, 0, 0, 0.5)"
        ></div>
        <div
          @click="tryNext()"
          class="h-full w-1/2 inline-block next-cursor"
          style="color: rgba(0, 0, 255, 0.5)"
        ></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";

const index = ref(1);
const preloadedImages = ref([]);

const imageList = {
  1: "1.png",
  2: "2.png",
  3: "3.png",
  4: "4.png",
  5: "5.png",
  6: "6.png",
  7: "7.png",
  8: "8.png",
  9: "9.png",
  10: "10.png",
  11: "11.png",
  12: "12.png",
  13: "13.png",
  14: "14.png",
  15: "15.png",
  16: "16.png",
  17: "17.png",
  18: "18.png",
  19: "19.png",
  20: "20.png",
  21: "21.png",
  22: "22.png",
  23: "23.png",
  24: "24.png",
  25: "25.png",
  26: "26.png",
  27: "27.png",
  28: "28.png",
  29: "29.png",
  30: "30.png",
  31: "31.png",
  32: "32.png",
  33: "33.png",
  34: "34.png",
  35: "35.png",
  36: "36.png",
  37: "37.png",
  38: "38.png",
  39: "39.png",
  40: "40.png",
  41: "41.png",
  42: "42.png",
  43: "43.png",
  44: "44.png",
  45: "45.png",
  46: "46.png",
  47: "47.png",
  48: "48.png",
  49: "49.png",
  50: "50.png",
  51: "51.png",
  52: "52.png",
  53: "53.png",
  54: "54.png",
  55: "55.png",
  56: "56.png",
  57: "57.png",
  58: "58.png",
  59: "59.png",
  60: "60.png",
  61: "61.png",
  62: "62.png",
  63: "63.png",
  64: "64.png",
  65: "65.png",
  66: "66.png",
  67: "67.png",
  68: "68.png",
  69: "69.png",
  70: "70.png",
  71: "71.png",
  72: "72.png",
  73: "73.png",
  74: "74.png",
  75: "75.png",
  76: "76.png",
  77: "77.png",
  78: "78.png",
  79: "79.png",
  80: "80.png",
  81: "81.png",
  82: "82.png",
  83: "83.png",
  84: "84.png",
  85: "85.png",
};

const preloadImages = () => {
  for (const key in imageList) {
    const img = new Image();
    img.src = `/img/${imageList[key]}`;
    preloadedImages.value.push(img);
  }
};

const tryNext = () => {
  if (index.value < Object.keys(imageList).length) {
    index.value++;
  }
};

const tryPrevious = () => {
  if (index.value > 1) {
    index.value--;
  }
};

const handleKeyPress = (event) => {
  if (event.key === "ArrowLeft") {
    tryPrevious();
  }
  if (event.key === "ArrowRight") {
    tryNext();
  }
  if (event.key === "f" || event.key === "F") {
    toggleFullscreen();
  }
};

const toggleFullscreen = () => {
  const container = document.getElementById("container");
  if (!document.fullscreenElement) {
    container.requestFullscreen().catch((err) => {
      console.log(`Error attempting to enable fullscreen: ${err.message}`);
    });
  } else {
    document.exitFullscreen();
  }
};

onMounted(() => {
  preloadImages();

  window.addEventListener("keydown", handleKeyPress);

  const container = document.getElementById("container");

  container.addEventListener("click", () => {
    return;
    if (!document.fullscreenElement) {
      container.requestFullscreen().catch((err) => {
        console.log(`Error attempting to enable fullscreen: ${err.message}`);
      });
    } else {
      document.exitFullscreen();
    }
  });
});

onUnmounted(() => {
  window.removeEventListener("keydown", handleKeyPress);
});
</script>

<style>
.next-cursor {
  cursor: url("/icons/arrow_forward.png") 0 0, auto;
}
.prev-cursor {
  cursor: url("/icons/arrow_back.png") 0 0, auto;
}
</style>
