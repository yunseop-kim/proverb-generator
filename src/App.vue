<template>
  <div>
    <h2>명언 제조기</h2>
    <h3>이 글을 보고 영감을 얻었습니다. <a href="http://mlbpark.donga.com/mp/b.php?p=1&b=bullpen&id=201707160006344965&select=&query=&user=&site=&reply=&source=&sig=h6jBSY21j3DRKfX@hca9Sl-g4hlq">(링크)</a></h3>
    <div>
      <input type="file" accept="image/*" @change="onFileChange">
      <button @click="save">Save</button>
    </div>
    <div>
      <label>내용 입력</label>
      <br>
      <textarea type="text" v-model="configs.proverb.text"/>
    </div>
    <div style="border: 1px solid #444444; width: 400px; height: 400px;">
      <v-stage name="stage" ref="stage" @mousedown="handleStageMouseDown" :config="stageSize">
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
            v-if="configs.image.image"
            :config="configs.image"
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
          <v-text ref="proverb" :config="configs.proverb"/>
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
      list: [],
      stageSize: {
        width: width,
        height: height
      },
      configs: {
        image: {
          name: "image",
          image: null,
          draggable: true,
          x: 0,
          y: 0
        },
        proverb: {
          name: "proverb",
          text: "",
          align: "center",
          fontSize: 15,
          lineHeight: 1,
          fill: "#aeaeae",
          draggable: true,
          x: 65,
          y: 350
        }
      }
    };
  },
  watch: {
    'configs.proverb.text'() {
      const proverbNode = this.$refs.proverb.getStage();
      const textWidth = proverbNode.getTextWidth();
      this.configs.proverb.x = (400 - textWidth) / 2;
      this.configs.proverb.y = 350;
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
          // const h = Math.ceil(this.height / 400);
          this.width = this.width / (w + 1);
          this.height = this.height / (w + 1);
        }
        self.configs.image.x = 200 - this.width * 0.5;
        self.configs.image.y = 50;
        self.configs.image.image = image;
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