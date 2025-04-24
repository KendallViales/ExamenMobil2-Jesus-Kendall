<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Scanner</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-button @click="scanBarcode">Scannear</ion-button>
      <ion-alert
        :is-open="showAlert"
        :header="'Texto escaneado'"
        :message="alertMessage"
        :buttons="['OK']"
        @didDismiss="showAlert = false"
      ></ion-alert>

      <ion-list>
        <ion-list-header>
          <ion-label>Historial de Escaneos</ion-label>
        </ion-list-header>
        <ion-item v-for="(item, index) in scanHistory" :key="index">
          <ion-label>{{ item }}</ion-label>
        </ion-item>
      </ion-list>
      <div>
        <div id="map" style="width: 100%; height: 400px;"></div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { IonButton, IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonAlert, IonList, IonListHeader, IonItem, IonLabel } from '@ionic/vue';
import { CapacitorBarcodeScanner } from '@capacitor/barcode-scanner';
import  mapboxgl  from 'mapbox-gl';
import { onMounted } from 'vue';
import 'mapbox-gl/dist/mapbox-gl.css';

mapboxgl.accessToken = 'pk.eyJ1Ijoia2VuZDIyNSIsImEiOiJjbTl1bDUxdmowYWxyMnZvcDZ0N3JrMnd6In0.PAXwtSzi0DdlFpzf9e0_ew';

const map = ref<any>(null);
const showAlert = ref(false);
const alertMessage = ref('');
const scanHistory = ref<string[]>([]); // Historial de códigos QR escaneados

onMounted(() => {
  map.value = new mapboxgl.Map({
	container: 'map', // container ID
	style: 'mapbox://styles/mapbox/streets-v12', // style URL
	center: [-85.446501,10.630537], // starting position [lng, lat]
	zoom: 9, // starting zoom
});
});


async function scanBarcode() {
  try {
    const result = await CapacitorBarcodeScanner.scanBarcode({ hint: 17 });

    if (result.ScanResult) {
      const content = result.ScanResult;

      scanHistory.value.unshift(content);
      if(content.includes('geo:')) {
        const coordinates = content.split(':')[1].split(',');
        const lat = parseFloat(coordinates[0]);
        const lng = parseFloat(coordinates[1]);
        new mapboxgl.Marker()
          .setLngLat([lng, lat])
          .addTo(map.value);
      }
      else if (content.includes('@') && content.includes('.')) {
        window.location.href = `mailto:${content}`;
      } else if (content.startsWith('http')) {
        window.open(content, '_blank');
      } else{
        alertMessage.value = content;
        showAlert.value = true;
      }
    } else {
      alertMessage.value = 'No se detectó contenido en el código QR.';
      showAlert.value = true;
    }
  } catch (error) {
    alertMessage.value = 'Error al escanear el código QR.';
    showAlert.value = true;
    console.error(error);
  }
}
</script>

<style scoped>
#container {
  text-align: center;
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  color: #8c8c8c;
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
