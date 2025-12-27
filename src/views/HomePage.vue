<template>
  <ion-page>
    <ion-header class="ion-no-border">
      <ion-toolbar>
        <ion-img src="/labeliq-nobg.png" alt="labelIQ" class="logo"></ion-img>
        <ion-icon
          :icon="informationCircleOutline"
          size="large"
          color="dark"
          slot="end"
          style="cursor: pointer"
          @click="aboutToast"
        ></ion-icon>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header class="ion-no-border" collapse="condense">
        <ion-toolbar>
          <ion-img src="/labeliq-nobg.png" alt="labelIQ" class="logo"></ion-img>
        </ion-toolbar>
      </ion-header>
      <ion-card style="margin-top: 2%">
        <ion-card-content>
          <ion-label>
            <ion-checkbox
              label-placement="end"
              @ion-change="toggleTermsCheck"
            ></ion-checkbox>
            I agree to the
            <router-link to="/terms-and-conditions" tag="a"
              >terms and conditions</router-link
            >
          </ion-label>
          <hr />

          <ion-list>
            <ion-label>Select Image from your device</ion-label>
            <ion-item>
              <input
                ref="imgInputRef"
                type="file"
                id="img-upload"
                :disabled="!termsAccepted"
                @change="handleImgUploadChange"
              />
            </ion-item>
            <ion-button
              id="open-loading"
              size="small"
              color="tertiary"
              @click="imgUploadDialog"
              :disabled="!imgFile"
              >Analyze</ion-button
            >
            <ion-loading
              v-if="haveImg"
              trigger="open-loading"
              :duration="5000"
              message="Launching AI results..."
            >
            </ion-loading>
            <img
              :src="preview"
              alt="preview"
              width="48px"
              height="48px"
              v-if="preview"
            />
            <ion-button
              size="small"
              v-if="haveImg"
              color="danger"
              @click="clearUploadImg"
              >Clear</ion-button
            >
            <br />
            <hr />
            <div
              style="display: flex; flex-direction: column; align-items: center"
            >
              <ion-label>Or click the camera to take a photo</ion-label>
              <ion-icon
                color="tertiary"
                :icon="arrowDown"
                size="large"
                class="arrow"
                v-if="!cameraIsOpen"
              ></ion-icon>
            </div>
          </ion-list>
        </ion-card-content>
      </ion-card>
      <ion-grid v-if="keptFotos.length > 0">
        <ion-list style="padding: 6px">
          <ion-label color="medium">Click an image to analyze it</ion-label>
          <ion-row>
            <!-- CHANGE: Create a new column and image component for each photo -->
            <ion-col size="1" :key="foto.filepath" v-for="foto in keptFotos">
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
                  @click="() => removeGridPhoto(foto)"
                ></ion-icon>
              </div>
            </ion-col>
          </ion-row>
        </ion-list>
        <ion-loading
          trigger="show-loading"
          :duration="5000"
          message="Please, wait..."
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
      <!-- Camera Modal -->
      <ion-modal :is-open="isModalOpen" @didDismiss="closeCameraModal">
        <ion-content>
          <div class="camera-modal">
            <video ref="videoRef" autoplay playsinline></video>
            <canvas ref="canvasRef" style="display: none"></canvas>

            <div class="actions">
              <ion-button color="success" @click="acceptPhoto">
                <ion-icon :icon="checkmark"></ion-icon>
              </ion-button>
              <ion-button color="danger" @click="rejectPhoto">
                <ion-icon :icon="close"></ion-icon>
              </ion-button>
            </div>
          </div>
        </ion-content>
      </ion-modal>

      <!-- <ion-fab vertical="bottom" horizontal="center" slot="fixed">
        <ion-fab-button aria-label="Fab button" @click="addNewToGallery()">
        <ion-fab-button
          size="small"
          color="dark"
          aria-label="Fab button"
          @click="takePic()"
          :disabled="!termsAccepted"
        >
          <ion-icon :icon="camera" size="small"></ion-icon>
        </ion-fab-button>
      </ion-fab> -->
      <ion-fab vertical="bottom" horizontal="center" slot="fixed">
        <ion-fab-button
          size="small"
          color="dark"
          aria-label="Fab button"
          @click="openCameraModal"
          :disabled="!termsAccepted"
        >
          <ion-icon :icon="camera" size="small"></ion-icon>
        </ion-fab-button>
      </ion-fab>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  arrowDown,
  informationCircleOutline,
  checkmark,
  camera,
  close,
  trash,
} from "ionicons/icons";
import { onBeforeUnmount, onMounted, ref } from "vue";
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
  IonModal,
  IonFab,
  IonFabButton,
  IonIcon,
  IonRow,
  IonToolbar,
  IonTitle,
  IonContent,
  modalController,
  toastController,
} from "@ionic/vue";

import {
  Camera,
  CameraDirection,
  CameraResultType,
  CameraSource,
} from "@capacitor/camera";
//import { ActionSheet, ActionSheetButtonStyle } from "@capacitor/action-sheet";
import SummaryModal from "@/components/SummaryModal.vue";

// // CHANGE: Add `usePhotoGallery` import
// import { usePhotoGallery } from "@/composables/usePhotoGallery";

import { puter } from "@heyputer/puter.js";

// // CHANGE: Destructure `addNewToGallery` from `usePhotoGallery()
// const { photos, addNewToGallery } = usePhotoGallery();

//const foto = ref<string | undefined>(undefined);
const keptFotos = ref<any[]>([]);
const cameraIsOpen = ref(false);
const termsAccepted = ref(false);

const toggleTermsCheck = () => {
  termsAccepted.value = !termsAccepted.value;
  if (termsAccepted.value) {
    presentToast();
  }
};

// For actionsheet
const imageUrl = ref<string | undefined>();
// camera
const takePic = async () => {
  try {
    cameraIsOpen.value = true;
    const image = await Camera.getPhoto({
      quality: 90,
      allowEditing: true,
      resultType: CameraResultType.Uri,
      direction: CameraDirection.Rear,
    });

    imageUrl.value = image.webPath;
    keptFotos.value.unshift(imageUrl.value);
    if (imageUrl.value) cameraIsOpen.value = false;
  } catch (error) {
    cameraIsOpen.value = false;
    console.error(error);
  }
};
//===========
const videoRef = ref<HTMLVideoElement | null>(null);
const canvasRef = ref<HTMLCanvasElement | null>(null);
const isModalOpen = ref(false);
let stream: MediaStream | null = null;
const photos = ref<{ id: number; data: string }[]>([]);

async function openCameraModal() {
  isModalOpen.value = true;
  await startCamera();
}

function closeCameraModal() {
  stopCamera();
  isModalOpen.value = false;
}

async function startCamera() {
  navigator.permissions
    .query({ name: "camera" })
    .then((result) => {
      if (result.state === "denied") {
        alert(
          "Camera access is denied. Some features might not work properly."
        );
        closeCameraModal();
      } else if (result.state === "prompt") {
        alert(
          "Camera access has not yet been determined. Check your browser settings to allow camera access."
        );
        closeCameraModal();
      } else {
        console.log(`Permissions: ${result.state}`);
        try {
          navigator.mediaDevices
            .getUserMedia({
              video: { facingMode: { exact: "environment" } },
              //video: true,
              audio: false,
              // video: { facingMode: { ideal: "environment" } },
            })
            .then((st) => {
              stream = st;
              if (videoRef.value) {
                videoRef.value.srcObject = stream;
                videoRef.value
                  .play()
                  .catch((e) => console.error("videoRef.value.play err ", e));
              }
            });
        } catch (err) {
          console.error("Camera error:", err);
          closeCameraModal();
          alert(`Cannot launch camera: ${err}`);
        }
      }
    })
    .catch((error) => {
      console.log("Got error :", error);
    });
  // try {
  //   stream = await navigator.mediaDevices.getUserMedia({
  //     video: { facingMode: { exact: "environment" } },
  //     //video: true,
  //     audio: false,
  //     // video: { facingMode: { ideal: "environment" } },
  //   });
  //   if (videoRef.value) {
  //     videoRef.value.srcObject = stream;
  //     await videoRef.value.play();
  //   }
  // } catch (err) {
  //   console.error("Camera error:", err);
  //   closeCameraModal();
  //   alert(`Cannot launch camera: ${err}`);
  // }
}

function stopCamera() {
  if (stream) {
    stream.getTracks().forEach((track) => track.stop());
    stream = null;
  }
}

async function acceptPhoto() {
  if (!videoRef.value || !canvasRef.value) return;
  const video = videoRef.value;
  const canvas = canvasRef.value;
  const ctx = canvas.getContext("2d");
  canvas.width = video.videoWidth;
  canvas.height = video.videoHeight;
  if (ctx) {
    ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
    const imgData = canvas.toDataURL("image/png");
    // await savePhoto(imgData);
    keptFotos.value.unshift(imgData);
  }
  closeCameraModal();
}

function rejectPhoto() {
  closeCameraModal();
}

// async function loadPhotos() {
//   photos.value = await db.table("photos").toArray();
// }

// async function savePhoto(data: string) {
//   await db.table("photos").add({ data });
//   await loadPhotos();
// }

// async function deletePhoto(id: number) {
//   await db.table("photos").delete(id);
//   await loadPhotos();
// }

//=============

const imgInputRef = ref<HTMLInputElement | null>(null);
const imgFile = ref<any>(null);
const haveImg = ref(false);

const preview = ref<any>();

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
  const blob = URL.createObjectURL(file);

  // console.log("File: ", blob);
  preview.value = blob;

  imgFile.value = file;
  keptFotos.value.unshift(preview.value);
  haveImg.value = true;
  //
  // console.log("imgFile: ", imgFile.value);

  // console.log("Type: ", typeof imgFile.value);
  // console.log("Array: ", keptFotos.value);
};
const imgUploadDialog = () => {
  if (imgFile.value !== null) {
    openSummaryModal(imgFile.value);

    clearUploadImg();
  }
};

const clearUploadImg = () => {
  if (imgInputRef.value !== null) {
    imgInputRef.value.value = "";
    imgFile.value = null;
    haveImg.value = false;
    preview.value = null;
  }
};

const removeGridPhoto = (id: any) => {
  if (window.confirm(`Are you sure you want to remove this item?`)) {
    // console.log("Done");
    const pos = keptFotos.value.indexOf(id);

    keptFotos.value.splice(pos, 1);
  }
};

// const takePhoto = async () => {
//   const photo = await Camera.getPhoto({
//     resultType: CameraResultType.Uri,
//     source: CameraSource.Camera,
//     quality: 100,
//   });
//   foto.value = photo.webPath;
//   keptFotos.value.unshift(foto.value);
//   console.log("Value: ", foto.value);
//   console.log("Type: ", typeof foto.value);
//   console.log("Array: ", keptFotos.value);
// };
// //
// const showActionSheet = async () => {
//   const result = await ActionSheet.showActions({
//     title: "Photo Options",
//     message: "Select an option to perform",
//     options: [
//       {
//         title: "Camera",
//       },
//       {
//         title: "Upload",
//       },
//       {
//         title: "Share",
//       },
//       {
//         title: "Remove",
//         style: ActionSheetButtonStyle.Destructive,
//       },
//     ],
//   });

//   console.log("Action Sheet result:", result);
//   if (result.index === 0) {
//     takePic();
//   }
// };

const presentToast = async () => {
  const toast = await toastController.create({
    message:
      "You'll be prompted to authenticate with Puter when using AI features for the first time. This also activates billing processes per your usage. Read 'terms and conditions' for details.",
    duration: 5000,
    position: "middle",
    color: "dark",
  });
  await toast.present();
};
const aboutToast = async () => {
  const toast = await toastController.create({
    message:
      "LabelIQ is your handy, AI-powered help in knowing more about what is contained in what you use/consume. Find out about food or product ingredients before you buy/use them. Agree to terms & conditions to enable action 'Choose File' and Camera buttons.",
    duration: 12000,
    position: "middle",
    color: "dark",
  });
  await toast.present();
};
const offlineToast = async () => {
  const toast = await toastController.create({
    message:
      "It looks like you have poor/no Internet connection. You need connectivity to use this app properly.",
    duration: 5000,
    position: "middle",
    color: "danger",
  });
  await toast.present();
};

onBeforeUnmount(() => {
  stopCamera();
});

onMounted(() => {
  if (!navigator.onLine) {
    offlineToast();
  }
});
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
ion-grid {
  --ion-grid-columns: 3;
  --ion-grid-margin-sm: 10px;
}
.img-div {
  width: 120px;
  height: 124px;
  /* background-color: rgb(157, 236, 157); */
  padding: 4px;
}

input {
  min-width: 320px;
}
input::placeholder {
  font-size: small;
}

.thumbnail {
  width: 84px;
  height: 84px;
  cursor: pointer;
  /* background-color: greenyellow; */
}
.logo {
  width: 80px;
  height: 38px;
  margin-left: 5px;
}

.camera-modal {
  display: flex;
  flex-direction: column;
  align-items: center;
}
video {
  width: 100%;
  max-width: 400px;

  border: 1px solid #ccc;
}
.actions {
  margin-top: 10px;
  display: flex;

  gap: 16px;
}
.thumb-wrapper {
  position: relative;
}
.thumb-wrapper img {
  width: 100%;
  border: 1px solid #ccc;
}
.trash-icon {
  /* position: absolute;
  top: 5px;
  right: 5px; */
  color: red;
  font-size: 24px;
  cursor: pointer;
}
/** animate direction to camera icon arrow */
@keyframes bounce {
  0%,
  20%,
  50%,
  80%,
  100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-20px);
  }
  60% {
    transform: translateY(-10px);
  }
}

.arrow {
  animation: bounce 1s infinite;
}
</style>
