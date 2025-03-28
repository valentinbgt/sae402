<template>
  <div class="flex items-center justify-center h-screen bg-gray-900 relative">
    <div
      v-if="loading"
      class="fixed inset-0 bg-gray-900 flex flex-col items-center justify-center z-50"
    >
      <h1 class="text-4xl text-white mb-6 font-bold">
        Chargement de l'aventure
      </h1>
      <div class="w-64 h-4 bg-gray-700 rounded-full overflow-hidden">
        <div
          class="h-full bg-blue-500 transition-all duration-300"
          :style="{ width: `${loadingProgress}%` }"
        ></div>
      </div>
      <p class="text-white mt-4">
        {{ Math.round(loadingProgress) }}% ({{ loadedResources }}/{{
          totalResources
        }})
      </p>
    </div>

    <div
      v-else-if="!gameStarted"
      class="fixed inset-0 bg-gray-900 flex flex-col items-center justify-center z-50 cursor-pointer"
      @click="startGame"
    >
      <h1 class="text-4xl text-white mb-6 font-bold">
        Clique sur l'écran pour rejoindre l'aventure
      </h1>
      <div class="text-white text-opacity-70 text-xl mt-4 animate-pulse">
        Cliquez n'importe où pour commencer
      </div>
    </div>

    <template v-else>
      <div class="absolute top-4 right-4 flex items-center space-x-4">
        <div class="items-center hidden md:flex">
          <span class="text-white mr-2">Volume</span>
          <input
            type="range"
            min="0"
            max="1"
            step="0.01"
            v-model="volume"
            @input="updateVolume"
            class="w-32"
          />
        </div>
        <button
          @click="toggleMusic"
          class="px-3 py-1 bg-gray-700 hover:bg-gray-600 rounded text-white"
        >
          Musique: {{ musicMuted ? "OFF" : "ON" }}
        </button>
      </div>

      <img
        src="/img/pierre_deter.png"
        class="absolute bottom-0 left-0 w-60 hidden lg:block"
        alt=""
      />
      <img
        src="/img/perso1_decor.png"
        class="absolute bottom-0 right-0 w-60 md:w-80"
        alt=""
      />
      <div
        class="absolute bottom-4 left-0 right-0 justify-center hidden min-[900px]:flex"
      >
        <img src="/img/titre_bd.png" class="w-96" alt="" />
      </div>
      <div
        id="container"
        class="h-3/5 aspect-square flex items-center justify-center relative"
        style="max-width: 100vw"
      >
        <img
          id="mainImage"
          :src="'/img/' + imageList[index]"
          class="h-full w-full object-contain"
        />
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
    </template>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, nextTick } from "vue";

useHead({
  title: "Michel et l'anneau",
});

const index = ref(1);
const preloadedImages = ref([]);
const preloadedAudio = ref([]);

const volume = ref(1);

const musicMuted = ref(false);

if (process.client) {
  const savedVolume = localStorage.getItem("masterVolume");
  if (savedVolume !== null) {
    volume.value = parseFloat(savedVolume);
  }

  const savedMusicState = localStorage.getItem("musicMuted");
  if (savedMusicState !== null) {
    musicMuted.value = savedMusicState === "true";
  }
}
const currentMusic = ref(null);
const MUSIC_VOLUME_MULTIPLIER = 0.6;

const loading = ref(true);
const loadingProgress = ref(0);
const loadedResources = ref(0);
const totalResources = ref(0);
const gameStarted = ref(false);

const soundList = {
  2: "piouf.ogg",
  5: "coup_de_poing.ogg",
  14: "corbeau.ogg",
  18: "pof.ogg",
  24: "pof.ogg",
  26: "pof.ogg",
  27: "ki.ogg",
  28: "ka.ogg",
  29: "me.ogg",
  30: "ha.ogg",
  31: "me2.ogg",
  32: "haaa.ogg",
  33: "pof.ogg",
  34: "boss_spawn.ogg",
  42: "pof.ogg",
  45: "pacman.ogg",
  46: "pacman.ogg",
  47: "pacman.ogg",
  49: "criquets.ogg",
  53: "glurps.ogg",
  62: "crounch.ogg",
  63: "pof.ogg",
  67: "boss_spawn.ogg",
  74: "transfo1.ogg",
  75: "transfo2.ogg",
  76: "transfo3.ogg",
  77: "pof.ogg",
  83: "hum.ogg",
  88: "wilhelm.ogg",
  r_key: "rot.ogg",
};

const musicList = {
  1: "main_music.mp3",
  33: "main_music.mp3",
  34: "STOP",
  35: "herobrine.mp3",
  66: "herobrine.mp3",
  67: "STOP",
  68: "dernier_boss.mp3",
  82: "dernier_boss.mp3",
  83: "fin.mp3",
};

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
  86: "86.png",
  87: "87.png",
  88: "88.png",
  89: "89.png",
  90: "90.png",
  91: "91.png",
  92: "92.png",
  93: "93.png",
  94: "94.png",
  95: "95.png",
  96: "96.png",
};

const toggleMusic = () => {
  musicMuted.value = !musicMuted.value;

  if (process.client) {
    localStorage.setItem("musicMuted", musicMuted.value.toString());
  }

  if (musicMuted.value && currentMusic.value) {
    fadeOut(currentMusic.value);
    currentMusic.value = null;
  } else if (!musicMuted.value && musicList[index.value]) {
    playMusic(musicList[index.value]);
  }
};

const preloadResources = async () => {
  const imageKeys = Object.keys(imageList);
  const soundKeys = Object.keys(soundList);
  const musicKeys = Object.keys(musicList);
  const extraImages = ["pierre_deter.png", "perso1_decor.png", "titre_bd.png"];
  totalResources.value =
    imageKeys.length + soundKeys.length + musicKeys.length + extraImages.length;
  const loadPromises = [];
  const createImageLoadPromise = (src) => {
    return new Promise((resolve) => {
      const img = new Image();
      img.src = src;
      img.onload = () => {
        loadedResources.value++;
        loadingProgress.value =
          (loadedResources.value / totalResources.value) * 100;
        resolve(img);
      };
      img.onerror = () => {
        loadedResources.value++;
        loadingProgress.value =
          (loadedResources.value / totalResources.value) * 100;
        resolve(null);
      };
    });
  };
  const createAudioLoadPromise = (src) => {
    return new Promise((resolve) => {
      const audio = new Audio(src);
      const onCanPlay = () => {
        loadedResources.value++;
        loadingProgress.value =
          (loadedResources.value / totalResources.value) * 100;
        resolve(audio);
      };
      audio.addEventListener("canplaythrough", onCanPlay, { once: true });
      audio.addEventListener(
        "error",
        () => {
          loadedResources.value++;
          loadingProgress.value =
            (loadedResources.value / totalResources.value) * 100;
          resolve(null);
        },
        { once: true }
      );
      audio.load();
    });
  };
  for (const key of imageKeys) {
    const promise = createImageLoadPromise(`/img/${imageList[key]}`);
    loadPromises.push(
      promise.then((img) => {
        if (img) preloadedImages.value.push(img);
      })
    );
  }
  for (const imgName of extraImages) {
    const promise = createImageLoadPromise(`/img/${imgName}`);
    loadPromises.push(
      promise.then((img) => {
        if (img) preloadedImages.value.push(img);
      })
    );
  }
  for (const key of soundKeys) {
    const promise = createAudioLoadPromise(`/sounds/${soundList[key]}`);
    loadPromises.push(
      promise.then((audio) => {
        if (audio) preloadedAudio.value.push(audio);
      })
    );
  }
  for (const key of musicKeys) {
    const promise = createAudioLoadPromise(`/sounds/${musicList[key]}`);
    loadPromises.push(
      promise.then((audio) => {
        if (audio) preloadedAudio.value.push(audio);
      })
    );
  }
  await Promise.all(loadPromises);
  loading.value = false;
};

const tryNext = () => {
  if (index.value < Object.keys(imageList).length) {
    index.value++;
    if (gameStarted.value && soundList[index.value]) {
      const audio = new Audio(`/sounds/${soundList[index.value]}`);
      audio.volume = volume.value;
      audio.play();
    }
    if (gameStarted.value) {
      checkAndUpdateMusic();
    }
  }
};

const tryPrevious = () => {
  if (index.value > 1) {
    index.value--;
    if (gameStarted.value) {
      checkAndUpdateMusic();
    }
  }
};

const handleKeyPress = (event) => {
  if (!gameStarted.value) return;

  if (event.key === "ArrowLeft") {
    tryPrevious();
  }
  if (event.key === "ArrowRight") {
    tryNext();
  }
  if (event.key === "f" || event.key === "F") {
    toggleFullscreen();
  }
  if (event.key === "r" || event.key === "R") {
    if (gameStarted.value) {
      const audio = new Audio(`/sounds/${soundList["r_key"]}`);
      audio.volume = volume.value;
      audio.play();
    }
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

const playMusic = (musicFile) => {
  if (musicMuted.value) return;

  const newMusic = new Audio(`/sounds/${musicFile}`);
  newMusic.loop = true;
  newMusic.volume = 0;
  const targetVolume = volume.value * MUSIC_VOLUME_MULTIPLIER;
  newMusic.play();
  if (currentMusic.value) {
    fadeOut(currentMusic.value);
    fadeIn(newMusic, targetVolume);
  } else {
    fadeIn(newMusic, targetVolume);
  }
  currentMusic.value = newMusic;
};

const fadeIn = (audioElement, targetVolume) => {
  let currentVol = 0;
  audioElement.volume = currentVol;
  const stepSize = targetVolume / 10;
  const stepTime = 100;
  const fadeInterval = setInterval(() => {
    currentVol += stepSize;
    if (currentVol >= targetVolume) {
      currentVol = targetVolume;
      clearInterval(fadeInterval);
    }
    audioElement.volume = currentVol;
  }, stepTime);
};

const fadeOut = (audioElement) => {
  let currentVol = audioElement.volume;
  const stepSize = currentVol / 10;
  const stepTime = 100;
  return new Promise((resolve) => {
    const fadeInterval = setInterval(() => {
      currentVol -= stepSize;
      if (currentVol <= 0) {
        currentVol = 0;
        clearInterval(fadeInterval);
        audioElement.pause();
        audioElement.currentTime = 0;
        resolve();
      }
      audioElement.volume = currentVol;
    }, stepTime);
  });
};

const updateVolume = () => {
  if (currentMusic.value && !musicMuted.value) {
    currentMusic.value.volume = volume.value * MUSIC_VOLUME_MULTIPLIER;
  }
  if (process.client) {
    localStorage.setItem("masterVolume", volume.value);
  }
};

const checkAndUpdateMusic = () => {
  if (musicMuted.value) return;

  if (musicList[index.value]) {
    const newMusicFile = musicList[index.value];
    if (!currentMusic.value || !currentMusic.value.src.includes(newMusicFile)) {
      playMusic(newMusicFile);
    }
  }
};

const initializeGameElements = () => {
  const container = document.getElementById("container");
  if (container) {
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
  }
  if (gameStarted.value) {
    checkAndUpdateMusic();
  }
};

const startGame = () => {
  gameStarted.value = true;
  nextTick(() => {
    if (!musicMuted.value && musicList[index.value]) {
      playMusic(musicList[index.value]);
    }
    initializeGameElements();
  });
};

watch(loading, (newValue) => {
  if (newValue === false) {
    nextTick(() => {
      initializeGameElements();
    });
  }
});

onMounted(() => {
  preloadResources();
  window.addEventListener("keydown", handleKeyPress);
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

<style>
.next-cursor {
  cursor: url("/icons/arrow_forward.png") 0 0, auto;
}
.prev-cursor {
  cursor: url("/icons/arrow_back.png") 0 0, auto;
}
</style>
