<template>
  <h1 class="title">Compress Image</h1>
  <div class="image-compressor">
    <input
      type="file"
      @change="onChange"
      class="image-compressor__input-file"
    />
    <div class="image-compressor__image-import">
      <canvas
        ref="previewCanvas"
        :width="canvasWidth"
        :height="canvasHeight"
        class="image-compressor__canvas"
      ></canvas>
    </div>
    <div class="image-compressor__download">
      <div class="image-compressor__config">
        <div class="image-compressor__config__range">
          <p class="image-compressor__config__range__title">Compress:</p>
          <input
            type="range"
            v-model="compressionLevel"
            min="1"
            max="100"
            class="image-compressor__config__input-range"
          />
        </div>
        <div class="image-compressor__config__range">
          <p class="image-compressor__config__range__title">Resize:</p>
          <input
            type="range"
            v-model="scale"
            min="0.1"
            max="2"
            step="0.1"
            class="image-compressor__config__input-scale"
          />
        </div>
        <div class="image-compressor__config__dimensions">
          <label
            >Width:
            <input
              type="number"
              v-model="canvasWidth"
              class="image-compressor__config__input-dimension"
            />
          </label>
          <label
            >Height:
            <input
              type="number"
              v-model="canvasHeight"
              class="image-compressor__config__input-dimension"
            />
          </label>
        </div>
      </div>
      <p class="image-compressor__info">
        Image Size: {{ compressedImageSize }} bytes
      </p>
      <label>Download Format:</label>
      <select v-model="downloadFormat" class="image-compressor__select">
        <option value="jpeg">JPEG</option>
        <option value="png">PNG</option>
        <option value="webp">WebP</option>
      </select>
      <button @click="downloadImage" class="image-compressor__button">
        Download
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
        this.canvasHeight = drawHeight;
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
.title {
  color: #333;
}
.image-compressor {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  padding: 1rem;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

  &__input-file {
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    background-color: #fff;
    cursor: pointer;
  }
  &__image-import {
    max-width: 768px;
    min-width: 768px;
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 1rem;
  }
  &__canvas {
    max-width: 100%;
  }

  &__info {
    font-size: 0.9rem;
    color: #666;
  }

  &__download {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;

    label {
      font-size: 0.9rem;
      font-weight: 600;
      color: #333;
    }

    .image-compressor__select {
      padding: 0.5rem;
      border: 1px solid #ccc;
      border-radius: 4px;
      outline: none;
      transition: border-color 0.3s;

      &:focus {
        border-color: #4caf50;
      }
    }

    .image-compressor__button {
      padding: 0.75rem 1rem;
      background-color: #4caf50;
      color: #fff;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      transition: background-color 0.3s;

      &:hover {
        background-color: #45a049;
      }
    }
  }

  &__config {
    display: flex;
    flex-direction: column;
    gap: 1rem;

    &__range {
      display: flex;
      align-items: center;
      gap: 1rem;

      &__title {
        font-size: 1rem;
        font-weight: 600;
        color: #333;
        min-width: 76px;
        max-width: 76px;
      }

      .image-compressor__config__input-range,
      .image-compressor__config__input-scale {
        flex-grow: 1;
        -webkit-appearance: none;
        width: 100%;
        height: 6px;
        background: #ddd;
        border-radius: 5px;
        outline: none;
        transition: background 0.3s;

        &::-webkit-slider-thumb {
          -webkit-appearance: none;
          appearance: none;
          width: 16px;
          height: 16px;
          background: #4caf50;
          border-radius: 50%;
          cursor: pointer;
        }

        &::-moz-range-thumb {
          width: 16px;
          height: 16px;
          background: #4caf50;
          border-radius: 50%;
          cursor: pointer;
        }
      }
    }

    &__dimensions {
      display: flex;
      flex-wrap: wrap;
      gap: 1rem;

      label {
        display: flex;
        flex-direction: column;
        font-size: 0.9rem;
        font-weight: 600;
        color: #333;

        .image-compressor__config__input-dimension {
          margin-top: 0.5rem;
          padding: 0.5rem;
          border: 1px solid #ccc;
          border-radius: 4px;
          outline: none;
          font-size: 1rem;
          transition: border-color 0.3s;

          &:focus {
            border-color: #4caf50;
          }
        }
      }
    }
  }

  @media (min-width: 768px) {
    flex-direction: row;

    &__config {
      flex: 1;
    }

    &__download {
      flex: 1;
      align-items: flex-start;
    }
  }
}
</style>
