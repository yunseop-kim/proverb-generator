<template>
  <div>
    <h2>명언 제조기</h2>
    <div>
      <input type="file" accept="image/*" @change="onFileChange">
      <button @click="save">Save</button>
    </div>
    <div>
      <label>내용 입력</label>
      <br>
      <textarea type="text" v-model="proverb"/>
    </div>
    <div style="border: 1px solid #444444; width: 400px; height: 400px;">
      <v-stage id="stage" ref="stage" @mousedown="handleStageMouseDown" :config="stageSize">
        <v-layer ref="dragLayer">
          <v-rect
            :config="{
              x: 0,
              y: 0,
              width: 400,
              height: 400,
              fill: 'white'
            }"
          />
          <v-image
            v-if="image"
            :config="{
                name: 'image',
                image,
                draggable: true,
                x: posX,
                y: posY
              }"
            @dragstart="handleDragStart"
            @dragend="handleDragEnd"
          />
          <v-transformer ref="transformer"/>
          <!-- <v-rect
            ref="quotation"
            :config="{
              name: 'quotation',
              x: 85,
              y: 350,
              draggable: true,
              width: 6,
              height: 25,
              fill: '#dfe2e5'
            }"
          />-->
          <v-text
            ref="proverb"
            :config="{
              name: 'proverb',
            text: proverb,
            align: 'center',
            fontSize: 15,
            lineHeight: 1,
            fill: '#aeaeae',
            draggable: true,
            x: textPosX,
            y: textPosY
          }"
          />
        </v-layer>
      </v-stage>
    </div>
  </div>
</template>

<script>
const width = 400;
const height = 400;

export default {
  data() {
    return {
      selectedShapeName: "",
      isDragging: false,
      image: null,
      list: [],
      proverb: "",
      posX: 0,
      posY: 0,
      textPosX: 65,
      textPosY: 350,
      stageSize: {
        width: width,
        height: height
      }
    };
  },
  watch: {
    proverb(oldVal) {
      const proverbNode = this.$refs.proverb.getStage();
      // const quotationNode = this.$refs.quotation.getStage();
      const quotationDefaultHeight = 20;
      const proverbDefaultHeight = 15;
      const textWidth = proverbNode.getTextWidth();
      console.log(textWidth);
      this.textPosX = (400 - textWidth) / 2;
      this.textPosY = 350;
    }
  },
  methods: {
    onFileChange(e) {
      const self = this;
      const file = e.target.files[0];
      const image = new window.Image();
      image.src = URL.createObjectURL(file);
      image.onload = function() {
        if (this.width > 400 || this.height > 400) {
          const w = Math.round(this.width / 400);
          console.log(`w:${w}`);
          // const h = Math.ceil(this.height / 400);
          this.width = this.width / (w + 1);
          this.height = this.height / (w + 1);
        }
        self.posX = 200 - this.width * 0.5;
        self.posY = 50;
        self.image = image;
      };
    },

    handleDragStart() {
      this.isDragging = true;
    },
    handleDragEnd() {
      this.isDragging = false;
    },

    downloadURI(uri, name) {
      var link = document.createElement("a");
      link.download = name;
      link.href = uri;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
    save() {
      var dataURL = this.$refs.stage.getStage().toDataURL();
      this.downloadURI(dataURL, "stage.png");
    },

    handleStageMouseDown(e) {
      // clicked on stage - cler selection
      if (e.target === e.target.getStage()) {
        this.selectedShapeName = "";
        this.updateTransformer();
        return;
      }

      // clicked on transformer - do nothing
      const clickedOnTransformer =
        e.target.getParent().className === "Transformer";
      if (clickedOnTransformer) {
        return;
      }

      const name = e.target.name();
      this.selectedShapeName = name;
      this.updateTransformer();
    },
    updateTransformer() {
      // here we need to manually attach or detach Transformer node
      const transformerNode = this.$refs.transformer.getStage();
      const stage = transformerNode.getStage();
      const { selectedShapeName } = this;

      const selectedNode = stage.findOne("." + selectedShapeName);
      // do nothing if selected node is already attached
      if (selectedNode === transformerNode.node()) {
        return;
      }

      if (selectedNode) {
        // attach to another node
        transformerNode.attachTo(selectedNode);
      } else {
        // remove transformer
        transformerNode.detach();
      }
      transformerNode.getLayer().batchDraw();
    }
  }
};
</script>