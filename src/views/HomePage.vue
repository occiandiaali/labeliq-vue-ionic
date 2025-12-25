<template>
  <ion-page>
    <ion-header class="ion-no-border">
      <ion-toolbar>
        <!-- <ion-title>LabelIQ</ion-title> -->
        <ion-img
          src="/src/assets/img/labeliq-nobg.png"
          alt="labelIQ"
          class="logo"
        ></ion-img>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header class="ion-no-border" collapse="condense">
        <ion-toolbar>
          <!-- <ion-title size="large">LabelIQ</ion-title> -->
          <ion-img
            src="/src/assets/img/logo.png"
            alt="labelIQ"
            class="logo"
          ></ion-img>
        </ion-toolbar>
      </ion-header>
      <ion-card style="margin-top: 2%">
        <ion-card-content>
          <ion-list>
            <ion-label>Select Image from your device</ion-label>
            <ion-item>
              <input
                ref="imgInputRef"
                type="file"
                id="img-upload"
                @change="handleImgUploadChange"
              />
            </ion-item>
            <ion-button
              id="open-loading"
              size="small"
              @click="imgUploadDialog"
              :disabled="imgFile === null"
              >Upload</ion-button
            >
            <ion-loading
              trigger="open-loading"
              :duration="5000"
              message="Launching AI results..."
            >
            </ion-loading>
            <ion-button
              size="small"
              v-if="haveImg"
              color="danger"
              @click="clearUploadImg"
              >Clear</ion-button
            ><br />
          </ion-list>
          <ion-checkbox>I agree to the terms and conditions</ion-checkbox>
        </ion-card-content>
      </ion-card>
      <ion-grid v-if="keptFotos.length > 0">
        <ion-list>
          <ion-row>
            <!-- CHANGE: Create a new column and image component for each photo -->
            <ion-col size="3" :key="foto.filepath" v-for="foto in keptFotos">
              <div class="img-div" id="show-loading">
                <ion-img
                  :src="foto"
                  alt="Photo"
                  class="thumbnail"
                  @click="() => handleSummaryModal(foto)"
                ></ion-img
                ><ion-icon
                  class="trash-icon"
                  color="danger"
                  :icon="trash"
                ></ion-icon>
              </div>
            </ion-col>
          </ion-row>
        </ion-list>
        <ion-loading
          trigger="show-loading"
          :duration="5000"
          message="Starting processes..."
        >
        </ion-loading>
      </ion-grid>
      <!-- <ion-grid v-if="photos.length > 0">
        <ion-list>
          <ion-row>
            
            <ion-col size="3" :key="photo.filepath" v-for="photo in photos">
              <div class="img-div">
    
                <ion-img
                  :src="photo.webviewPath"
                  alt="item"
                  class="thumbnail"
                  @click="() => openSummaryModal(photo.webviewPath)"
                ></ion-img
                ><ion-icon
                  class="trash-icon"
                  color="danger"
                  :icon="trash"
                ></ion-icon>
              </div>
            </ion-col>
          </ion-row>
        </ion-list>
      </ion-grid> -->

      <ion-fab vertical="bottom" horizontal="center" slot="fixed">
        <!-- CHANGE: Add a click event listener to the floating action button. -->

        <!-- <ion-fab-button aria-label="Fab button" @click="addNewToGallery()"> -->
        <ion-fab-button aria-label="Fab button" @click="takePhoto()">
          <ion-icon :icon="camera"></ion-icon>
        </ion-fab-button>
      </ion-fab>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { camera, close, trash } from "ionicons/icons";
import { ref } from "vue";
import {
  IonButton,
  IonCard,
  IonCardContent,
  IonCheckbox,
  IonCol,
  IonPage,
  IonGrid,
  IonHeader,
  IonImg,
  IonList,
  IonItem,
  IonLabel,
  IonLoading,
  IonFab,
  IonFabButton,
  IonIcon,
  IonRow,
  IonToolbar,
  IonTitle,
  IonContent,
  modalController,
} from "@ionic/vue";

import { Camera, CameraResultType, CameraSource } from "@capacitor/camera";

import SummaryModal from "@/components/SummaryModal.vue";

// CHANGE: Add `usePhotoGallery` import
import { usePhotoGallery } from "@/composables/usePhotoGallery";

import { puter } from "@heyputer/puter.js";

// CHANGE: Destructure `addNewToGallery` from `usePhotoGallery()
const { photos, addNewToGallery } = usePhotoGallery();

//const uploading = ref(false);
const foto = ref<string | undefined>(undefined);
const keptFotos = ref<any[]>([]);

// const urlInputRef = ref<HTMLInputElement | null>(null);
// const urlFile = ref(null);
// const haveUrl = ref(false);
//const uploadImgRef = ref(null);
const imgInputRef = ref<HTMLInputElement | null>(null);
const imgFile = ref<any>(null);
const haveImg = ref(false);

const blobFromUrl = async (url: any) => {
  const response = await fetch(url);
  return await response.blob();
};
const handleSummaryModal = async (p: any) => {
  let text = "Nothing was extracted!";
  const blob = await blobFromUrl(p);
  // console.log("Blob: ", blob);
  // console.log("BlobType: ", typeof blob);
  try {
    text = await puter.ai.img2txt(blob);
    const modal = await modalController.create({
      component: SummaryModal,
      componentProps: { extraction: text },
    });
    modal.present();
  } catch (error) {
    alert(JSON.stringify(error));
  }
};

const openSummaryModal = async (p: string | undefined) => {
  let text = "Nothing was extracted!";
  if (p !== undefined) {
    text = await puter.ai.img2txt(p);
  }
  const modal = await modalController.create({
    component: SummaryModal,
    componentProps: { extraction: text },
  });
  modal.present();
};

const handleImgUploadChange = (event: any) => {
  const file = event.target.files[0];
  imgFile.value = file;
  keptFotos.value.unshift(imgFile.value?.name);
  haveImg.value = true;
  //
  console.log("imgFile: ", imgFile.value);
  console.log("Type: ", typeof imgFile.value);
  console.log("Array: ", keptFotos.value);
};
const imgUploadDialog = () => {
  //uploading.value = true;
  if (imgFile.value !== null) {
    openSummaryModal(imgFile.value);
    // handleSummaryModal(imgFile.value);
    clearUploadImg();
  }
  // setTimeout(() => {
  //   uploading.value = false;
  // }, 5000);
};

const clearUploadImg = () => {
  if (imgInputRef.value !== null) {
    imgInputRef.value.value = "";
    imgFile.value = null;
    haveImg.value = false;
  }
};

const takePhoto = async () => {
  const photo = await Camera.getPhoto({
    resultType: CameraResultType.Uri,
    source: CameraSource.Camera,
    quality: 100,
  });
  foto.value = photo.webPath;
  keptFotos.value.unshift(foto.value);
  console.log("Value: ", foto.value);
  console.log("Type: ", typeof foto.value);
  console.log("Array: ", keptFotos.value);
};
</script>

<style lang="css" scoped>
/* ion-content {
  --background: linear-gradient(#4786ef, #f4f5f8);
} */
ion-content {
  background: #fca82a;
  --background: linear-gradient(315deg, #fca82a, #d85c27);
}

ion-toolbar {
  /*
* Created with https://www.css-gradient.com
* Gradient link: https://www.css-gradient.com/?c1=fca82a&c2=d85c27&gt=l&gd=dtl
*/
  background: #fca82a;
  --background: linear-gradient(315deg, #fca82a, #d85c27);
}
.img-div {
  position: relative;
  width: 120px;
  height: 124px;
  background-color: rgb(157, 236, 157);
  padding: 4px;
}
input {
  min-width: 320px;
}
input::placeholder {
  font-size: small;
}

.trash-icon {
  position: absolute;
  top: 5;
  right: 5;
}
.thumbnail {
  width: 100px;
  height: 100px;
  cursor: pointer;
  margin-left: 6px;
}
.logo {
  width: 80px;
  height: 38px;
  margin-left: 5px;
}
</style>
