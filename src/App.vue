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
      <v-btn
        class="my-2"
        color="primary"
        :disabled="!file"
        block
        @click="detectFaces(file)"
        >Detect!!</v-btn
      >
      <v-row justify="center" align="center">
        <canvas :width="canvasWidth" height="450" ref="preview"></canvas>
      </v-row>
      <v-overlay :value="overlay">
        <v-progress-circular indeterminate size="64"></v-progress-circular>
      </v-overlay>
    </v-container>
  </v-app>
</template>

<script>
import detectFaces from "./libs/visionApi";

export default {
  name: "App",
  data: () => ({
    file: null,
    rules: [
      (value) =>
        !value ||
        value.size < 2000000 ||
        "Image size should be less than 2 MB!",
    ],
    canvasWidth: 800,
    imgWidth: 800,
    results: {},
    overlay: false,
  }),
  methods: {
    addDropFile(e) {
      this.file = e.dataTransfer.files[0];
      this.setImage();
    },
    setImage() {
      this.clearCanvas();
      console.log("setImage");
      if (this.file && this.file.type.startsWith("image/")) {
        const url = window.URL.createObjectURL(this.file);
        const img = new window.Image();
        img.src = url;
        img.onload = () => {
          this.imgWidth = img.width;
          const canvas = this.$refs.preview;
          const context = canvas.getContext("2d");
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
    detectFaces: async function() {
      this.overlay = true;
      try {
        const data = await detectFaces(this.file);

        if (data && data.responses) {
          const faceAnnotations = data.responses[0].faceAnnotations;
          const boundingPolies = [];
          faceAnnotations.forEach((annotation) => {
            boundingPolies.push(annotation.boundingPoly);
          });
          this.results = boundingPolies;
          this.drawRects();
        }
        this.overlay = false;
      } catch (error) {
        console.log(error);
        this.overlay = false;
      }
    },
    getRects() {
      let rects = [];

      this.results.forEach((poly) => {
        const xArray = poly.vertices.map((v) => v.x);
        const yArray = poly.vertices.map((v) => v.y);

        const xmin = Math.min.apply(Math, xArray);
        const xmax = Math.max.apply(Math, xArray);
        const ymin = Math.min.apply(Math, yArray);
        const ymax = Math.max.apply(Math, yArray);
        const width = xmax - xmin;
        const height = ymax - ymin;

        rects.push({
          x: xmin,
          y: ymin,
          width,
          height,
        });
      });

      return rects;
    },
    drawRects() {
      const canvas = this.$refs.preview;
      const context = canvas.getContext("2d");
      const rects = this.getRects();
      context.strokeStyle = "rgb(0, 255, 0)";
      context.lineWidth = 2;
      rects.forEach((rect) => {
        let scale = 1;
        if (this.imgWidth > canvas.width) {
          scale = canvas.width / this.imgWidth;
        }
        context.strokeRect(
          rect.x * scale,
          rect.y * scale,
          rect.width * scale,
          rect.height * scale
        );
      });
    },
  },
};
</script>
