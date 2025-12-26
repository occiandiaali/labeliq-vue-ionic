<template>
  <ion-page>
    <ion-header class="ion-no-border">
      <ion-toolbar>
        <ion-icon
          :icon="arrowBack"
          size="small"
          color="dark"
          slot="start"
          style="cursor: pointer; margin: 6px"
          role="button"
          @click="goBack"
        ></ion-icon>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <div style="padding: 6px">
        <h1>Terms &amp; Conditions</h1>
        <h3>Coming Soon</h3>
        <p>
          Lorem, ipsum dolor sit amet consectetur adipisicing elit. Dolores
          autem amet corporis. Quisquam tempora dolores voluptate earum ipsam,
          eaque culpa maxime obcaecati necessitatibus temporibus, ut autem, vero
          odio ea ipsum?
        </p>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { arrowBack } from "ionicons/icons";
import {
  IonContent,
  IonPage,
  IonHeader,
  IonIcon,
  IonToolbar,
} from "@ionic/vue";
import router from "@/router";

const goBack = () => router.go(-1);
</script>
