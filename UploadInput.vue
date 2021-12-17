<template>
    <div class="">
      <div 
        v-if="!disabled"
        class="flex justify-center px-6 pt-5 pb-6 border-2 border-gray-300 border-dashed rounded-md"
        @dragover.prevent
        @dragenter.prevent
        @drop.prevent="dropFile"
      >
        <div class="space-y-1 text-center">
          <ArrowCircleUpIcon class="mx-auto h-12 w-12 text-gray-400"></ArrowCircleUpIcon>
          <div class="flex text-lg text-center">
            <label :for="`file-upload-${label}`" class="w-full cursor-pointer rounded-md text-blue-600 hover:text-blue-500 focus-within:outline-none focus-within:ring-2 focus-within:ring-offset-2 focus-within:ring-indigo-500">
              <span class="font-semibold">Sélectionnez</span>
              <input :id="`file-upload-${label}`" name="file-upload" type="file" class="sr-only" @change="uploadFile"/>
              <span class="pl-1 text-gray-600">ou glissez-déposez votre {{ label}}</span>
            </label>
          </div>
          <p class="text-sm text-gray-400 italic">{{ description }}</p>
        </div>
      </div>

      <div v-else
        class="flex justify-center px-6 pt-5 pb-6 border-2 border-gray-300 border-dashed rounded-md bg-gray-100 cursor-not-allowed"
      >
        <ArrowCircleUpIcon class="mx-auto h-12 w-12 text-gray-400 my-8"></ArrowCircleUpIcon>
      </div>


      <!-- <div v-if="loadedFile" class="flex justify-center items-center my-3">
        <div class="inline-block relative w-full">
          <embed :src="loadedFile" class="shadow-md w-full" type="application/pdf" width="100%" height="500px"/>
          <button @click="deletePreview" class="absolute top-2 right-2 p-2 bg-opacity-70 hover:bg-gray-200 rounded-lg text-gray-600">
            <XIcon class="w-6 h-6"/>
          </button>
        </div>
      </div> -->

      <button @click="openFile" class="bg-indigo-100 flex w-full p-2 my-3 justify-center rounded shadow-sm" v-if="availableFile">
        Voir le fichier
          <EyeIcon class="w-6 h-6 mx-3"></EyeIcon> 
        </button>
    </div>
</template>

<script>
import { ref } from 'vue';
import { XIcon, ArrowCircleUpIcon, EyeIcon } from '@heroicons/vue/outline';

export default {
  props: {
    label: {
      type: String,
      default: 'document',
    },
    description: {
      type: String,
      default: '',
    },
    availableFile: {
      type: Boolean,
      default: false,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
  },
  components: {
      XIcon, ArrowCircleUpIcon, EyeIcon
  },

  setup(props, { emit, $http }) {
    let loadedFile = ref();

    const getContent = (img) => img.split(',')[1];

    const getContentType = (img) => img.split(';')[0].split(':')[1];

    const deletePreview = () => loadedFile.value = null;

    const uploadFile = (event) => convertFileAsUrl(event.target.files);

    const dropFile = (event) => convertFileAsUrl(event.dataTransfer.files);

    const openFile = async () => {
      let value = loadedFile.value;
      if (value === undefined) {
        emit('opened')
        return;
      }

      const tab = window.open('', '');
      const content = (`<embed width=100% height=100%  src="${value}" type="application/pdf" />`);
      tab.document.write(content);
    };

    const convertFileAsUrl = (files) => {
      const reader = new FileReader;
      reader.onload = e => {
        loadedFile.value = e.target.result;
        emit('updated', {
          content: getContent(loadedFile.value),
          "content-type": getContentType(loadedFile.value),
        });
      }
      reader.readAsDataURL(files[0]);
    };


    return {
      loadedFile,
      uploadFile,
      deletePreview,
      openFile,
      dropFile
    }
  }
};
</script>
