<template>
  <div class="app">
    <div v-if="!image" class="input">
      <input v-if="!image" type="file" id="upload" accept="image/*" @change="handleImageUpload" />
    </div>

    <div v-if="image" class="image-container-wrapper">
      <div class="image-container">
        <div class="image-wrapper" :style="{
          padding: `${padding.top}px ${padding.right}px ${padding.bottom}px ${padding.left}px`
        }">
          <img :src="image" ref="imageElement" @load="updateImageDimensions" />
        </div>
      </div>
    </div>
  </div>

  <div v-if="image" class="controls">
    <div v-if="!independentPadding">
      <label>Horizontal : </label>
      <input type="range" min="0" max="500" v-model="padding.horizontal" @input="syncHorizontalPadding" />
      <input type="number" min="0" max="500" v-model="padding.horizontal" @input="syncHorizontalPadding" />
      <br>
      <label>Vertical : </label>
      <input type="range" min="0" max="500" v-model="padding.vertical" @input="syncVerticalPadding" />
      <input type="number" min="0" max="500" v-model="padding.vertical" @input="syncVerticalPadding" />
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
      fileName: null,
      imageDimensions: {
        width: 0,
        height: 0,
      },
      padding: {
        top: 0,
        right: 0,
        bottom: 0,
        left: 0,
        horizontal: 0,
        vertical: 0,
      },
      independentPadding: false,
    };
  },

  methods: {
    handleImageUpload(event) {
      const file = event.target.files[0];
      if (file) {
        this.fileName = file.name.split('.').slice(0, -1).join('.'); // Supprime l'extension
        this.image = URL.createObjectURL(file);
      }
    },
    updateImageDimensions() {
      const imgElement = this.$refs.imageElement;
      this.imageDimensions.width = imgElement.naturalWidth;
      this.imageDimensions.height = imgElement.naturalHeight;
    },
    syncHorizontalPadding() {
      this.padding.left = this.padding.horizontal;
      this.padding.right = this.padding.horizontal;
    },
    syncVerticalPadding() {
      this.padding.top = this.padding.vertical;
      this.padding.bottom = this.padding.vertical;
    },
    togglePaddingMode() {
      this.independentPadding = !this.independentPadding;
      if (!this.independentPadding) {
        this.padding.horizontal = this.padding.left;
        this.padding.vertical = this.padding.top;
        this.syncHorizontalPadding();
        this.syncVerticalPadding();
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

          const fileName = this.fileName ? `${this.fileName}_with-padding.png` : "image_with-padding.png";
          a.download = fileName;

          a.click();
          URL.revokeObjectURL(url);
        }
      });
    },
  },
};
</script>

<style>
* {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

body {
  background-color: #121212;
}

.app {
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.image-container-wrapper {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: calc(100vh - 250px);
}

.image-container {
  max-width: 100%;
  text-align: center;
}

.image-wrapper {
  display: inline-block;
  border: 1px solid rgb(255, 145, 0);
  padding: 0;
  background-color: rgba(255, 145, 0, 0.041);
}

img {
  display: block;
  max-width: 300px;
  padding: 0;
  border: 1px solid rgb(255, 145, 0);
  background-color: rgb(255, 255, 255);
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
  width: 50px;
  margin-left: 10px;
}

input[type="range"] {
  width: 200px;
  margin-left: 10px;
}

.input {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 30px;
  border-radius: 10px;
  margin-top: calc(50vh - 90px);
  background-color: #f8f8f8;
  box-shadow: rgba(0, 0, 0, 0.25) 0px 54px 55px, rgba(0, 0, 0, 0.12) 0px -12px 30px, rgba(0, 0, 0, 0.12) 0px 4px 6px, rgba(0, 0, 0, 0.17) 0px 12px 13px, rgba(0, 0, 0, 0.09) 0px -3px 5px;
}

input[type="file"] {
  position: relative;
  color: black;
}

input[type="file"]::file-selector-button {
  width: 121px;
  color: transparent;
}

input[type="file"]::before {
  position: absolute;
  pointer-events: none;
  top: 10px;
  left: 16px;
  height: 20px;
  width: 20px;
  content: "";
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='%23ff9100'%3E%3Cpath d='M18 15v3H6v-3H4v3c0 1.1.9 2 2 2h12c1.1 0 2-.9 2-2v-3h-2zM7 9l1.41 1.41L11 7.83V16h2V7.83l2.59 2.58L17 9l-5-5-5 5z'/%3E%3C/svg%3E");
}

input[type="file"]::after {
  position: absolute;
  pointer-events: none;
  top: 11px;
  left: 40px;
  color: rgb(255, 145, 0);
  content: "Upload File";
}

input[type="file"]::file-selector-button {
  border-radius: 8px;
  padding: 0 16px;
  height: 40px;
  cursor: pointer;
  background-color: white;
  border: 1px solid rgba(0, 0, 0, 0.16);
  box-shadow: 0px 1px 0px rgba(0, 0, 0, 0.05);
  margin-right: 16px;
  transition: background-color 200ms;
}

input[type="file"]::file-selector-button:hover {
  background-color: #f3f4f6;
}

input[type="file"]::file-selector-button:active {
  background-color: #ebe9e5;
}
</style>
