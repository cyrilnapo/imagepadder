<template>
  <div class="app">
    <h1>Image Padding Tool</h1>

    <input type="file" accept="image/*" @change="handleImageUpload" />

    <div v-if="image" class="image-container">
      <div
        class="image-wrapper"
        :style="{
          padding: `${padding.top}px ${padding.right}px ${padding.bottom}px ${padding.left}px`
        }"
      >
        <img :src="image" ref="imageElement" @load="updateImageDimensions" />
      </div>
    </div>

    <button @click="downloadImage">Télécharger l'image</button>
  </div>

  <div v-if="image" class="controls">
    <div v-if="!independentPadding">
      <label>Padding uniforme: {{ padding.all }}</label>
      <input type="range" min="0" max="500" v-model="padding.all" @input="syncPadding" />
      <input type="number" min="0" max="500" v-model="padding.all" @input="syncPadding" />
    </div>

    <div v-if="independentPadding">
      <div>
        <label>Top: {{ padding.top }}</label>
        <input type="range" min="0" max="500" v-model="padding.top" />
        <input type="number" min="0" max="500" v-model="padding.top" />
      </div>
      <div>
        <label>Right: {{ padding.right }}</label>
        <input type="range" min="0" max="500" v-model="padding.right" />
        <input type="number" min="0" max="500" v-model="padding.right" />
      </div>
      <div>
        <label>Bottom: {{ padding.bottom }}</label>
        <input type="range" min="0" max="500" v-model="padding.bottom" />
        <input type="number" min="0" max="500" v-model="padding.bottom" />
      </div>
      <div>
        <label>Left: {{ padding.left }}</label>
        <input type="range" min="0" max="500" v-model="padding.left" />
        <input type="number" min="0" max="500" v-model="padding.left" />
      </div>
    </div>

    <button @click="togglePaddingMode">
      {{ independentPadding ? 'Passer en mode uniforme' : 'Passer en mode individuel' }}
    </button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      image: null,
      imageDimensions: {
        width: 0,
        height: 0,
      },
      padding: {
        top: 10,
        right: 10,
        bottom: 10,
        left: 10,
        all: 10,
      },
      independentPadding: false,
    };
  },
  methods: {
    handleImageUpload(event) {
      const file = event.target.files[0];
      if (file) {
        this.image = URL.createObjectURL(file);
      }
    },
    updateImageDimensions() {
      const imgElement = this.$refs.imageElement;
      this.imageDimensions.width = imgElement.naturalWidth;
      this.imageDimensions.height = imgElement.naturalHeight;
      console.log("Image dimensions:", this.imageDimensions);
    },
    syncPadding() {
      this.padding.top = this.padding.all;
      this.padding.right = this.padding.all;
      this.padding.bottom = this.padding.all;
      this.padding.left = this.padding.all;
    },
    togglePaddingMode() {
      this.independentPadding = !this.independentPadding;

      if (!this.independentPadding) {
        this.padding.all = this.padding.top;
        this.syncPadding();
      }
    },
    downloadImage() {
      const imgElement = this.$refs.imageElement;
      if (!imgElement || !this.imageDimensions.width || !this.imageDimensions.height) {
        console.error("Image dimensions not loaded yet.");
        return;
      }

      const canvas = document.createElement("canvas");
      const context = canvas.getContext("2d");

      const width = Number(this.imageDimensions.width) + Number(this.padding.left) + Number(this.padding.right);
      const height = Number(this.imageDimensions.height) + Number(this.padding.top) + Number(this.padding.bottom);

      console.log("Canvas dimensions:", { width, height });

      canvas.width = width;
      canvas.height = height;

      context.clearRect(0, 0, canvas.width, canvas.height);

      context.drawImage(
        imgElement,
        this.padding.left,
        this.padding.top,
        this.imageDimensions.width,
        this.imageDimensions.height
      );

      canvas.toBlob((blob) => {
        if (blob) {
          const url = URL.createObjectURL(blob);
          const a = document.createElement("a");
          a.href = url;
          a.download = "image_with_padding.png";
          a.click();
          URL.revokeObjectURL(url);
        } else {
          console.error("Failed to create blob. Canvas drawing might have failed.");
        }
      });
    },
  },
};
</script>

<style>
.app {
  text-align: center;
  padding: 20px;
  padding-bottom: 120px;
}

.image-container {
  display: inline-block;
  position: relative;
}

.image-wrapper {
  background-color: rgba(0, 0, 0, 0.1);
  display: inline-block;
}

img {
  display: block;
  max-width: 100%;
}

.controls {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  background: #fff;
  padding: 20px;
  border-radius: 10px 10px 0 0;
  box-shadow: 0 -4px 6px rgba(0, 0, 0, 0.1);
  z-index: 10;
  text-align: left;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

button {
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 16px;
}

input[type="number"] {
  width: 60px;
  margin-left: 10px;
}

input[type="range"] {
  width: 100px;
  margin-left: 10px;
}
</style>
