<template>
  <v-app>
    <v-container v-cloak @drop.prevent="addDropFile" @dragover.prevent>
      <v-file-input
        v-model="file"
        accept="image/png, image/jpeg, image/bmp"
        prepend-icon="mdi-camera"
        placeholder="画像をドラッグ＆ドロップか選択してください。"
      ></v-file-input>
      <canvas></canvas>
    </v-container>
  </v-app>
</template>

<script>
export default {
  name: "App",
  data: () => ({
    file: null,
    rules: [
      (value) =>
        !value ||
        value.size < 2000000 ||
        "Avatar size should be less than 2 MB!",
    ],
  }),
  methods: {
    addDropFile(e) {
      this.file = e.dataTransfer.files[0];
      console.log(this.file);
    },
  },
};
</script>
