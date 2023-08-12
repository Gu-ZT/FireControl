<template no-wrapping>
  <div class="back">
    <div class="fire">
      <FireControl no-wrapping/>
    </div>
  </div>
</template>

<script setup lang="ts">
import FireControl from "@/components/FireControl.vue";
</script>

<style lang="scss">
html, body {
  margin: 0;
  padding: 0;
}

.back {
  height: 100vh;
  background: linear-gradient(to right, #c7d2fe, #fecaca, #fef9c3);
  display: flex;
  justify-content: center;
  align-items: center;
}

.fire {
  width: 400px;
  height: auto;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(3vh);
}
</style>
