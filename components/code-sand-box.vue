<template>
  <button
    class="abs-tr m-2 rounded-md text-white text-sm px-2 py-1 bg-gradient-to-r from-purple-400 via-pink-500 to-red-500"
    @click="toggleExample"
  >
    例子
  </button>
  <transition name="fade">
    <div class="flex flex-col fixed inset-0 bg-white" v-show="!hidden">
      <span
        @click="toggleExample"
        class="flex bg-red-500 justify-center items-center rounded-full w-3 h-3 m-1"
      />
      <div class="flex-1">
        <slot></slot>
      </div>
    </div>
  </transition>
</template>

<script>
import { ref } from "vue";
export default {
  setup() {
    const hidden = ref(true);
    return {
      hidden,
    };
  },
  methods: {
    toggleExample() {
      this.hidden = !this.hidden;
    },
  },
};
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
