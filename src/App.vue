<template>
  <div id="app">
    <div class="header">text editor</div>
    <div class="main-wrapper">
      <div class="can-wrapper">
        <canvas ref="can" width="670" height="670"></canvas>
      </div>
      <div class="text-layer-wrapper">
        <div class="text-layer-title">Text Layer</div>
        <div class="text-options">
          <div class="input-wrapper">
            <label>Enter text</label>
            <br />
            <textarea
              :class="{
                'main-text': true,
                'is-disabled': hasActiveObject,
              }"
              type="text"
              v-model="text"
            />
          </div>
          <div class="size-controls">
            <div class="input-wrapper">
              <label>Font size (px)</label>
              <br />
              <input type="number" v-model="fontSize" @change="setFontSize" />
            </div>
            <div class="input-wrapper">
              <label>Line height (px)</label>
              <br />
              <input
                type="number"
                v-model="lineHeight"
                @change="setLineHeight"
              />
            </div>
            <div class="input-wrapper zoom">
              <label>Zoom</label>
              <br />
              <input disabled type="text" v-model="zoom" />
            </div>
          </div>
          <div class="select-wrapper">
            <label>Font family</label>
            <br />
            <v-select
              @input="setFontFamily"
              v-model="selectedFont"
              :options="fonts"
            />
          </div>
          <div
            :class="{
              'is-disabled': buttonDisabled,
              'add-btn': true,
            }"
            @click="addTextLayer"
          >
            Add text
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import { fabric } from "fabric";

  const defaultValues = {
    fontSize: 14,
    lineHeight: 14,
    selectedFont: "Arial",
  };

  export default {
    name: "App",
    data() {
      return {
        canvas: null,
        activeObject: null,
        text: "",
        fontSize: defaultValues.fontSize,
        lineHeight: defaultValues.lineHeight,
        selectedFont: defaultValues.selectedFont,
        fonts: [
          "Arial",
          "Verdana",
          "Helvetica",
          "Tahoma",
          "Trebuchet MS",
          "Times New Roman",
          "Georgia",
          "Garamond",
          "Courier New",
          "Brush Script MT",
        ],
        zoom: "x1",
      };
    },
    mounted() {
      const ref = this.$refs.can;
      this.canvas = new fabric.Canvas(ref, {
        backgroundColor: "#fff",
      });
      this.canvas.on({
        "selection:updated": this.handleElement,
        "selection:created": this.handleElement,
        "before:selection:cleared": this.elementDeslected,
        "object:scaled": this.objectResize,
      });
    },
    methods: {
      addTextLayer() {
        const text = new fabric.Text(this.text, {
          left: 100,
          top: 100,
          fontSize: +this.fontSize,
          fontFamily: this.selectedFont,
          lineHeight: +this.lineHeight / +this.fontSize,
        });
        this.canvas.add(text);
      },
      setFontSize() {
        if (!this.hasActiveObject) {
          return;
        }
        this.canvas.getActiveObject().set("fontSize", +this.fontSize);
        this.canvas
          .getActiveObject()
          .set("lineHeight", +this.lineHeight / +this.fontSize);
        this.canvas.requestRenderAll();
      },
      setLineHeight() {
        if (!this.hasActiveObject) {
          return;
        }
        // this way we make sure to preserve lineHeight in units relative to fontSize
        this.canvas
          .getActiveObject()
          .set("lineHeight", +this.lineHeight / +this.fontSize);
        this.canvas.requestRenderAll();
      },
      setFontFamily() {
        if (!this.hasActiveObject) {
          return;
        }
        this.canvas.getActiveObject().set("fontFamily", this.selectedFont);
        this.canvas.requestRenderAll();
      },
      handleElement(obj) {
        this.activeObject = obj.target;
        this.text = obj.target.text;
        this.fontSize = obj.target.fontSize;
        // this way we make sure to keep lineHeight displayed in px
        this.lineHeight = obj.target.lineHeight * obj.target.fontSize;
        this.zoom = `${"x" + obj.target.zoomY}`;
      },
      objectResize(obj) {
        //forgive me for using setTimeout here :)
        setTimeout(() => {
          this.zoom = `${
            "x" + Math.round((obj.target.zoomY + Number.EPSILON) * 100) / 100
          }`;
        }, 200);
      },
      elementDeslected() {
        this.activeObject = null;
        this.text = "";
      },
    },
    computed: {
      buttonDisabled() {
        return (
          this.hasActiveObject ||
          !this.text ||
          !this.text.trim() ||
          +this.fontSize < 1 ||
          +this.lineHeight < 1
        );
      },
      hasActiveObject() {
        return !!this.activeObject;
      },
    },
  };
</script>

<style>
  body {
    margin: 0;
    padding: 0;
    color: #818181;
    font-family: "Arial";
  }
  .header {
    background-color: #43ce93;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    padding: 30px;
    text-align: center;
    color: #fff;
    font-size: 20px;
    font-weight: bold;
  }
  .main-wrapper {
    display: flex;
    height: 100vh;
    padding-top: 83px;
    box-sizing: border-box;
  }
  .main-wrapper .can-wrapper {
    background-color: #e5e5e5;
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
  }
  .can-wrapper canvas {
    border: 1px solid black;
  }
  .main-wrapper .text-layer-wrapper {
    width: 280px;
    padding: 40px 24px;
    background-color: #fff;
    -webkit-box-shadow: -4px 0px 12px 2px #c3c3c3;
    box-shadow: -4px 0px 12px 2px #c3c3c3;
  }
  .main-wrapper .text-layer-wrapper .text-layer-title {
    border-bottom: 1px solid #cdcdcd;
    padding-bottom: 8px;
    margin-bottom: 15px;
    font-size: 18px;
    font-weight: bold;
  }
  .main-text {
    box-sizing: border-box;
    width: 100%;
  }
  .main-text.is-disabled {
    opacity: 0.5;
    pointer-events: none;
  }
  label {
    font-size: 14px;
  }
  .main-wrapper .text-layer-wrapper .add-btn {
    cursor: pointer;
    background-color: #43ce93;
    box-sizing: border-box;
    text-align: center;
    margin-top: 25px;
    border-radius: 3px;
    color: #fff;
    font-weight: bold;
    display: inline-block;
    padding: 12px 16px;
    width: 100%;
    transition: 0.2s all;
  }
  .main-wrapper .text-layer-wrapper .add-btn:hover {
    background-color: #3c9e75;
  }
  .main-wrapper .text-layer-wrapper .add-btn.is-disabled {
    opacity: 0.5;
    pointer-events: none;
  }
  .size-controls {
    display: flex;
  }
  .size-controls .input-wrapper {
    width: 40%;
  }
  .size-controls .input-wrapper.zoom {
    width: 20%;
  }
  .size-controls .input-wrapper.zoom input {
    width: 100%;
    box-sizing: border-box;
  }
  .size-controls input {
    width: 80px;
  }
  .input-wrapper input,
  .input-wrapper textarea {
    font-family: Arial, Helvetica, sans-serif;
    border-radius: 5px;
    border: 1px solid #cdcdcd;
    padding: 5px;
    margin-top: 5px;
    margin-bottom: 12px;
    outline: none;
    font-size: 16px;
  }
  .select-wrapper .v-select {
    margin-top: 5px;
  }
</style>
