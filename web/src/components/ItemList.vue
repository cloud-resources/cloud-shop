<script setup lang="ts">
// External Libraries
import { computed } from "vue"

// Stores
import { useShopStore } from "@/stores/shop"
const shopStore = useShopStore()

import { useConfigStore } from "@/stores/config"
const configStore = useConfigStore()

// Utils
import { callback } from "@/utils/callback"
import { formatPrice } from "@/utils/formatPrice"
import { getImageSrc, onImageError } from "@/utils/getImageSrc"

const filteredItems = computed(() => {
  return shopStore.items.filter(item => shopStore.selectedCategory === "all" || item.category === shopStore.selectedCategory)
})

const addToCart = (name: string): void => {
  const item = shopStore.items.find(item => item.name === name)
  if (!item) return

  const existingIndex = shopStore.cart.findIndex(cartItem => cartItem.name === item.name)

  if (existingIndex !== -1) {
    shopStore.cart[existingIndex].quantity += 1
  } else {
    shopStore.cart.push({
      label: item.label,
      name: item.name,
      price: item.price,
      quantity: 1,
      category: item.category,
    })
  }
  callback({ action: "addItem" })
}
</script>

<template>
  <transition name="list-fade" mode="out-in">
    <section class="item-list" :key="shopStore.selectedCategory">
      <div class="item" v-for="item in filteredItems" :key="item.name">
        <div class="item-image">
          <img :src="getImageSrc(item.name ?? 'fallback-item')" @error="onImageError" :alt="item.name" draggable="false" />
        </div>
        <div class="info">
          <div class="item-label">{{ item.label }}</div>
          <div class="item-price">{{ formatPrice(item.price) }}</div>
        </div>
        <div class="add-to-cart" @click="addToCart(item.name)">{{ configStore.locales.main.item.add_cart }}</div>
      </div>
    </section>
  </transition>
</template>

<style scoped>
.item-list {
  position: relative;
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  grid-auto-rows: minmax(25vh, 20vh);
  overflow: hidden;
  overflow-y: auto;
  scroll-behavior: smooth;
  width: 100%;
  height: 100%;
  padding-right: 1.5vh;
  gap: 1.75vh;

  &::-webkit-scrollbar {
    width: 0.5vh;
  }
  &::-webkit-scrollbar-thumb {
    background-color: rgba(255, 255, 255, 0.15);
    border-radius: 0.2vh;
    cursor: pointer;
  }
  &::-webkit-scrollbar-track {
    background-color: rgba(255, 255, 255, 0.05);
    border-radius: 0.2vh;
  }
}

.list-fade {
  &-enter-active,
  &-leave-active {
    transition:
      opacity 150ms ease,
      transform 150ms ease;
  }
  &-enter-from,
  &-leave-to {
    opacity: 0;
    transform: translateY(1.25vh);
  }
}

.item {
  overflow: hidden;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
  width: 100%;
  height: 100%;
  background-color: rgb(30, 30, 30);
  border-radius: 0.5vh;
  padding: 1.5vh;

  .item-image {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    background: radial-gradient(82.5% 50% at 50% 50%, rgba(255, 255, 255, 0.15) 0%, rgba(255, 255, 255, 0) 100%);
    width: 100%;
    height: 70%;

    img {
      z-index: 1;
      display: block;
      max-width: 8.5vh;
      max-height: 8.5vh;
      width: auto;
      height: auto;
      object-fit: contain;
    }
  }

  .info {
    display: flex;
    position: relative;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    font-family: "PFDinDisplayPro Medium", sans-serif;

    .item-label {
      width: 65%;
      font-size: 1.3vh;
      color: rgba(255, 255, 255, 0.8);
      margin: 0;
    }

    .item-price {
      font-size: 1.3vh;
      font-family: "PFDinDisplayPro Medium", sans-serif;
      color: rgb(102, 249, 111);
      text-shadow: 0 0 1.19vh rgba(102, 249, 111, 0.5);
      margin: 0;
    }
  }

  .add-to-cart {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 3.5vh;
    border-radius: 0.5vh;
    font-size: 1.25vh;
    font-family: "PFDinDisplayPro Medium", sans-serif;
    color: rgb(255, 255, 255);
    background-color: rgb(47, 48, 48);
    cursor: pointer;
    transition:
      250ms background-color,
      250ms box-shadow;

    &:hover {
      background-color: rgb(255, 255, 255);
      box-shadow: 0 0.4vh 2.3vh rgba(255, 255, 255, 0.35);
      color: rgb(0, 0, 0);
    }
  }
}
</style>
