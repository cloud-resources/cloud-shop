<script setup lang="ts">
const props = defineProps({
  locales: { type: Object, required: true },
  position: { type: String, required: true },
})
</script>

<template>
  <div
    class="header"
    :style="{
      'justify-content': props.position === 'right' ? 'flex-end' : 'flex-start',
      'align-items': props.position === 'right' ? 'flex-end' : 'flex-start',
      'text-align': props.position === 'right' ? 'right' : 'left',
    }"
  >
    <div class="title">
      <span class="title-text">{{ props.locales.title }}</span>
      <span class="title-tag">{{ props.locales.tag }}</span>
    </div>
    <p class="description">{{ props.locales.description }}</p>
  </div>
</template>

<style scoped>
.header {
  position: relative;
  display: flex;
  flex-direction: column;
  width: fit-content;
  gap: 0.5vh;
}

.title {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  gap: 1vh;
  text-transform: uppercase;

  &-text {
    font-family: "PFDinDisplayPro Bold", sans-serif;
    font-size: 2.8vh;
    letter-spacing: 0.06em;
    color: rgb(255, 255, 255);
  }

  &-tag {
    padding: 0.7vh 1.7vh;
    border-radius: 0.35vh;
    font-family: "PFDinDisplayPro Medium", sans-serif;
    font-size: 1.7vh;
    color: rgba(255, 255, 255, 0.8);
    background-color: rgb(63, 63, 63);
  }
}

.description {
  font-size: 1.4vh;
  color: rgba(255, 255, 255, 0.5);
}
</style>
