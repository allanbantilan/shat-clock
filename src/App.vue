<template>
  <div
    class="min-h-screen bg-gradient-to-tr from-indigo-50 via-white to-indigo-50 flex items-center justify-center p-6 font-sans"
  >
    <div class="max-w-md w-full p-8 bg-white rounded-3xl shadow-2xl space-y-8">
      <!-- Header -->
      <div class="text-center">
        <h1 class="text-4xl font-extrabold text-indigo-700 mb-1 tracking-wide">Shat Clock</h1>
        <p class="text-indigo-500 font-medium">BAWAL MAHIHINA</p>
      </div>

      <!-- Time Selection -->
      <div class="flex justify-center space-x-6">
        <button
          v-for="option in timeOptions"
          :key="option.time"
          @click="selectTime(option.time)"
          :class="[
            'px-6 py-3 rounded-full font-semibold transition-all duration-300 shadow-md select-none text-center',
            selectedTime === option.time
              ? 'bg-indigo-600 text-white shadow-indigo-400/60 scale-105'
              : 'bg-indigo-100 text-indigo-600 hover:bg-indigo-200 focus:outline-none focus:ring-4 focus:ring-indigo-300',
          ]"
        >
          <div class="text-lg font-bold">{{ option.time }}s</div>
          <div class="text-xs">{{ option.label }}</div>
        </button>
      </div>

      <!-- Timer Display -->
      <div class="relative flex flex-col items-center">
        <svg class="w-40 h-40" viewBox="0 0 120 120">
          <!-- Background Circle -->
          <circle
            cx="60"
            cy="60"
            r="54"
            stroke="#e0e7ff"
            stroke-width="12"
            fill="none"
            filter="url(#blur)"
          />
          <!-- Progress Circle -->
          <circle
            cx="60"
            cy="60"
            r="54"
            fill="none"
            :stroke="currentTime <= 5 ? '#ef4444' : '#6366f1'"
            stroke-width="12"
            stroke-linecap="round"
            :stroke-dasharray="339.292"
            :stroke-dashoffset="339.292 - (progressPercentage / 100) * 339.292"
            class="transition-stroke duration-1000 ease-linear"
            style="transform-origin: 50% 50%; transform: rotate(-90deg)"
          />
          <defs>
            <filter
              id="blur"
              x="-20%"
              y="-20%"
              width="140%"
              height="140%"
              color-interpolation-filters="sRGB"
            >
              <feDropShadow
                dx="0"
                dy="0"
                stdDeviation="3"
                flood-color="#a5b4fc"
                flood-opacity="0.4"
              />
            </filter>
          </defs>
        </svg>

        <div
          class="absolute inset-0 flex flex-col items-center justify-center pointer-events-none select-none mb-8"
        >
          <span
            :class="[
              'text-3xl font-bold tracking-tight',
              currentTime <= 5 ? 'text-red-500' : 'text-indigo-700',
            ]"
          >
            {{ currentTime }}
          </span>
          <span class="mt-2 text-indigo-400 font-semibold uppercase tracking-wide text-sm"
            >seconds</span
          >
        </div>

        <div class="mt-4 text-indigo-500 font-medium text-sm">
          {{ selectedTime }} second shot clock
        </div>
      </div>

      <!-- Controls -->
      <div class="flex justify-between space-x-6">
        <button
          @click="isRunning ? stop() : start()"
          :disabled="currentTime <= 0"
          :class="[
            'flex-1 py-3 rounded-full font-semibold shadow-lg transition-all duration-300 focus:outline-none focus:ring-4',
            isRunning
              ? 'bg-red-500 hover:bg-red-600 focus:ring-red-400 text-white'
              : 'bg-indigo-600 hover:bg-indigo-700 focus:ring-indigo-400 text-white',
            currentTime <= 0 ? 'opacity-50 cursor-not-allowed' : '',
          ]"
        >
          {{ isRunning ? "Stop" : "Start" }}
        </button>

        <!-- Reset + Auto Reset stacked -->
        <div class="flex flex-col space-y-4 flex-1">
          <button
            @click="reset"
            class="py-3 rounded-full font-semibold bg-red-300 hover:bg-red-400 focus:outline-none focus:ring-4 focus:ring-gray-300 text-red-700 shadow-lg transition-all duration-300"
          >
            Reset
          </button>
          <button
            @click="startAutoReset"
            class="w-full py-3 rounded-full font-semibold bg-blue-300 hover:bg-blue-400 focus:outline-none focus:ring-4 focus:ring-gray-300 text-blue-700 shadow-lg transition-all duration-300 relative"
          >
            Consequence Mode
            <span
              class="absolute top-2 right-4 w-3 h-3 rounded-full"
              :class="autoResetEnabled ? 'bg-emerald-600' : 'bg-red-500'"
            ></span>
          </button>
        </div>
      </div>

      <!-- Status -->
      <div class="text-center mt-4">
        <span
          :class="[
            'inline-flex items-center justify-center px-4 py-1 rounded-full text-sm font-semibold tracking-wider shadow-sm select-none',
            isRunning
              ? 'bg-green-600 text-white'
              : currentTime <= 0
              ? 'bg-red-100 text-red-800'
              : 'bg-gray-100 text-gray-800',
          ]"
        >
          <span
            :class="[
              'w-3 h-3 rounded-full mr-2 inline-block',
              isRunning ? 'bg-green-500' : currentTime <= 0 ? 'bg-red-500' : 'bg-gray-400',
            ]"
          ></span>
          {{ isRunning ? "Running" : currentTime <= 0 ? "Time Up!" : "Stopped" }}
        </span>
      </div>
     <p class="text-center text-xs text-gray-500 mt-2">Created by: Allan Bantilan</p>
    </div>

    <!-- Confirmation Dialog -->
    <div
      v-if="showConfirmDialog"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-6 z-50"
    >
      <div class="bg-white rounded-2xl p-6 max-w-sm w-full shadow-2xl">
        <div class="text-center">
          <div class="mb-4">
            <div
              class="w-16 h-16 mx-auto bg-orange-100 rounded-full flex items-center justify-center"
            >
              <svg
                class="w-8 h-8 text-orange-600"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L3.732 16.5c-.77.833.192 2.5 1.732 2.5z"
                ></path>
              </svg>
            </div>
          </div>
          <h3 class="text-xl font-bold text-gray-900 mb-2">Shot Confirmation</h3>
          <p class="text-gray-600 mb-6">
            Did the participant drink their shot within the time limit?
          </p>
          <div class="flex space-x-3">
            <button
              @click="handleConfirmation(true)"
              class="flex-1 bg-green-600 hover:bg-green-700 text-white font-semibold py-3 px-4 rounded-xl transition-colors duration-200"
            >
              Yes
            </button>
            <button
              @click="handleConfirmation(false)"
              class="flex-1 bg-red-600 hover:bg-red-700 text-white font-semibold py-3 px-4 rounded-xl transition-colors duration-200"
            >
              No
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Roulette Wheel -->
    <div
      v-if="showRoulette"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50"
    >
      <div class="bg-white rounded-2xl p-4 sm:p-6 w-full max-w-sm sm:max-w-lg shadow-2xl">
        <div class="text-center">
          <h3 class="text-xl sm:text-2xl font-bold text-gray-900 mb-3 sm:mb-4">
            Consequence Roulette
          </h3>
          <p class="text-sm sm:text-base text-gray-600 mb-4 sm:mb-6">
            Time to face the consequences! Spin the wheel!
          </p>

          <!-- Roulette Wheel -->
          <div class="relative w-64 h-64 sm:w-80 sm:h-80 mx-auto mb-4 sm:mb-6">
            <!-- Pointer -->
            <div class="absolute top-[-12px] left-1/2 transform -translate-x-1/2 z-20 rotate-180">
              <div
                class="w-0 h-0 border-l-[10px] border-r-[10px] border-b-[20px] border-l-transparent border-r-transparent border-b-red-600"
              ></div>
            </div>

            <!-- Wheel -->
            <svg
              class="w-full h-full"
              :style="{
                transform: `rotate(${rouletteRotation}deg)`,
                transition: isSpinning ? 'transform 3s cubic-bezier(0.4, 0, 0.2, 1)' : 'none',
              }"
              viewBox="0 0 200 200"
            >
              <!-- Wheel segments -->
              <g v-for="(consequence, index) in consequences" :key="index">
                <path
                  :d="getSegmentPath(index)"
                  :fill="getSegmentColor(index)"
                  stroke="white"
                  stroke-width="2"
                />
                <text
                  :x="getTextX(index)"
                  :y="getTextY(index)"
                  :transform="getTextTransform(index)"
                  class="text-xs font-semibold fill-white text-center"
                  text-anchor="middle"
                  dominant-baseline="middle"
                >
                  {{ consequence }}
                </text>
              </g>

              <!-- Center circle -->
              <circle cx="100" cy="100" r="15" fill="#374151" />
              <circle cx="100" cy="100" r="8" fill="#6b7280" />
            </svg>
          </div>

          <!-- Result display -->
          <div v-if="selectedConsequence && !isSpinning" class="mb-4 sm:mb-6">
            <div class="bg-red-100 border border-red-300 rounded-lg p-3 sm:p-4">
              <h4 class="text-base sm:text-lg font-bold text-red-800 mb-2">Your Consequence:</h4>
              <p class="text-red-700 font-semibold text-lg sm:text-xl">{{ selectedConsequence }}</p>
            </div>
          </div>

          <!-- Action buttons -->
          <div class="flex flex-col sm:flex-row space-y-2 sm:space-y-0 sm:space-x-3">
            <button
              v-if="!selectedConsequence"
              @click="spinRoulette"
              :disabled="isSpinning"
              class="w-full bg-gradient-to-r from-red-500 to-pink-500 hover:from-red-600 hover:to-pink-600 disabled:opacity-50 disabled:cursor-not-allowed text-white font-bold py-3 px-4 sm:px-6 rounded-xl transition-all duration-200 shadow-lg text-sm sm:text-base"
            >
              {{ isSpinning ? "Spinning..." : "SPIN THE WHEEL!" }}
            </button>

            <template v-if="selectedConsequence && !isSpinning">
              <button
                v-if="!hasUsedSpin"
                @click="spinAgain"
                class="w-full bg-gradient-to-r from-yellow-500 to-orange-500 hover:from-yellow-600 hover:to-orange-600 text-white font-bold py-3 px-4 sm:px-6 rounded-xl transition-all duration-200 shadow-lg text-sm sm:text-base"
              >
                SPIN AGAIN (1 Try Left)
              </button>
              <button
                @click="closeRoulette"
                class="w-full bg-gray-600 hover:bg-gray-700 text-white font-semibold py-3 px-4 rounded-xl transition-colors duration-200 text-sm sm:text-base"
              >
                Accept Consequence
              </button>
            </template>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onUnmounted } from "vue";

export default {
  setup() {
    const selectedTime = ref(24);
    const currentTime = ref(24);
    const isRunning = ref(false);
    const intervalId = ref(null);

    const autoResetEnabled = ref(false);
    const autoResetInterval = ref(null);
    const showConfirmDialog = ref(false);
    const showRoulette = ref(false);
    const isSpinning = ref(false);
    const rouletteRotation = ref(0);
    const selectedConsequence = ref("");
    const hasUsedSpin = ref(false); 

    const timeOptions = [
      { time: 24, label: "Pro" },
      { time: 50, label: "Jerico" },
      { time: 100, label: "Palugi" },
    ];

    const consequences = ["Take 2 shots instead"];

    const progressPercentage = computed(() => {
      return ((selectedTime.value - currentTime.value) / selectedTime.value) * 100;
    });

    const selectTime = (time) => {
      selectedTime.value = time;
      currentTime.value = time;
      stop();
    };

    const start = () => {
      if (currentTime.value <= 0) return;

      isRunning.value = true;
      intervalId.value = setInterval(() => {
        if (currentTime.value > 0) {
          currentTime.value--;
        } else {
          stop();
    
          if (autoResetEnabled.value) {
            setTimeout(() => {
              showConfirmDialog.value = true;
            }, 1000); 
          }
        }
      }, 1000);
    };

    const stop = () => {
      isRunning.value = false;
      if (intervalId.value) {
        clearInterval(intervalId.value);
        intervalId.value = null;
      }
    };

    const reset = () => {
      stop();
      currentTime.value = selectedTime.value;
    };

    const handleConfirmation = (drankShot) => {
      showConfirmDialog.value = false;

      if (drankShot) {

        reset();
        start();
      } else {
        showRoulette.value = true;
        selectedConsequence.value = ""; 
        hasUsedSpin.value = false;
      }
    };

    const spinRoulette = () => {
      if (isSpinning.value) return;

      isSpinning.value = true;
      selectedConsequence.value = ""; 
      const spins = 5 + Math.random() * 5; 
      const finalRotation = spins * 360 + Math.random() * 360;
      rouletteRotation.value += finalRotation;

      setTimeout(() => {
        isSpinning.value = false;
        const normalizedRotation = rouletteRotation.value % 360;
        const segmentSize = 360 / consequences.length;
        const selectedIndex =
          Math.floor((360 - normalizedRotation) / segmentSize) % consequences.length;
        selectedConsequence.value = consequences[selectedIndex];
      }, 3000); 
    };

    const spinAgain = () => {
      hasUsedSpin.value = true; 
      selectedConsequence.value = null; 
      spinRoulette(); 
    };

    const closeRoulette = () => {
      showRoulette.value = false;
      selectedConsequence.value = "";
      hasUsedSpin.value = false; 
      reset(); 
    };

    // Roulette helper functions
    const getSegmentPath = (index) => {
      const segmentAngle = 360 / consequences.length;
      const startAngle = index * segmentAngle;
      const endAngle = (index + 1) * segmentAngle;

      const startAngleRad = (startAngle * Math.PI) / 180;
      const endAngleRad = (endAngle * Math.PI) / 180;

      const x1 = 100 + 85 * Math.cos(startAngleRad);
      const y1 = 100 + 85 * Math.sin(startAngleRad);
      const x2 = 100 + 85 * Math.cos(endAngleRad);
      const y2 = 100 + 85 * Math.sin(endAngleRad);

      const largeArcFlag = segmentAngle > 180 ? 1 : 0;

      return `M 100 100 L ${x1} ${y1} A 85 85 0 ${largeArcFlag} 1 ${x2} ${y2} Z`;
    };

    const getSegmentColor = (index) => {
      const colors = [
        "#ef4444",
        "#f97316",
        "#eab308",
        "#22c55e",
        "#06b6d4",
        "#3b82f6",
        "#8b5cf6",
        "#ec4899",
      ];
      return colors[index % colors.length];
    };

    const getTextX = (index) => {
      const segmentAngle = 360 / consequences.length;
      const angle = ((index * segmentAngle + segmentAngle / 2) * Math.PI) / 180;
      return 100 + 60 * Math.cos(angle);
    };

    const getTextY = (index) => {
      const segmentAngle = 360 / consequences.length;
      const angle = ((index * segmentAngle + segmentAngle / 2) * Math.PI) / 180;
      return 100 + 60 * Math.sin(angle);
    };

    const getTextTransform = (index) => {
      const segmentAngle = 360 / consequences.length;
      const angle = index * segmentAngle + segmentAngle / 2;
      const textX = getTextX(index);
      const textY = getTextY(index);

      if (angle > 90 && angle < 270) {
        return `rotate(${angle + 180} ${textX} ${textY})`;
      }
      return `rotate(${angle} ${textX} ${textY})`;
    };

    let autoResetCheckInterval = null;

    const startAutoReset = () => {
      if (autoResetEnabled.value) {
        autoResetEnabled.value = false;
        clearInterval(autoResetCheckInterval);
        autoResetCheckInterval = null;
        return;
      }

      autoResetEnabled.value = true;
    };

    // Cleanup on component unmount
    onUnmounted(() => {
      if (intervalId.value) {
        clearInterval(intervalId.value);
      }
      if (autoResetCheckInterval) {
        clearInterval(autoResetCheckInterval);
      }
    });

    return {
      selectedTime,
      currentTime,
      isRunning,
      timeOptions,
      progressPercentage,
      selectTime,
      start,
      stop,
      reset,
      startAutoReset,
      autoResetEnabled,
      showConfirmDialog,
      handleConfirmation,
      showRoulette,
      isSpinning,
      rouletteRotation,
      selectedConsequence,
      hasUsedSpin,
      consequences,
      spinRoulette,
      spinAgain,
      closeRoulette,
      getSegmentPath,
      getSegmentColor,
      getTextX,
      getTextY,
      getTextTransform,
    };
  },
};
</script>

<style scoped>
.transition-stroke {
  transition-property: stroke-dashoffset;
}
</style>
