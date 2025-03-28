<template>
  <div class="flex items-center justify-center h-screen bg-gray-900 relative">
    <!-- Loading screen -->
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

    <!-- Click to play screen -->
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

    <!-- Game content -->
    <template v-else>
      <!-- Volume and music controls in top right -->
      <div class="absolute top-4 right-4 flex items-center space-x-4">
        <div class="flex items-center">
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
const index = ref(1);
const preloadedImages = ref([]);
const preloadedAudio = ref([]);
// Initialize volume with proper browser check
const volume = ref(1); // Default to 1
// Add music mute state with localStorage persistence
const musicMuted = ref(false); // Default to not muted
// Check if code is running in browser before accessing localStorage
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
const MUSIC_VOLUME_MULTIPLIER = 0.3; // Music plays at 30% of master volume

// Loading screen variables
const loading = ref(true);
const loadingProgress = ref(0);
const loadedResources = ref(0);
const totalResources = ref(0);
const gameStarted = ref(false);

// Resources to preload
const soundList = {
  19: "pof.ogg",
  25: "pof.ogg",
  27: "pof.ogg",
  34: "pof.ogg",
  42: "pof.ogg",
  45: "pacman.mp3",
  46: "pacman.mp3",
  47: "pacman.mp3",
  63: "pof.ogg",
  76: "pof.ogg",
};

const musicList = {
  1: "main_music.mp3",
  2: "course_intro.mp3",
  3: "mario_kart_stadium.mp3",
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
};

// Toggle music on/off
const toggleMusic = () => {
  musicMuted.value = !musicMuted.value;

  // Save the music setting to localStorage
  if (process.client) {
    localStorage.setItem("musicMuted", musicMuted.value.toString());
  }

  // If turning music off, fade out current music
  if (musicMuted.value && currentMusic.value) {
    fadeOut(currentMusic.value);
    currentMusic.value = null;
  }
  // If turning music on, start playing the appropriate music for the current slide
  else if (!musicMuted.value && musicList[index.value]) {
    playMusic(musicList[index.value]);
  }
};

const preloadResources = async () => {
  const imageKeys = Object.keys(imageList);
  const soundKeys = Object.keys(soundList);
  const musicKeys = Object.keys(musicList);
  // Add extra UI images to preload
  const extraImages = ["pierre_deter.png", "perso1_decor.png", "titre_bd.png"];
  totalResources.value =
    imageKeys.length + soundKeys.length + musicKeys.length + extraImages.length;
  // Create all loading promises in parallel
  const loadPromises = [];
  // Function to create a load promise for an image
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
        resolve(null); // Resolve even on error, but with null
      };
    });
  };
  // Function to create a load promise for audio
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
          resolve(null); // Resolve even on error, but with null
        },
        { once: true }
      );
      audio.load();
    });
  };
  // Create all image load promises
  for (const key of imageKeys) {
    const promise = createImageLoadPromise(`/img/${imageList[key]}`);
    loadPromises.push(
      promise.then((img) => {
        if (img) preloadedImages.value.push(img);
      })
    );
  }
  // Create UI image load promises
  for (const imgName of extraImages) {
    const promise = createImageLoadPromise(`/img/${imgName}`);
    loadPromises.push(
      promise.then((img) => {
        if (img) preloadedImages.value.push(img);
      })
    );
  }
  // Create sound effect load promises
  for (const key of soundKeys) {
    const promise = createAudioLoadPromise(`/sounds/${soundList[key]}`);
    loadPromises.push(
      promise.then((audio) => {
        if (audio) preloadedAudio.value.push(audio);
      })
    );
  }
  // Create music track load promises
  for (const key of musicKeys) {
    const promise = createAudioLoadPromise(`/sounds/${musicList[key]}`);
    loadPromises.push(
      promise.then((audio) => {
        if (audio) preloadedAudio.value.push(audio);
      })
    );
  }
  // Wait for all resources to load in parallel
  await Promise.all(loadPromises);
  // All resources loaded
  loading.value = false;
};

const tryNext = () => {
  if (index.value < Object.keys(imageList).length) {
    index.value++;
    // Play sound if there's one associated with this image and game has started
    if (gameStarted.value && soundList[index.value]) {
      const audio = new Audio(`/sounds/${soundList[index.value]}`);
      // Apply master volume control to sounds (100% of master volume)
      audio.volume = volume.value;
      audio.play();
    }
    // Only check and update music if game has started
    if (gameStarted.value) {
      checkAndUpdateMusic();
    }
  }
};

const tryPrevious = () => {
  if (index.value > 1) {
    index.value--;
    // Only check and update music if game has started
    if (gameStarted.value) {
      checkAndUpdateMusic();
    }
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

// Music handling functions
const playMusic = (musicFile) => {
  // Don't play music if it's muted
  if (musicMuted.value) return;

  // Create a new audio element for the music
  const newMusic = new Audio(`/sounds/${musicFile}`);
  newMusic.loop = true;
  newMusic.volume = 0; // Start at 0 volume for fade in effect
  // Apply current volume setting - music plays at multiplier of master volume
  const targetVolume = volume.value * MUSIC_VOLUME_MULTIPLIER;
  // Start playing the new music immediately
  newMusic.play();
  // If there's already music playing, fade it out while fading in the new one simultaneously
  if (currentMusic.value) {
    fadeOut(currentMusic.value);
    // Start fading in the new music immediately
    fadeIn(newMusic, targetVolume);
  } else {
    // If no music was playing, just fade in the new one
    fadeIn(newMusic, targetVolume);
  }
  // Update the current music reference
  currentMusic.value = newMusic;
};

const fadeIn = (audioElement, targetVolume) => {
  let currentVol = 0;
  audioElement.volume = currentVol;
  // Calculate step size for a 2-second fade (20 steps)
  const stepSize = targetVolume / 20;
  const stepTime = 100; // 100ms per step = 2 seconds total
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
  // Calculate step size for a 2-second fade (20 steps)
  const stepSize = currentVol / 20;
  const stepTime = 100; // 100ms per step = 2 seconds total
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

// Update the volume control for all audio
const updateVolume = () => {
  if (currentMusic.value && !musicMuted.value) {
    // Music plays at 30% of master volume
    currentMusic.value.volume = volume.value * MUSIC_VOLUME_MULTIPLIER;
  }
  // Store the updated volume in localStorage with browser check
  if (process.client) {
    localStorage.setItem("masterVolume", volume.value);
  }
};

// Check if we need to change music for the current image
const checkAndUpdateMusic = () => {
  // Skip music updates if music is muted
  if (musicMuted.value) return;

  if (musicList[index.value]) {
    const newMusicFile = musicList[index.value];
    // If there's no current music playing or it's a different track, play the new music
    if (!currentMusic.value || !currentMusic.value.src.includes(newMusicFile)) {
      playMusic(newMusicFile);
    }
    // If it's the same music, no need to restart it
  }
};

// Initialize game elements after loading is complete
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
  // Only check and update music if the game has started
  if (gameStarted.value) {
    checkAndUpdateMusic();
  }
};

// Start the game
const startGame = () => {
  gameStarted.value = true;
  // Wait for DOM to update before playing music
  nextTick(() => {
    // Play the initial music for the first slide only if music is not muted
    if (!musicMuted.value && musicList[index.value]) {
      playMusic(musicList[index.value]);
    }
    // Initialize the game UI elements
    initializeGameElements();
  });
};

// Watch for loading to complete, then initialize game elements
watch(loading, (newValue) => {
  if (newValue === false) {
    // Wait for DOM to update before accessing elements
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
