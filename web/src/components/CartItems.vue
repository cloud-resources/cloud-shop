<script setup lang="ts">
// External Libraries
import { Icon } from "@iconify/vue"

// Stores
import { useShopStore } from "@/stores/shop"
const shopStore = useShopStore()

// Utils
import { callback } from "@/utils/callback"
import { formatPrice } from "@/utils/formatPrice"
import { getImageSrc, onImageError } from "@/utils/getImageSrc"

// Modify Quantity

const decrementQuantity = (index: number): void => {
  const item = shopStore.cart[index]
  if (!item || item.quantity <= 0) return

  item.quantity--
  if (item.quantity === 0) {
    removeItem(index)
    return
  }
  callback({ action: "updateQuantity" })
}

const incrementQuantity = (index: number): void => {
  const item = shopStore.cart[index]
  if (!item || item.quantity >= 999) return

  item.quantity++
  callback({ action: "updateQuantity" })
}

const removeItem = (index: number): void => {
  shopStore.cart.splice(index, 1)
  callback({ action: "removeItem" })
}

// Validate Input

const validateNumberInput = (event: KeyboardEvent): void => {
  const key = event.key
  const allowedKeys = ["Backspace", "Tab", "ArrowLeft", "ArrowUp", "ArrowRight", "ArrowDown", "Delete"]
  const isCtrlA = event.ctrlKey && key === "a"
  const isNumber = /^[0-9]$/.test(key)

  if (isNumber || allowedKeys.includes(key) || isCtrlA) return
  event.preventDefault()
}

const validateQuantityInput = (index: number, event?: KeyboardEvent): void => {
  const item = shopStore.cart[index]
  if (!item) return

  if (item.quantity === 0) {
    shopStore.cart.splice(index, 1)
  } else if (item.quantity <= 0) {
    item.quantity = 1
  } else if (item.quantity >= 999) {
    item.quantity = 999
  }

  if (event) {
    ;(event.target as HTMLElement).blur()
  }
}
</script>

<template>
  <div v-if="!shopStore.cart.length" class="no-items">
    <Icon icon="mdi:shopping-basket-off" />
  </div>

  <section class="cart-items">
    <transition-group name="item-move">
      <div class="item" v-for="(item, index) in shopStore.cart" :key="item.name">
        <div class="item-info">
          <img class="item-image" :src="getImageSrc(item.name ?? 'fallback-item')" @error="onImageError" :alt="item.name" draggable="false" />
          <div class="text-container">
            <span class="item-label">{{ item.label }}</span>
            <span class="item-price">{{ formatPrice(item.price * item.quantity) }}</span>
          </div>
        </div>

        <div class="button-container">
          <button class="decrement" @click="decrementQuantity(index)">
            <Icon class="icon" icon="typcn:minus" width="45%" />
          </button>

          <input
            class="counter"
            type="text"
            minlength="1"
            maxlength="3"
            :placeholder="String(item.quantity)"
            v-model.number="item.quantity"
            @keydown="validateNumberInput"
            @blur="validateQuantityInput(index)"
            @keyup.enter="validateQuantityInput(index, $event)"
          />

          <button class="increment" @click="incrementQuantity(index)">
            <Icon class="icon" icon="typcn:plus" width="45%" />
          </button>

          <button class="remove" @click="removeItem(index)">
            <Icon class="icon" icon="mdi:trash-can" width="50%" />
          </button>
        </div>
      </div>
    </transition-group>
  </section>
</template>

<style scoped>
.cart-items {
  overflow: overlay;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: flex-end;
  overflow-y: auto;
  scrollbar-gutter: stable;
  scroll-behavior: smooth;
  width: calc(100% + 2.2vh);
  height: 100%;
  max-height: calc((7 * 6.5vh) + (6 * 1vh));
  padding-right: 1.5vh;
  gap: 1vh;

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

.item {
  flex-shrink: 0;
  position: relative;
  width: 100%;
  height: 6.5vh;
  background-color: rgb(30, 30, 30);
  border-radius: 0.5vh;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5vh;
  will-change: transform, opacity;

  .item-info {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
    gap: 1.5vh;

    .item-image {
      z-index: 1;
      display: block;
      max-width: 4.75vh;
      max-height: 4.75vh;
      width: auto;
      height: auto;
      object-fit: contain;
    }

    .text-container {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: flex-start;
      gap: 0.5vh;
      font-family: "PFDinDisplayPro Medium", sans-serif;

      .item-label {
        text-align: left;
        font-size: 1.25vh;
        color: rgba(255, 255, 255, 0.8);
      }

      .item-price {
        font-size: 1.25vh;
        color: rgb(102, 249, 111);
        text-shadow: 0 0 1.19vh rgba(102, 249, 111, 0.5);
      }
    }
  }

  .button-container {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 1vh;

    .counter {
      all: unset;
      box-sizing: border-box;
      width: auto;
      height: auto;
      min-width: 1ch;
      max-width: 3ch;
      font-size: 1.3vh;
      color: rgb(255, 255, 255);
      text-align: center;
      font-family: "PFDinDisplayPro Medium", sans-serif;
    }

    .decrement,
    .increment {
      all: unset;
      display: flex;
      justify-content: center;
      align-items: center;
      width: 3.25vh;
      height: 3.25vh;
      color: rgb(255, 255, 255);
      border-radius: 0.5vh;
      background-color: rgb(47, 48, 48);
      cursor: pointer;
      transition: 200ms background-color;

      &:hover {
        background-color: rgb(255, 255, 255);
        color: rgb(0, 0, 0);
      }
    }

    .remove {
      all: unset;
      display: flex;
      justify-content: center;
      align-items: center;
      margin-left: 0.75vh;
      width: 3.25vh;
      height: 3.25vh;
      color: rgb(255, 255, 255);
      border-radius: 0.5vh;
      background-color: rgb(128, 52, 52);
      cursor: pointer;
      transition: 200ms background-color;

      &:hover {
        background-color: rgb(168, 65, 65);
      }
    }
  }
}

.no-items {
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
  width: calc(100% + 2.2vh);
  height: calc((7 * 6.5vh) + (6 * 1vh));
  padding-right: 1.5vh;
  font-size: 10vh;
  color: rgba(255, 255, 255, 0.15);
}

.item-move {
  &-enter-active {
    transition:
      transform 300ms ease-in-out,
      opacity 300ms ease-in-out;
  }
  &-enter-from {
    transform: translate3d(0, 25%, 0);
    opacity: 0;
  }
  &-enter-to {
    transform: translate3d(0, 0, 0);
    opacity: 1;
  }
}
</style>
