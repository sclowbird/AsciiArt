<template>
  <div class="hello">
    <img id="ascii-logo" alt="Vue logo" src="../assets/logo.png">
    <div class="input-container">
      <label for="input">Choose an image</label>
      <input type="file" id="input" @change="loadImageData">
    </div>

    <img hidden id="my-image">
    <div id="ascii-container">
      <div id="ascii" v-for="(item, index) in asciiObj" :key="index">
        <p class="overflow-visible">{{ item.row }}</p>
      </div>
    </div>
    <canvas hidden id="image-canvas"></canvas>
    <canvas id="phone-canvas" width="320" height="700"></canvas>

    <p>
      <i>Uploaded images are not saved. Image orientation on phones may be flipped.</i>
    </p>
  </div>
</template>

<script>
export default {
  name: "AsciiArt",
  data() {
    return {
      imageWidth: 0,
      imageHeight: 0,
      asciiObj: [{ row: " " }]
    };
  },
  props: {
    msg: String
  },
  methods: {
    loadImageData(e) {
      this.reInitialize();
      let imageFile = e.target.files;
      if (!imageFile.length) return;
      this.readImageData(imageFile[0]);
    },

    readImageData(imageFile) {
      let fr = new FileReader();
      // onload event gets triggered after content of readAsDataUrl is ready
      fr.onload = () => this.getImageDataResult(fr);
      fr.readAsDataURL(imageFile);
    },

    getImageDataResult(fileReader) {
      let img = document.getElementById("my-image");
      img.onload = () => this.processImage(img);
      img.src = fileReader.result;
    },

    processImage(img) {
      let ctx = this.getCanvasContext();
      this.drawImageToCanvas(img, ctx);
      let imageColors = this.getImageColors(img, ctx);
      let pixelTuples = this.createPixelTuples(imageColors);
      let pixelsWithoutAlpha = this.removeAlphaChannel(pixelTuples);
      let pixelBrightness = this.convertPixelToBrightness(pixelsWithoutAlpha);
      let asciiString = this.mapBrigthnessToAscii(pixelBrightness);
      this.addLineBreaks(asciiString);
      this.convertObjIntoImg();
    },

    getCanvasContext() {
      let canvas = this.createCanvas();
      let ctx = canvas.getContext("2d");
      return ctx;
    },

    createCanvas() {
      //let canvas = document.createElement("canvas");
      let canvas = document.getElementById("image-canvas");
      return canvas;
    },

    drawImageToCanvas(img, ctx) {
      const maximumWidth = 100;
      const originalPixelSize = img.height * img.width;

      this.imageWidth = img.width;
      this.imageHeight = img.height;

      let scalingFactor = maximumWidth / this.imageWidth;

      this.imageWidth = Math.round(this.imageWidth * scalingFactor);
      this.imageHeight = Math.round(this.imageHeight * scalingFactor);

      ctx.drawImage(
        img,
        0,
        0,
        img.width,
        img.height,
        0,
        0,
        this.imageWidth,
        this.imageHeight
      );
    },

    getImageColors(img, ctx) {
      let imgColors = ctx.getImageData(0, 0, this.imageWidth, this.imageHeight)
        .data;

      this.imageWidth = Math.floor(this.imageWidth * 3);

      let imgColorArray = Object.values(imgColors);
      return imgColorArray;
    },

    createPixelTuples(imgColorArray) {
      let pixelTuples = [];
      let counter = 0;
      for (let i = 0; i < imgColorArray.length / 4; i++) {
        let tempArray = [];
        if (i != 0) {
          counter += 3;
        }
        for (let j = 0; j < 4; j++) {
          tempArray.push(imgColorArray[i + j + counter]);
        }
        pixelTuples.push(tempArray);
      }
      return pixelTuples;
    },

    removeAlphaChannel(pixelTuples) {
      for (let i = 0; i < pixelTuples.length; i++) {
        pixelTuples[i].splice(3, 1);
      }
      return pixelTuples;
    },

    convertPixelToBrightness(pixelTuples) {
      for (let i = 0; i < pixelTuples.length; i++) {
        let rgbAverage = 0;
        for (let j = 0; j < 1; j++) {
          rgbAverage += pixelTuples[i][0] * pixelTuples[i][0] * 0.241;
          rgbAverage += pixelTuples[i][1] * pixelTuples[i][1] * 0.691;
          rgbAverage += pixelTuples[i][2] * pixelTuples[i][2] * 0.068;
        }

        rgbAverage = Math.sqrt(rgbAverage);

        pixelTuples[i] = rgbAverage;
      }
      return pixelTuples;
    },

    createAsciiCharacters() {
      let asciiChars =
        //'.`^",:;Il!i~+_-?][}{1)(|\\/tfjrxnuvczXYUJCLQ0OZmwqpdbkhao*#MW&8%B@$';
        '$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/|()1{}[]?-_+~i!lI;:,"^`.';
      return asciiChars;
    },

    mapBrigthnessToAscii(pixelBrightness) {
      const maxBrigthness = 255;
      const conversionFactor = 4;
      let asciiToString = "";
      let asciiChars = this.createAsciiCharacters();
      let counter = 0;
      for (let i = 0; i < pixelBrightness.length; i++) {
        let correspondingAsciiChar = Math.round(
          pixelBrightness[i] / conversionFactor
        );

        pixelBrightness[i] = asciiChars[correspondingAsciiChar];
        for (let j = 0; j < 3; j++) {
          asciiToString += pixelBrightness[i];
        }
      }
      return asciiToString;
    },

    addLineBreaks(asciiString) {
      let rowStrings = "";
      for (let i = 1; i < asciiString.length + 1; i++) {
        rowStrings += asciiString[i - 1];
        if (i % this.imageWidth == 0) {
          this.asciiObj.push({ row: rowStrings });
          rowStrings = "";
        }
      }
    },

    reInitialize() {
      this.imageWidth = 0;
      this.imageHeight = 0;
      this.asciiObj = [{ row: " " }];
    },

    convertObjIntoImg() {
      let canvas = document.getElementById("phone-canvas");
      let ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.font = "2px Courier";
      let a = [];
      let y = 10;

      for (let prop in this.asciiObj) {
        a.push(this.asciiObj[prop]);
      }

      for (let i = 1; i < a.length; i++) {
        ctx.fillText(a[i].row, 0, y);
        y += 5;
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#ascii-logo {
  margin-top: 30px;
  margin-bottom: 50px;
}

.input-container {
  margin-bottom: 50px;
}

input {
  width: 0.1px;
  height: 0.1px;
  opacity: 0;
  overflow: hidden;
  position: absolute;
  z-index: -1;
}

label {
  font-size: 1em;
  color: white;
  width: 90px;
  height: 20px;
  padding: 5px;
  background: #232b53;
  border-radius: 3px;
  cursor: pointer;
  margin-bottom: 30px;
}

label:hover {
  background-color: rgba(82, 1, 1, 0.317);
}

.hello {
  display: flex;
  flex-direction: column;
  align-items: center;
}

#ascii-container {
  border: 3px #232b53 solid;
  border-radius: 5px;
  background: rgba(255, 255, 255, 0.337);
}

@media only screen and (max-width: 450px) {
  #ascii-container {
    border: 3px #232b53 solid;
    border-radius: 5px;
    background: rgba(255, 255, 255, 0.337);
    display: none;
  }
}

@media only screen and (max-height: 450px) {
  #ascii-container {
    border: 3px #232b53 solid;
    border-radius: 5px;
    background: rgba(255, 255, 255, 0.337);
    display: none;
  }
}

@media only screen and (min-width: 451px) {
  #phone-canvas {
    visibility: hidden;
    display: none;
  }
}

@media only screen and (min-width: 451px) and (max-height: 450px) {
  #phone-canvas {
    visibility: visible;
  }
}

@media only screen and (max-width: 450px) {
  .overflow-visible {
    font-family: "Press Start 2P", monospace;
    color: black;
    font-size: 1px;
    font-weight: bolder;
  }
}

@media only screen and (max-height: 450px) {
  .overflow-visible {
    font-family: "Press Start 2P", monospace;
    color: black;
    font-size: 1px;
    font-weight: bolder;
  }
}

@media only screen and (min-width: 451px) {
  .overflow-visible {
    font-family: "Press Start 2P", monospace;
    color: black;
    font-size: 2px;
    font-weight: bolder;
  }
}
</style>
