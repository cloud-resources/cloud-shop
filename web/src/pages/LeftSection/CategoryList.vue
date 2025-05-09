<script setup lang="ts">
// External Libraries
import { ref, nextTick, onMounted } from "vue"
import { Icon } from "@iconify/vue"

// Stores
import { useShopStore } from "@/stores/shop"
const shopStore = useShopStore()

// Composables
import { useShopEvents } from "@/composables/useShopEvents"
const { getCategories } = useShopEvents()

// Utils
import { fetchData } from "@/utils/api"

const selectCategory = (category: string): void => {
  shopStore.selectedCategory = category
  scrollIntoView()
  fetchData({ label: "selectCategory" })
}

const categorySlideRef = ref<HTMLElement | null>(null)
const scrollIntoView = (): void => {
  nextTick(() => {
    const container = categorySlideRef.value
    if (!container) return

    const selectedCategoryElement = container.querySelector(".selected") as HTMLElement | null
    const lastIndex = shopStore.categories.length - 2

    if (shopStore.selectedCategory === "all") {
      container.scrollTo({ left: 0, behavior: "smooth" })
    } else if (selectedCategoryElement) {
      const scrollOptions: ScrollIntoViewOptions = {
        behavior: "smooth",
        block: "nearest",
        inline: "center",
      }

      selectedCategoryElement.scrollIntoView(scrollOptions)

      if (shopStore.selectedCategory === shopStore.categories[lastIndex]?.type) {
        container.scrollTo({ left: container.scrollWidth, behavior: "smooth" })
      }
    }
  })
}

onMounted(async () => {
  await getCategories()
})
</script>

<template>
  <section class="categories">
    <div class="category-slide" ref="categorySlideRef">
      <div
        v-for="(category, index) in shopStore.categories"
        :key="index"
        class="category"
        @click="selectCategory(category.type)"
        :class="{ selected: shopStore.selectedCategory === category.type }"
      >
        <div class="icon-holder">
          <Icon :icon="category.icon" />
        </div>
        <div class="name-holder">
          <p class="category-name">{{ category.name }}</p>
        </div>
      </div>
    </div>
  </section>
</template>

<style lang="scss">
.categories {
  display: flex;
  overflow: hidden;
  gap: 1vh;
  width: 100%;
  margin-top: 1vh;

  .category-slide {
    display: flex;
    overflow: auto;
    overflow-x: auto;
    white-space: nowrap;
    scroll-behavior: smooth;
    width: fit-content;
    max-width: 100%;
    gap: 1.9vh;
    padding: 1vh 1vh 1.75vh 0;

    .category {
      flex-shrink: 0;
      display: flex;
      justify-content: flex-start;
      align-items: center;
      gap: 0.75vh;
      align-items: stretch;
      width: fit-content;
      height: fit-content;
      font-size: 0.85rem;
      color: rgba(255, 255, 255, 0.8);
      transition: 0.25s background-color;
      cursor: pointer;

      &:hover,
      &.selected {
        .icon-holder,
        .name-holder {
          background-color: rgb(255, 255, 255);
          color: rgb(0, 0, 0);

          svg {
            color: rgb(0, 0, 0);
          }
        }
      }

      .icon-holder {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 5.5vh;
        height: auto;
        border-radius: 0.5vh;
        background-color: rgb(30, 30, 30);
        transition: 0.25s background-color;

        svg {
          width: 45%;
          height: auto;
          color: rgb(255, 255, 255);
        }
      }

      .name-holder {
        text-align: center;
        background-color: rgb(30, 30, 30);
        padding: 2vh 2.75vh;
        border-radius: 0.5vh;
        transition: 0.25s background-color;

        .category-name {
          font-family: "PFDinDisplayPro-Bold", sans-serif;
        }
      }
    }

    &::-webkit-scrollbar-thumb,
    &::-webkit-scrollbar-track {
      background-color: rgba(255, 255, 255, 0.05);
      border-radius: 0.2vh;
    }

    &::-webkit-scrollbar-thumb {
      cursor: pointer;
    }

    &::-webkit-scrollbar {
      height: 0.5vh;
    }
  }
}
</style>
