<template>
  <div>
    <div class="mt-2">
      <div 
        v-if="imgs.length < 1" 
        class="flex justify-center px-6 pt-5 pb-6 border-2 border-gray-300 border-dashed rounded-md"
        @drop.prevent="dropFile"
      >
        <div class="space-y-1 text-center">
          <svg class="mx-auto h-12 w-12 text-gray-400" stroke="currentColor" fill="none" viewBox="0 0 48 48" aria-hidden="true">
            <path d="M28 8H12a4 4 0 00-4 4v20m32-12v8m0 0v8a4 4 0 01-4 4H12a4 4 0 01-4-4v-4m32-4l-3.172-3.172a4 4 0 00-5.656 0L28 28M8 32l9.172-9.172a4 4 0 015.656 0L28 28m0 0l4 4m4-24h8m-4-4v8m-12 4h.02" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
          </svg>
          <div class="flex text-sm text-gray-600">
            <label for="file-upload" class="relative cursor-pointer bg-white rounded-md font-medium text-green-600 hover:text-green-500 focus-within:outline-none focus-within:ring-2 focus-within:ring-offset-2 focus-within:ring-green-500">
              <span v-if="multiple">Ajouter des documents</span>
              <span v-else>Ajouter un document</span>
              <input id="file-upload" name="file-upload" type="file" class="sr-only" @change="previewFilesEvent" :multiple="multiple"/>
            </label>
            <p class="pl-1">ou glisser-déposer ici</p>
          </div>
          <p class="text-xs text-gray-500">
            Image (formats PNG, JPG) ou document (format PDF)
          </p>
        </div>
      </div>
      <div v-else class="flex justify-center items-center">
        <div class="inline-block relative w-full" v-for="img in imgs" :key="img.id">
          <embed :src="img" class="shadow-md w-full "/>
          <button @click="deletePreview" class="absolute top-2 right-2 p-2 bg-white bg-opacity-70 hover:bg-gray-200 rounded-lg text-gray-600">
            <XIcon class="w-6 h-6"/>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import { XIcon } from '@heroicons/vue/outline';

export default {
  props: {
    multiple: Boolean,
    content: {
      default: null,
    },
  },
  components: {
      XIcon,
  },
  setup(props, { emit }) {
    const imgs = ref([]);

    const previewImage = ref(props.content);
    const getContent = (img) => img.split(',')[1];

    const getContentType = (img) => img.split(';')[0].split(':')[1];

    const previewFilesEvent = (event) => setPreviewFromFile(event.target.files);

    const deletePreview = () => imgs.value = [];

    const dropFile = (event) => setPreviewFromFile(event.dataTransfer.files);

    const setPreviewFromFile = (files) => {
      Array.from(files).forEach((_, index) => {
        if (files && files[index]) {
          let reader = new FileReader
          reader.onload = e => {
            previewImage.value = e.target.result;
            imgs.value.push(previewImage.value);
            if (!props.multiple) {
              emit('update', {
                content: getContent(imgs.value[0]),
                "content-type": getContentType(imgs.value[0]),
              });
            }
          }
          reader.readAsDataURL(files[index])
        }
      });
    };


    return {
      imgs,
      previewFilesEvent,
      deletePreview,
      dropFile
    }
  }
};
</script>