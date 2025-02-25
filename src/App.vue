<script setup lang="ts">
import { ref, onMounted } from "vue";
import { PinPad, PinPadClient } from "@placetopay/pinpad-sdk";

const URL = "https://pinpad-dev.placetopay.ws";
// const URL = "https://pinpad-UAT.placetopay.ws";
const TOKEN = "189|KB38AU9OTgMORAIcrihdy5HP2yTduHsfVTxHh1Em";

const transactionId = ref<string>("");
const pinpadRender = ref<HTMLElement | null>(null);

const pinpad = new PinPad({ mode: "static" });
const client = new PinPadClient(URL, TOKEN);

const createTransaction = async () => {
  try {
    const response = await client.createTransaction();
    const positions = response.data.data.positions.split("").join(",");
    if (pinpadRender.value) {
      await pinpad.render("#pinpad-render", positions);
    }
    transactionId.value = response.data.data.transactionId;
  } catch (error) {
    console.error("Error creating transaction:", error);
  }
};

const onSubmit = async (event: Event) => {
  event.preventDefault();

  const form = event.target as HTMLFormElement;
  const formData = new FormData(form);
  const pin = formData.get("pin") as string;

  if (!pin) {
    console.error("No PIN received");
    return;
  }

  const data = {
    transactionId: transactionId.value,
    format: 0,
    pin,
    pan: 1234010293091,
  };

  try {
    const res = await client.generatePinBlock(data);
    console.log("PinBlock Response:", res);
  } catch (err) {
    console.error("Error generating pin block:", err);
  }
};

onMounted(() => {
  createTransaction();
});
</script>

<template>
  <div>
    <a href="https://vuejs.org/" target="_blank">
      <img src="/public/vite.svg" class="logo vue" alt="Vue logo" />
    </a>
  </div>
  <h1>Vite + Vue 3.5</h1>
  <form @submit.prevent="onSubmit">
    <div id="pinpad-render" ref="pinpadRender"></div>
    <button class="btn btn-danger btn-lg" type="submit">Continuar</button>
  </form>
</template>

<style>
.logo {
  height: 50px;
}
</style>
