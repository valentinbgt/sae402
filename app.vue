<template>
  <div class="flex items-center justify-center min-h-screen bg-gray-100">
    <img :src="'/img/' + imageList[index]" />
  </div>
</template>

<script setup>
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
};

onMounted(() => {
  window.addEventListener("keydown", handleKeyPress);
});

onUnmounted(() => {
  window.removeEventListener("keydown", handleKeyPress);
});

const handleKeyPress = (event) => {
  //console.log("Key pressed:", event.key);

  if (event.key == "ArrowLeft") {
    tryNext();
  }

  if (event.key == "ArrowRight") {
    tryPrevious();
  }

  function tryNext() {
    if (index.value > 1) {
      index.value--;
    }
  }

  function tryPrevious() {
    if (index.value < Object.keys(imageList).length) {
      index.value++;
    }
  }
};
</script>
