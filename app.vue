<template>
  <div class="flex items-center justify-center h-screen bg-gray-700">
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
