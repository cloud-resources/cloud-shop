<script setup lang="ts">
// External Libraries
import { Icon } from "@iconify/vue"

// Stores
import { useShopStore } from "@/stores/shop"
const shopStore = useShopStore()

// Utils
import { formatPrice } from "@/utils/formatPrice"
import { getImageSrc } from "@/utils/getImageSrc"

// Modify Quantity
const decrementQuantity = (index: number): void => {
  const item = shopStore.cart[index]
  if (!item || item.quantity <= 0) return

  item.quantity--
  if (item.quantity === 0) {
    shopStore.cart.splice(index, 1)
  }
}
const incrementQuantity = (index: number): void => {
  const item = shopStore.cart[index]
  if (!item || item.quantity >= 999) return
  if (item.category === 'weapons') {
        // Prevent incrementing quantity for items in the "weapons" category
        return;
      }
      
  item.quantity++
  
}

// Validate Inputs
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
  <div v-if="shopStore.cart.length === 0" class="no-cart">
    <Icon icon="mdi:shopping-basket-off" />
  </div>
  <section class="item-list">
    <div v-for="(item, index) in shopStore.cart" :key="item.name" class="item">
      <div class="front">
        <img :src="getImageSrc(item.name)" draggable="false" />
        <div class="text-container">
          <p class="item-label">{{ item.label }}</p>
          <span class="item-price">{{ formatPrice(item.price) }}</span>
        </div>
        <div class="button-container">
          <div class="decrement" @click="decrementQuantity(index)">
            <Icon class="icon" icon="typcn:minus" />
          </div>
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
          <div class="increment" @click="incrementQuantity(index)">
            <Icon class="icon" icon="typcn:plus" />
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style lang="scss">
.no-cart {
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  width: 100%;
  font-size: 7.5rem;
  color: rgba(255, 255, 255, 0.15);
}

.item-list {
  overflow: overlay;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  scrollbar-gutter: stable;
  scroll-behavior: smooth;
  height: 100%;
  max-height: 59vh;
  gap: 1vh;

  .item {
    flex-shrink: 0;
    position: relative;
    width: 96%;
    height: 6.5vh;
    background-color: rgb(30, 30, 30);
    border-radius: 0.5vh;
    display: flex;
    align-items: center;
    padding: 1.5vh;

    .front {
      display: flex;
      flex: 1;
      align-items: center;
      justify-content: space-between;
      height: 100%;
      font-family: "PFDinDisplayPro-Medium", sans-serif;

      img {
        width: 6.25vh;
        height: auto;
        padding-right: 1.75vh;
        object-fit: contain;
      }

      .text-container {
        display: flex;
        flex: 1;
        flex-direction: row;
        align-items: center;
        gap: 1vh;

        .item-label {
          width: 10vh;
          text-align: left;
          font-size: 0.9rem;
          color: rgba(255, 255, 255, 0.8);
        }

        .item-price {
          font-size: 0.9rem;
          color: rgb(102, 249, 111);
          text-shadow: 0 0 1.19vh rgba(102, 249, 111, 0.5);
        }
      }

      .button-container {
        display: flex;
        flex-direction: row;
        align-items: center;
        gap: 1.5vh;

        .counter {
          all: unset;
          box-sizing: border-box;
          width: auto;
          height: auto;
          min-width: 1ch;
          max-width: 3ch;
          font-size: 0.85rem;
          color: rgb(255, 255, 255);
          text-align: center;
        }

        .decrement,
        .increment {
          display: flex;
          justify-content: center;
          align-items: center;
          width: 3.25vh;
          height: 3.25vh;
          font-size: 0.85rem;
          border-radius: 0.65vh;
          background-color: rgb(47, 48, 48);
          color: rgb(255, 255, 255);
          transition: 0.25s background-color;

          &:hover {
            background-color: rgb(255, 255, 255);
            color: rgb(0, 0, 0);
            cursor: pointer;
          }
        }
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
    width: 0.5vh;
  }
}
</style>
