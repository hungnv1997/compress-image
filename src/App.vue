<template>
  <div class="image-compressor">
    <input
      type="file"
      @change="onChange"
      class="image-compressor__input-file"
    />
    <input
      type="range"
      v-model="compressionLevel"
      min="1"
      max="100"
      class="image-compressor__input-range"
    />
    <input
      type="range"
      v-model="scale"
      min="0.1"
      max="2"
      step="0.1"
      class="image-compressor__input-scale"
    />
    <div class="image-compressor__dimensions">
      <label
        >Width:
        <input
          type="number"
          v-model="canvasWidth"
          class="image-compressor__input-dimension"
        />
      </label>
      <label
        >Height:
        <input
          type="number"
          v-model="canvasHeight"
          class="image-compressor__input-dimension"
        />
      </label>
    </div>
    <canvas
      ref="previewCanvas"
      :width="canvasWidth"
      :height="canvasHeight"
      class="image-compressor__canvas"
    ></canvas>
    <p class="image-compressor__info">
      Compressed Image Size: {{ compressedImageSize }} bytes
    </p>
    <div class="image-compressor__download">
      <label>Download Format:</label>
      <select v-model="downloadFormat" class="image-compressor__select">
        <option value="jpeg">JPEG</option>
        <option value="png">PNG</option>
        <option value="webp">WebP</option>
      </select>
      <button @click="downloadImage" class="image-compressor__button">
        Download Compressed Image
      </button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      file: null,
      compressionLevel: 50,
      scale: 1,
      compressedImageSize: 0,
      compressedDataUrl: "",
      canvasWidth: 0,
      canvasHeight: 0,
      downloadFormat: "jpeg", // Default download format
    };
  },
  watch: {
    compressionLevel(newVal) {
      if (this.file) {
        this.drawPreview(this.file, newVal, this.scale);
      }
    },
    canvasWidth() {
      if (this.file) {
        this.drawPreview(this.file, this.compressionLevel, this.scale);
      }
    },
    canvasHeight() {
      if (this.file) {
        this.drawPreview(this.file, this.compressionLevel, this.scale);
      }
    },
    scale(newVal) {
      if (this.file) {
        this.drawPreview(this.file, this.compressionLevel, newVal);
      }
    },
  },
  methods: {
    onChange(e) {
      const file = e.target.files[0];
      if (!file) return;

      // Validate file type (optional)
      const validTypes = ["image/jpeg", "image/png"];
      if (!validTypes.includes(file.type)) {
        console.error("Unsupported file type. Please upload an image.");
        return;
      }

      // Read the file and update the canvas
      const reader = new FileReader();
      reader.onload = this.handleFileLoad;
      reader.readAsDataURL(file);
    },
    handleFileLoad(e) {
      const imgSrc = e.target.result;
      this.file = imgSrc;

      const img = new Image();
      img.onload = () => {
        this.canvasWidth = img.width * this.scale;
        this.canvasHeight = img.height * this.scale;
        this.drawPreview(imgSrc, this.compressionLevel, this.scale);
      };
      img.src = imgSrc;
    },
    drawPreview(imgSrc, compressionLevel, scale) {
      const canvas = this.$refs.previewCanvas;
      const ctx = canvas.getContext("2d");
      const img = new Image();
      img.src = imgSrc;

      img.onload = () => {
        const drawWidth = img.width * scale;
        const drawHeight = img.height * scale;
        this.canvasWidth = drawWidth;
        this.canvasHeight = drawWidth;
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.drawImage(
          img,
          0,
          0,
          img.width,
          img.height,
          0,
          0,
          drawWidth,
          drawHeight,
        );
        this.compressImage(canvas, compressionLevel);
      };
    },
    compressImage(canvas, compressionLevel) {
      const compressionQuality = compressionLevel / 100;
      canvas.toBlob(
        (blob) => {
          this.compressedImageSize = blob.size;
          this.compressedDataUrl = URL.createObjectURL(blob);

          // Optionally, you can draw the compressed image back on the canvas
          const compressedImg = new Image();
          compressedImg.src = this.compressedDataUrl;
          compressedImg.onload = () => {
            const ctx = canvas.getContext("2d");
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.drawImage(compressedImg, 0, 0, canvas.width, canvas.height);
          };
        },
        `image/${this.downloadFormat}`,
        compressionQuality,
      );
    },
    downloadImage() {
      const link = document.createElement("a");
      link.href = this.compressedDataUrl;
      link.download = `compressed_image.${this.downloadFormat}`;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
  },
};
</script>

<style lang="scss">
.image-compressor {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);

  &__input-file,
  &__input-range,
  &__input-scale,
  &__select {
    margin: 10px 0;
    padding: 10px;
    border-radius: 5px;
    border: 1px solid #ccc;
    font-size: 16px;
  }

  &__dimensions {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;

    label {
      display: flex;
      flex-direction: column;
      align-items: center;
      font-size: 14px;
      color: #555;

      .image-compressor__input-dimension {
        width: 80px;
        margin-top: 5px;
        padding: 5px;
        border-radius: 5px;
        border: 1px solid #ccc;
        font-size: 16px;
      }
    }
  }

  &__canvas {
    border: 1px solid #ccc;
    border-radius: 10px;
    margin: 20px 0;
  }

  &__info {
    font-size: 16px;
    margin: 10px 0;
    color: #555;
  }

  &__download {
    display: flex;
    align-items: center;
    margin-top: 20px;

    label {
      font-size: 16px;
      margin-right: 10px;
    }

    .image-compressor__select {
      width: 100px;
    }

    .image-compressor__button {
      padding: 10px 20px;
      background-color: #007bff;
      color: #fff;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-size: 16px;
      transition: background-color 0.3s;

      &:hover {
        background-color: #0056b3;
      }
    }
  }
}
</style>
