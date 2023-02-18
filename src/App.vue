<script setup lang="ts">
import { saveAs } from 'file-saver';
import JSZip from 'jszip';

/**
 * For demo purposes
 */
const DEMO_FILE_AMOUNT = 1000;
const DEMO_URLS = new Array(DEMO_FILE_AMOUNT).fill(null).map((_val, i) => `https://robohash.org/${i}.png`);

/**
 * Utility
 */
const downloadAsBlob = async (url: string) => fetch(url).then((resp) => resp.blob());

const waitForSeconds = async (timeInSeconds: number) => new Promise((resolve) => setTimeout(resolve, timeInSeconds * 1000));

const downloadInParallel = async (urls: string[], maxParalellism = 1, throttleInSeconds = 0) => {
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
  } while (copyOfUrls.length);
  console.log(result);
  return result.filter((entry) => entry.status != 'rejected').map((entry: any) => entry.value);
};

const downloadAndStoreAsZip = async () => {
  let allDownloads = await downloadInParallel(DEMO_URLS);
  console.log(allDownloads);
  var zip = new JSZip();
  allDownloads.forEach((blob, i) => zip.file(`file-${i}.png`, blob));
  zip.generateAsync({ type: 'blob' }).then((content: Blob) => saveAs(content, 'example.zip'));
};

const executeDownload = async () => {
  await downloadAndStoreAsZip();
};
</script>

<template>
  <header>
    <div class="wrapper">
      <button @click="executeDownload">Download plenty of files</button>
      <br />
      <ul>
        <li v-for="url in DEMO_URLS" v-bind:key="url">
          {{ url }}
        </li>
      </ul>
    </div>
  </header>

  <RouterView />
</template>

<style scoped></style>
