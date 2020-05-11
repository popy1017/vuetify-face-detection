<template>
  <v-app>
    <v-container v-cloak @drop.prevent="addDropFile" @dragover.prevent>
      <v-file-input
        v-model="file"
        @change="setImage"
        @click:clear="clearImage"
        accept="image/png, image/jpeg, image/bmp"
        prepend-icon="mdi-camera"
        placeholder="画像をドラッグ＆ドロップか選択してください。"
      ></v-file-input>
      <v-row justify="center" align="center">
        <canvas width="500" height="500" ref="preview"></canvas>
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
    ctx: null,
  }),
  methods: {
    addDropFile(e) {
      this.file = e.dataTransfer.files[0];
    },
    setImage() {
      this.clearImage();
      console.log("setImage");
      if (this.file && this.file.type.startsWith("image/")) {
        const url = window.URL.createObjectURL(this.file);
        const img = new window.Image();
        img.src = url;
        img.onload = () => {
          console.log(img.width);
          this.ctx = this.$refs.preview.getContext("2d");
          const canvas = this.$refs.preview;

          if (img.width > 500) {
            const scale = 500 / img.width;
            const dstWidth = img.width * scale;
            const dstHeight = img.height * scale;
            canvas.width = dstWidth;
            canvas.height = dstHeight;
            this.ctx.drawImage(
              img,
              0,
              0,
              img.width,
              img.height,
              0,
              0,
              dstWidth,
              dstHeight
            );
            this.ctx.strokeRect(0, 0, 100, 100);
          } else {
            this.ctx.drawImage(img, 0, 0);
          }
        };
      }
    },
    clearImage() {
      console.log("clear image");
      const canvas = this.$refs.preview;
      console.dir(canvas);
      const result = this.ctx.clearRect(0, 0, canvas.width, canvas.height);
      console.log(result);
    },
  },
  mounted() {
    this.ctx = this.$refs.preview.getContext("2d");
  },
};
</script>
