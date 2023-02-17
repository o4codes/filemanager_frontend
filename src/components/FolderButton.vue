<script>

import streamsaver from "streamsaver";
import { WritableStream } from "web-streams-polyfill/ponyfill";
import * as zip from "@zip.js/zip.js";

export default {
  props: {
    folderId: {
      type: String,
      required: true,
    },
    baseUrl: {
      type: String,
      required: true
    }
  },

  methods: {
    async getFile(file) {
      const resp=await fetch(file.url);
      const blob=await resp.blob();
      file.blob=blob;
      return file;
    },

    async getFolderBlobs() {
      // get folder files
      console.log("Downloading Files");
      try {
        const response=await fetch(
          `${this.baseUrl}/folders/${this.folderId}/children-flat-files`
        );
        let files =await response.json();
        files.map(
          (value) => (value.url=`${this.baseUrl}/files/${value.id}/download`)
        );
        console.log("Done downloading of files");
        return await Promise.all(files.map((file) => this.getFile(file)));
      } catch(error) {
        return console.log(error);
      }
    },

    async downloadFolder() {
      const files=await this.getFolderBlobs();
      return this.exportZip(files);
    },

    async exportZip(files) {
      console.log("exporting to zip");
      const zipWriter = new zip.ZipWriter(
        new zip.BlobWriter("application/zip")
      );
      await Promise.all(
        files.map((file) => {
          zipWriter.add(file.path, new zip.BlobReader(file.blob));
        })
      );
      const currentDate = new Date().getTime();
      const fileName = `combined-${currentDate}.zip`;
      const downloadBlob = await zipWriter.close();
      console.log("done exporting to zip");
      this.downloadZip(downloadBlob, fileName);
    },

    downloadZip(blob, filename) {
      const fileStream = streamsaver.createWriteStream(filename, {
        size: blob.size, // Makes the percentage visiable in the download
      });
      const readableStream = blob.stream();

      // more optimized pipe version
      // (Safari may have pipeTo but it's useless without the WritableStream)
      if (window.WritableStream && readableStream.pipeTo) {
        return readableStream
          .pipeTo(fileStream)
          .then(() => console.log("done writing"));
      }

      // Write (pipe) manually
      window.writer = fileStream.getWriter();

      const reader = readableStream.getReader();
      const pump = () =>
        reader
          .read()
          .then((res) =>
            res.done ? writer.close() : writer.write(res.value).then(pump)
          );

      pump();
    },
  },
};
</script>

<template>
  <button @click="downloadFolder" id="download-folder-btn">
    Click to Download Folder
  </button>
</template>

<style scoped>
#download-folder-btn {
  background-color: white;
  color: dodgerblue;
  border: 1px solid dodgerblue;
  padding: 10px 15px;
  border-radius: 5px;
  outline-style: none;
  font-weight: bolder;
}

#download-folder-btn:active {
  background-color: dodgerblue;
  color: white;
  border: none;
}
</style>