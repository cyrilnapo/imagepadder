<template>
  <div class="app">
    <input v-if="!image" type="file" accept="image/*" @change="handleImageUpload" />

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
    <button class="modeChange" @click="togglePaddingMode">
      {{ independentPadding ? 'Passer en mode uniforme' : 'Passer en mode individuel' }}
    </button>
    <button class="btnDownload" @click="downloadImage">Télécharger l'image</button>
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
        top: 0,
        right: 0,
        bottom: 0,
        left: 0,
        all: 0,
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
  display: inline-block;
  border: 2px dashed rgb(51, 164, 184);
  padding: 0; 
}

img {
  display: block;
  max-width: 100%;
  padding: 0;
}


.controls {
  align-items: center;
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
  padding: 10px 20px;
  font-size: 16px;
}

.btnDownload {
  background-color: #4c8baf;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.modeChange {
  background-color: #f0f0f0;
  color: #333;
  border: none;
  border-radius: 5px;
  cursor: pointer;
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
