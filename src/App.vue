<template>
  <div
    class="max-w-xl mx-auto h-screen overflow-hidden border flex flex-col justify-center"
  >
    <label
      class="w-48 h-48 bg-gray-200 rounded-lg jz-card block mx-auto p-9 text-center border-gray-300 border"
    >
      <i class="pb-3 text-amber-600 block">+</i>
      <strong class="block">Upload Image</strong>
      <input
        @change="handelUpload"
        type="file"
        accept="image/*"
        class="opacity-0 w-px h-px absolute left-0 top-0"
      />
    </label>

    <div class="mt-5">
      <div ref="REF_CROP"></div>
    </div>
    <div class="text-center">
      <button
        @click="getCrop"
        type="button"
        class="rounded bg-amber-400 px-3 py-2 hover:bg-amber-500"
      >
        Get image
      </button>
    </div>
    <img
      v-if="CROP_RESULT"
      :src="CROP_RESULT"
      class="mx-auto mb-3 rounded border p-3 mt-5"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, onUnmounted, ref } from "vue";
import Croppie from "croppie";
import "croppie/croppie.css";

export default defineComponent({
  setup() {
    const REF_CROP = ref(null as HTMLElement | null);
    const CROP_INST = ref(null as null | Croppie);
    const CROP_RESULT = ref("");

    const handelUpload = (e: Event) => {
      const target = e.target as HTMLInputElement;
      if (!target || target === null) return;
      const files = target.files || [];
      if (!files.length) return;

      const _reader = new FileReader();
      _reader.onload = (e) => {
        if (typeof e.target?.result === "string") {
          generateCrop(e.target?.result);
        }
      };
      _reader.readAsDataURL(files[0]);
    };

    const generateCrop = (url: string) => {
      if (CROP_INST.value !== null) {
        CROP_INST.value.bind({ url });
      }
    };

    const getCrop = () => {
      if (CROP_INST.value !== null) {
        CROP_INST.value.result({ type: "base64" }).then((d) => {
          console.log(d);
          CROP_RESULT.value = d;
        });
      }
    };

    const initCrop = () => {
      const app = REF_CROP.value;
      if (app !== null) {
        CROP_INST.value = new Croppie(app, {
          viewport: { width: 150, height: 150 },
          boundary: { width: 200, height: 200 },
        });
        CROP_INST.value.bind({ url: require("../src/assets/logo.png") });
      }
    };

    const removeCrop = () => {
      if (CROP_INST.value) CROP_INST.value?.destroy();
    };

    onMounted(() => initCrop());

    onUnmounted(() => removeCrop());

    return {
      REF_CROP,
      handelUpload,
      getCrop,
      CROP_RESULT,
    };
  },
});
</script>
