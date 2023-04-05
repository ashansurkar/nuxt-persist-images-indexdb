<template>
  <div class="grid justify-center items-center h-52">
    <div
      class="border border-1 p-4 rounded-lg shadow-lg cursor-pointer"
      @click="open"
    >
      Upload
    </div>
    <div
      class="flex flex-wrap gap-4 w-full items-center"
      v-if="fileList?.length"
    >
      <div
        v-for="file in fileList"
        :key="file?.name"
        class="w-fit h-52 grid relative justify-center items-center rounded-md"
      >
        <img
          v-if="file.name"
          :src="getBlobUrl(file)"
          class="max-w-52 max-h-52 p-4"
          alt="Image"
          preview
        />
        <!--  d -->
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { useFileDialog } from '@vueuse/core';
import localforage from 'localforage';
const localStorage = ref(
  localforage.createInstance({
    name: 'IndexedImageStore',
  })
);
onMounted(() => {
  localStorage.value.getItem('imageStorekey').then((value: any) => {
    if (value) {
      let imgs = JSON.parse(value);
      imgs = imgs.map((dataURL: any) => {
        let blob = dataURLToBlob(dataURL);
        let file = new File([blob], 'image.png', { type: 'image/png' });
        return file;
      });
      fileList.value = [...imgs];
    }
  });
});
const dataURLToBlob = (dataURL: any) => {
  const parts = dataURL.split(',');
  const contentType = parts[0].split(':')[1];
  const b64Data = atob(parts[1]);
  const byteArrays = [];
  for (let i = 0; i < b64Data.length; i++) {
    byteArrays.push(b64Data.charCodeAt(i));
  }
  return new Blob([new Uint8Array(byteArrays)], { type: contentType });
};
const { files, open, reset } = useFileDialog({
  accept: ['image/*'],
  multiple: true,
});
const fileList = ref([]);
const getBlobUrl = (file) => {
  let fileSrc = URL.createObjectURL(file);
  return fileSrc;
};
watch(files, () => {
  fileList.value = [...files.value];
  let fileUrls: string[] = [];
  for (let file of fileList.value) {
    let reader = new FileReader();
    reader.onload = function (event) {
      let imageUrl: string | ArrayBuffer = event.target?.result || '';

      const image = new Image();
      image.onload = function () {
        // create a canvas element and set its dimensions to the resized image dimensions
        const canvas = document.createElement('canvas');
        let width = image.width;
        let height = image.height;
        let maxWidth = 1200;
        let maxHeight = 1200;
        if (width > height) {
          if (width > maxWidth) {
            height *= maxWidth / width;
            width = maxWidth;
          }
        } else {
          if (height > maxHeight) {
            width *= maxHeight / height;
            height = maxHeight;
          }
        }
        canvas.width = width;
        canvas.height = height;

        // draw the image on the canvas with the resized dimensions
        const context = canvas.getContext('2d');
        context?.drawImage(image, 0, 0, canvas.width, canvas.height);
        // get the resized image data as a data URL with reduced quality
        const imageDataUrl = canvas.toDataURL('image/jpeg', 0.8); // reduce quality to 80%

        // create a new object with the image data and add it to the object store
        // localStorage.value.setItem('imageStorekey', JSON.stringify(imageDataUrl));

        fileUrls.push(imageDataUrl);
        // to store all files in one key
        if (fileUrls.length === fileList.value.length) {
          console.log({ fileUrls });
          localStorage.value.setItem('imageStorekey', JSON.stringify(fileUrls));
        }
      };
      image.src = imageUrl as string;
    };
    reader.readAsDataURL(file);
  }
});
</script>
