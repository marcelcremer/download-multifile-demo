<script setup lang="ts">
import { saveAs } from 'file-saver';
import JSZip from 'jszip';
import { ref, computed } from 'vue';
import './app.css';

/**
 * For demo purposes
 */
const DEMO_FILE_AMOUNT = 1000;
const DEMO_URLS = new Array(DEMO_FILE_AMOUNT).fill(null).map((_val, i) => `https://robohash.org/${i}.png`);

/**
 * Progress bar
 */
const current = ref(0);
const max = ref(DEMO_FILE_AMOUNT);
const currentStatus = computed(() => Math.floor((current.value * 100) / max.value));

/**
 * Utility
 */
const downloadAsBlob = async (url: string) => fetch(url).then((resp) => resp.blob());

const waitForSeconds = async (timeInSeconds: number) => new Promise((resolve) => setTimeout(resolve, timeInSeconds * 1000));

const downloadInParallel = async (urls: string[], maxParalellism = 10, throttleInSeconds = 0) => {
  // eslint-disable-next-line
  let result: PromiseSettledResult<Blob>[] = [];
  const copyOfUrls = urls.slice();
  do {
    const filesToDownload = copyOfUrls.splice(0, maxParalellism);
    console.log('Downloading next', filesToDownload);
    const blobs = await Promise.allSettled(filesToDownload.map(async (url: string) => downloadAsBlob(url)));
    console.log({ blobs });
    result = result.concat(blobs);
    if (throttleInSeconds) await waitForSeconds(throttleInSeconds);
    current.value = current.value + maxParalellism;
  } while (copyOfUrls.length);
  console.log(result);
  return result.filter((entry) => entry.status != 'rejected').map((entry: any) => entry.value);
};

const downloadAndStoreAsZip = async () => {
  let allDownloads = await downloadInParallel(DEMO_URLS);
  console.log(allDownloads);
  var zip = new JSZip();
  allDownloads.forEach((blob, i) => zip.file(`${Math.floor(i / 10)}/file-${i}.png`, blob));
  zip.generateAsync({ type: 'blob' }).then((content: Blob) => saveAs(content, `demo-${Date.now()}.zip`));
};

const executeDownload = async () => {
  await downloadAndStoreAsZip();
};
</script>

<template>
  <div class="p-3 text-center">
    <span class="text-4xl">Client Batch Download Example</span>
  </div>
  <div class="p-3">
    <button
      @click="executeDownload"
      class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800"
    >
      Download plenty of files
    </button>
    Status: {{ current }} / {{ max }} ({{ currentStatus }}%)

    <div class="w-full bg-gray-200 rounded-full dark:bg-gray-700">
      <div class="bg-blue-600 text-xs font-medium text-blue-100 text-center p-0.5 leading-none rounded-full" :style="{ width: currentStatus + '%' }">{{ currentStatus }}%</div>
    </div>
    <br />
    <div class="text-center">
      <ul class="list-disc list-inside">
        <li v-for="url in DEMO_URLS" v-bind:key="url">
          {{ url }}
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped></style>
