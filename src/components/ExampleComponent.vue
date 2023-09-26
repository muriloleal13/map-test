<template>
  <div class="d-flex text-center" style="height: 20vh">
    <div class="m-auto">
      <h4>Your Position</h4>
      Latitude: {{ currPos.lat.toFixed(2) }}, Longitude:
      {{ currPos.lng.toFixed(2) }}
    </div>
  </div>
  <div ref="mapDiv" style="width: 100%; height: 70vh" />
</template>

<script>
/* eslint-disable no-undef */
import {
  computed,
  ref,
  onMounted,
  defineComponent,
  onUnmounted,
  watch,
} from 'vue';
import { useGeolocation } from '../useGeolocation';
import { Loader } from '@googlemaps/js-api-loader';

const GOOGLE_MAPS_API_KEY = import.meta.env.VITE_GOOGLE_API_KEY;

export default defineComponent({
  name: 'ExampleComponent',
  props: {
    title: {
      type: String,
      required: true,
    },
    active: {
      type: Boolean,
    },
  },
  setup() {
    const { coords } = useGeolocation();
    const currPos = computed(() => ({
      lat: coords.value.latitude,
      lng: coords.value.longitude,
    }));
    const otherPos = ref(null);

    const loader = new Loader({ apiKey: GOOGLE_MAPS_API_KEY });
    const mapDiv = ref(null);
    let map = ref(null);
    let clickListener = null;
    let rotation = 0; // Inicializa a rotação do mapa

    onMounted(async () => {
      await loader.load();
      map.value = new google.maps.Map(mapDiv.value, {
        center: currPos.value,
        // center: { lat: -29.68524040960456, lng: -53.80358747115425 },
        // center: { lat: -29.98561984040264, lng: -52.37887461278487 },
        zoom: 20,
        heading: 320,
        tilt: 47.5,
        mapId: '90f87356969d889c',
      });

      rotateMap();
    });

    onUnmounted(async () => {
      if (clickListener) clickListener.remove();
    });

    // Função para girar gradualmente o mapa
    const rotateMap = () => {
      const rotationIncrement = 1; // Valor de incremento da rotação
      const maxRotation = 360; // Valor máximo de rotação (360 graus)

      // Atualiza a rotação do mapa
      rotation = (rotation + rotationIncrement) % maxRotation;
      map.value?.setHeading(rotation);

      // Define o intervalo para continuar girando o mapa
      setTimeout(rotateMap, 50); // Ajuste o valor do timeout conforme necessário
    };

    return { currPos, otherPos, mapDiv };
  },
});
</script>
