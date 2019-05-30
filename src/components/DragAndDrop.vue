<template>
  <div class="container mx-auto px-2">
    <div class="flex">
      <div id="file-drag-drop w-1/2 p-2">
        <form
          ref="fileform"
          class="fileform bg-cyan"
          :class="{ dragOn: isDraggedOn }"
        >
          <span class="font-body drop-files text-white"
            >Drop the files here!</span
          >
        </form>
      </div>
      <div class="w-1/2 p-4">
        <p>
          Files:
        </p>
        <div v-show="files.length > 0">
          <ul v-for="item in files" :key="item.id">
            <li class="text-xs">
              <img :src="item.path" class="w-12" />
              {{ (item.size / 1000000).toFixed(2) }}mb
              {{ item.name }}
            </li>
          </ul>
        </div>
      </div>
    </div>
    <button
      @click="startConvert"
      class="bg-orange-500 text-white p-2 rounded mt-4 float-right"
    >
      Start Conversion
    </button>
  </div>
</template>

<script>
const fs = require("fs");
const path = require("path");
const imagemin = require("imagemin");
const imageminWebp = require("imagemin-webp");
const fixPath = require("fix-path");

export default {
  name: "DragAndDrop",
  data() {
    return {
      dragAndDropCapable: false,
      files: [],
      doneFiles: [],
      isDraggedOn: false,
      done: false
    };
  },
  mounted() {
    this.dragAndDropCapable = this.determineDragAndDropCapable();
    if (this.dragAndDropCapable) {
      [
        "drag",
        "dragstart",
        "dragend",
        "dragover",
        "dragenter",
        "dragleave",
        "drop"
      ].forEach(
        function(evt) {
          this.$refs.fileform.addEventListener(
            evt,
            function(e) {
              e.preventDefault();
              e.stopPropagation();
            }.bind(this),
            false
          );
        }.bind(this)
      );

      this.$refs.fileform.addEventListener(
        "drop",
        function(e) {
          for (let i = 0; i < e.dataTransfer.files.length; i++) {
            this.files.push(e.dataTransfer.files[i]);
            console.log(e.dataTransfer.files[i]);
          }
        }.bind(this)
      );

      this.$refs.fileform.addEventListener("dragenter", () => {
        this.isDraggedOn = true;
      });

      this.$refs.fileform.addEventListener("dragleave", () => {
        this.isDraggedOn = false;
      });
      this.$refs.fileform.addEventListener("drop", () => {
        this.isDraggedOn = false;
      });
    }
  },
  methods: {
    determineDragAndDropCapable() {
      var div = document.createElement("div");
      return (
        ("draggable" in div || ("ondragstart" in div && "ondrop" in div)) &&
        "FormData" in window &&
        "FileReader" in window
      );
    },
    startConvert() {
      this.files.forEach(el => {
        this.convertImages(el, (err, file) => {
          if (err) {
            console.log("ERROR");
            console.log(err);
            return;
          }

          this.doneFiles.push(file);
          if (this.doneFiles.length === this.files.length) {
            this.done = true;
          }
          return;
        });
      });
    },
    convertImages(file, cb) {
      fs.readFile(file.path, (err, buf) => {
        if (err) {
          console.log("READ ERROR");
          cb(err);
          return;
        }
        imagemin([file.path], path.join(path.dirname(file.path), "build"), {
          use: [imageminWebp({ quality: 50 })]
        })
          .then(file => {
            cb(null, Object.assign(file, { original: buf }));
          })
          .catch(error => {
            console.log("CONVERT ERROR");
            cb(error);
          });
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.fileform {
  border: 6px solid rgba(0, 0, 0, 0);
  transition: 0.4s;
}

.dragOn {
  border: 6px solid rgba(0, 100, 0, 0.6);
}

form {
  display: block;
  height: 300px;
  width: 300px;
  text-align: center;
  line-height: 275px;
  border-radius: 4px;
}
</style>
