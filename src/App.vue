<template>
  <div class="max-w-xl mx-auto h-screen py-20">
    <div class="card">
      <label v-if="!CROP_INST" class="p-9 h-full flex flex-col justify-center">
        <span class="text-4xl font-extrabold pb-3 text-amber-600 block">+</span>
        <strong class="block">Upload Image</strong>
        <input
          @change="handelUpload"
          type="file"
          accept="image/*"
          class="opacity-0 w-px h-px absolute left-0 top-0"
        />
      </label>
      <div v-else class="text-right">
        <button
          @click="removeCrop"
          type="button"
          class="py-2 px-3 font-extrabold"
          title="Remove"
        >
          X
        </button>
      </div>
      <div ref="REF_CROP"></div>
      <button
        v-if="CROP_INST"
        @click="getCrop"
        type="button"
        class="rounded bg-amber-400 px-3 py-2 hover:bg-amber-500 mt-auto mb-5"
      >
        Get image
      </button>
    </div>

    <div v-if="CROP_INST" class="card h-[250px]">
      <div class="flex flex-col justify-center p-5 h-full">
        <figure class="mx-auto mb-3 border p-2 rounded">
          <img v-if="CROP_RESULT" :src="CROP_RESULT" />
          <div v-else class="w-[150px] h-[150px] bg-gray-100"></div>
        </figure>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onUnmounted, ref } from "vue";
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
          setCrop(e.target?.result);
        }
      };
      _reader.readAsDataURL(files[0]);
    };

    /**
     * 若 Crop 尚未產生，則建立一個實體
     * 若已存在實體，則變更圖片
     */
    const setCrop = (url: string) => {
      if (CROP_INST.value !== null) {
        CROP_INST.value.bind({ url });
        return;
      }
      generateCrop(url);
    };

    /** 產生一個 Crop */
    const generateCrop = (url: string) => {
      const app = REF_CROP.value;
      if (app !== null) {
        CROP_INST.value = new Croppie(app, {
          viewport: { width: 150, height: 150 },
          boundary: { width: 200, height: 200 },
          customClass: "h-auto",
        });
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

    const removeCrop = () => {
      if (CROP_INST.value) {
        CROP_INST.value?.destroy();
        CROP_INST.value = null;
        CROP_RESULT.value = "";
      }
    };

    onUnmounted(() => removeCrop());

    return {
      REF_CROP,
      handelUpload,
      getCrop,
      CROP_RESULT,
      CROP_INST,
      removeCrop,
    };
  },
});
</script>

<style lang="postcss">
html {
  @apply bg-gray-200;
}
.card {
  @apply rounded-lg block mx-auto text-center border-gray-300 border shadow w-[300px];
}
</style>
