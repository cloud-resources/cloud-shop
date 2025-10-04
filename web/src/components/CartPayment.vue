<script setup lang="ts">
// External Libraries
import { computed, ref, watch } from "vue"
import { Icon } from "@iconify/vue"

// Stores
import { useShopStore } from "@/stores/shop"
const shopStore = useShopStore()

import { useConfigStore } from "@/stores/config"
const configStore = useConfigStore()

// Utils
import { callback } from "@/utils/callback"
import { formatPrice } from "@/utils/formatPrice"

const payCart = async (type: string): Promise<void> => {
  if (shopStore.cart.length === 0) return

  const response = await callback({
    action: "payItems",
    type,
    cart: shopStore.cart,
  })
  if (response) clearCart()
}

const clearCart = (): void => {
  if (shopStore.cart.length === 0) return
  shopStore.cart = []
}

const totalPrice = computed(() => shopStore.cart.reduce((total, item) => total + item.price * item.quantity, 0))
const animatedPrice = ref(totalPrice.value)
const displayPrice = computed(() => formatPrice(Math.round(animatedPrice.value)))

let rafId: number | null = null
let startTime: number
let startValue: number
let targetValue: number
const duration = 150

const animatePrice = (target: number) => {
  if (rafId) cancelAnimationFrame(rafId)

  startTime = performance.now()
  startValue = animatedPrice.value
  targetValue = target

  const step = (time: number) => {
    const elapsed = time - startTime
    const progress = Math.min(elapsed / duration, 1)
    const eased = 1 - (1 - progress) * (1 - progress)
    animatedPrice.value = Math.round(startValue + (targetValue - startValue) * eased)

    if (progress < 1) {
      rafId = requestAnimationFrame(step)
    } else {
      animatedPrice.value = targetValue
    }
  }

  rafId = requestAnimationFrame(step)
}

watch(totalPrice, newPrice => animatePrice(newPrice))
</script>

<template>
  <section class="payment">
    <div class="payment-title">{{ configStore.locales.cart.payment.title }}</div>
    <div class="price">{{ displayPrice }}</div>
    <div class="pay">
      <button @click="payCart('bank')">
        <Icon icon="mingcute:bank-card-fill" />
        {{ configStore.locales.cart.payment.pay_bank }}
      </button>

      <button @click="payCart('cash')">
        <Icon icon="mdi:wallet" />
        {{ configStore.locales.cart.payment.pay_cash }}
      </button>
    </div>
  </section>
</template>

<style scoped>
.payment {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 100%;
  height: fit-content;
  justify-content: space-around;
  gap: 0.7vh;
  padding: 1.5vh;
  border-radius: 0.5vh;
  background-color: rgb(30, 30, 30);
}

.payment-title {
  font-family: "PFDinDisplayPro Medium", sans-serif;
  font-size: 1.7vh;
  letter-spacing: 0.05em;
  color: rgb(255, 255, 255);
}

.price {
  overflow: auto;
  width: 100%;
  height: fit-content;
  font-family: "PFDinDisplayPro Bold", sans-serif;
  font-size: 4.2vh;
  color: rgb(102, 249, 111);
  text-shadow: 0 0 1.19vh rgba(102, 249, 111, 0.4);
}

.pay {
  margin-left: auto;
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  gap: 1vh;
  width: 100%;
  height: fit-content;

  button {
    all: unset;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 5vh;
    gap: 0.5vh;
    font-family: "PFDinDisplayPro Bold", sans-serif;
    font-size: 1.4vh;
    color: rgb(255, 255, 255);
    border-radius: 0.5vh;
    background-color: rgb(47, 48, 48);
    transition: 200ms background-color;

    svg {
      width: 2.5vh;
      height: auto;
      color: rgb(255, 255, 255);
    }

    &:hover {
      background-color: rgb(255, 255, 255);
      color: rgb(0, 0, 0);
      cursor: pointer;

      svg {
        color: rgb(0, 0, 0);
      }
    }
  }
}
</style>
