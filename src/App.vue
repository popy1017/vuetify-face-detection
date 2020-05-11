<template>
  <v-app>
    <v-container v-cloak @drop.prevent="addDropFile" @dragover.prevent>
      <v-file-input
        v-model="file"
        @change="setImage"
        @click:clear="clearCanvas"
        accept="image/png, image/jpeg, image/bmp"
        prepend-icon="mdi-camera"
        placeholder="画像をドラッグ＆ドロップするか選択してください。"
      ></v-file-input>
      <v-row justify="center" align="center">
        <canvas :width="canvasWidth" height="450" ref="preview"></canvas>
      </v-row>
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
    canvasWidth: 800,
  }),
  methods: {
    addDropFile(e) {
      this.file = e.dataTransfer.files[0];
    },
    setImage() {
      this.clearCanvas();
      console.log("setImage");
      if (this.file && this.file.type.startsWith("image/")) {
        const url = window.URL.createObjectURL(this.file);
        const img = new window.Image();
        img.src = url;
        img.onload = () => {
          const canvas = this.$refs.preview;
          const context = canvas.getContext("2d");
          console.log(canvas.width);
          if (img.width > canvas.width) {
            const scale = canvas.width / img.width;
            const sw = img.width;
            const sh = img.height;
            const dw = img.width * scale;
            const dh = img.height * scale;
            canvas.width = dw;
            canvas.height = dh;
            context.drawImage(img, 0, 0, sw, sh, 0, 0, dw, dh);
          } else {
            context.drawImage(img, 0, 0);
          }
        };
      }
    },
    clearCanvas() {
      const canvas = this.$refs.preview;
      const context = canvas.getContext("2d");
      context.clearRect(0, 0, canvas.width, canvas.height);
    },
  },
};
</script>
