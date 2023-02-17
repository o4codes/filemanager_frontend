<script>
import FolderButton from "./components/FolderButton.vue";
import FileButton from "./components/FileButton.vue";

export default {
  data() {
    return {
      BASE_URL: "http://127.0.0.1:8000/api/v1/drive",
      FOLDER_ID: "199c7161-ceea-4c51-8ddb-36156b524d14",
      FILES: [
        "08a17bd3-6070-476b-a688-6b5e0a7f1c06",
        "2e7926c5-f341-44cf-b4f2-450774ab43fe",
      ],
    };
  },
  components: { FolderButton, FileButton },
};
</script>    

<template>
  <main>
    <div class="container">
      <FolderButton :folderId="FOLDER_ID"  :baseUrl="BASE_URL"/>
      <FileButton :files="FILES" :baseUrl="BASE_URL"/>
    </div>
    <div id="link-container" class="container"></div>
  </main>
</template>

<style scoped>
* {
  margin: 0px;
  padding: 0px;
  box-sizing: border-box;
}

main {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

a {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  font-size: 14px;
}

.container {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>