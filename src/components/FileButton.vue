<script>
import streamsaver from "streamsaver";
import { WritableStream } from "web-streams-polyfill/ponyfill";
import * as zip from "@zip.js/zip.js";

export default {
  props: {
    files: {
      type: Array,
      required: true
    },
    baseUrl: {
      type: String,
      required: true
    }
  },

  methods: {
    async downloadFiles() {
      // get file details
      // get file blob
      let files_detials = await Promise.all(
        this.files.map(file_id => this.getFile(file_id))
      )
      if (files_detials.length > 1){
        console.log(files_detials)
        return await this.exportZip(files_detials)
      }
      else {
        let file = files_detials[0]
        return this.download(file.blob, file.name)
      }
    },
    
    async getFileDetails(file_id){
      try {
        const resp=await fetch(`${this.baseUrl}/files/${file_id}`);
        const json_data=await resp.json();
        return json_data;
      } catch(error) {
        return console.log(error);
      }
    },

    async getFileBlob(file_id) {
      const resp=await fetch(`${this.baseUrl}/files/${file_id}/download`);
      const blob=await resp.blob();
      return blob;
    },

    async getFile(file_id) {
      let file_results =  await Promise.all([
        this.getFileDetails(file_id),
        this.getFileBlob(file_id)
      ])
      return {
        ...file_results[0],
        blob: file_results[1]
      }
    },

    async exportZip(files) {
      console.log("exporting to zip");
      const zipWriter = new zip.ZipWriter(
        new zip.BlobWriter("application/zip")
      );
      await Promise.all(
        files.map((file) => {
          zipWriter.add(file.name, new zip.BlobReader(file.blob));
        })
      );
      const currentDate = new Date().getTime();
      const fileName = `combined-${currentDate}.zip`;
      const downloadBlob = await zipWriter.close();
      console.log("done exporting to zip");
      return this.download(downloadBlob, fileName);
    },

    download(blob, filename) {
      console.log("Download started")
      // If the WritableStream is not available (Firefox, Safari), take it from the ponyfill
      if (!window.WritableStream) {
        streamsaver.WritableStream = WritableStream;
        window.WritableStream = WritableStream;
      }

      const fileStream = streamsaver.createWriteStream(filename, {
        size: blob.size
      });
      const readableStream = blob.stream();

      //  More optimized
      if (readableStream.pipeTo) {
        console.log("using direct piping");
        return readableStream.pipeTo(fileStream);
      } else {
        window.writer = fileStream.getWriter();
        const reader = readableStream.getReader();
        const pump = () =>
          reader
            .read()
            .then((res) =>
              res.done ? writer.close() : writer.write(res.value).then(pump)
            );

        pump();
      }
    },
  },
};
</script>

<template>
  <button @click="downloadFiles" id="download-files-btn">
    Click to Download File
  </button>
</template>

<style scoped>
#download-files-btn {
  background-color: dodgerblue;
  padding: 10px 15px;
  border-radius: 5px;
  outline-style: none;
  border: none;
  color: white;
  font-weight: bolder;
  margin-right: 20px;
}

#download-files-btn:active {
  background-color: white;
  color: dodgerblue;
  border: 1px solid dodgerblue;
}
</style>