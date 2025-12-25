<template>
  <ion-header>
    <ion-toolbar>
      <ion-buttons slot="end">
        <ion-button color="danger" @click="cancel">Close</ion-button>
      </ion-buttons>
      <ion-title>Result</ion-title>
      <!-- <ion-buttons slot="end">
        <ion-button color="success" @click="confirm" :strong="true"
          >Save</ion-button
        >
      </ion-buttons> -->
    </ion-toolbar>
  </ion-header>
  <ion-content class="ion-padding">
    <!-- <ion-item>
      <ion-input
        label-placement="stacked"
        label="Enter your name"
        v-model="name"
        placeholder="Your name"
      ></ion-input>
    </ion-item>
    <ion-item> </ion-item> -->
    <ion-card>
      <ion-card-header>
        <ion-card-subtitle>Received summation</ion-card-subtitle>
      </ion-card-header>
      <ion-card-content v-if="!name">
        <p>Please, wait. I'm working..</p>
      </ion-card-content>
      <ion-card-content v-else>
        {{ name }}
      </ion-card-content>
    </ion-card>
    <ion-card v-if="name">
      <ion-card-header>
        <ion-card-subtitle>Next move</ion-card-subtitle>
      </ion-card-header>
      <ion-card-content>
        <ion-button color="dark">Export</ion-button>
        <ion-button color="danger">Discard</ion-button>
      </ion-card-content>
    </ion-card>
  </ion-content>
</template>

<script setup lang="ts">
import {
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardSubtitle,
  IonContent,
  IonHeader,
  IonTitle,
  IonToolbar,
  IonButtons,
  IonButton,
  IonItem,
  IonInput,
  modalController,
} from "@ionic/vue";
import { onMounted, ref } from "vue";

import { puter } from "@heyputer/puter.js";

const props = defineProps(["extraction"]);

//const name = ref();
const name = ref();

const cancel = () => modalController.dismiss(null, "cancel");
const confirm = () => modalController.dismiss(name.value, "confirm");
const promptHeavy = `
You are an expert in nutrition, dermatology, and pharmacology. 
Analyze the following list of ingredients from a [food/cosmetic/medicine] label. 
Provide a clear, structured summary that includes:

1. ✅ Safe/Common Ingredients  
   - Brief notes on their purpose (e.g., preservative, moisturizer, nutrient)

2. ⚠️ Potential Allergens  
   - Highlight common triggers (nuts, gluten, dairy, soy, fragrances, parabens, etc.)

3. 🚫 Ingredients to Beware Of  
   - Toxicity, regulatory concerns, side effects, controversial additives

4. 👤 Suitability  
   - Indicate if safe for children, pregnant women, sensitive skin, vegans, etc.

5. 📊 Final Summary  
   - Overall safety rating: **Safe / Use with caution / Avoid**

Formatting rules:  
- Use bullet points or a table for readability  
- Keep explanations concise and avoid technical jargon  
- Use symbols/emojis (✅ ⚠️ 🚫 👤 📊) to make scanning easier  

Ingredients: ${props.extraction}

`;

const promptLite = `
Analyze this [food/cosmetic/medicine] ingredient list. 
Give me a short, clear summary with:

✅ Safe/common items  
⚠️ Possible allergens  
🚫 Ingredients to avoid  
📊 Overall verdict (Safe / Caution / Avoid)

Keep it concise, use bullet points + emojis, no jargon.  
Ingredients: ${props.extraction}

`;

// onMounted(() => {
//   puter.ai
//     .chat(
//       `Identify the ingredients in the text, and analyze for safety and allergens:\n${props.extraction}`,
//       { model: "meta-llama/llama-4-scout" }
//     )
//     .then((response) => {
//       name.value = response.message?.content;
//     })
//     .catch((e) => (name.value = e));
// });
onMounted(() => {
  puter.ai
    .chat(promptLite, { model: "meta-llama/llama-4-scout" })
    .then((response) => {
      name.value = response.message?.content;
    })
    .catch((e) => (name.value = e));
});
</script>
