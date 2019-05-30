<template>
  <div class="font-sans container mt-6 mx-auto px-2">
    <div class="flex justify-end align-middle mb-4">
      <transition name="fade">
        <p class="font-mono text-green-600 self-center mr-4" v-show="done">
          DONE!
        </p>
      </transition>
      <button
        @click="startConvert(files)"
        class="bg-orange-500 font-mono text-sm text-white py-2 px-6 rounded-full float-right self-center"
      >
        Start Conversion
      </button>
    </div>
    <div class="block">
      <div id="file-drag-drop w-full p-2">
        <form
          ref="fileform"
          class="fileform w-full h-40 rounded-lg hover:shadow-xl shadow-md flex bg-blue-500 text-center border-dashed border-4 border-blue-700"
        >
          <span class="font-mono drop-files text-white self-center">
            Drop your files here
          </span>
        </form>
      </div>
    </div>
    <div class="w-full pt-4 px-1">
      <button id="outputFolderSelector" class="very-sweet-looking">Open</button>
      <p>Output Folder: {{ outputFolder }}</p>
      <input
        @change="processFile($event)"
        id="outputFolderInput"
        type="file"
        style="display: none"
        webkitdirectory
      />
      <div v-show="files.length > 0">
        <p class="mb-1 font-mono text-sm">
          Files:
        </p>
        <ul v-for="item in files" :key="item.id" class="w-full">
          <li class="text-xs flex mb-2 border rounded shadow">
            <div class="mr-4 w-1/6">
              <img :src="'file://' + item.path" class=" w-32 p-1" />
            </div>
            <div class="w-5/6 pt-3">
              <button
                class="block rounded-full border border-red-600 text-red-600 w-6 h-6 mr-3 float-right"
              >
                X
              </button>
              {{ item.name }}<br />
              {{ (item.size / 1000000).toFixed(2) }}mb
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
const fs = require("fs");
const path = require("path");
const imagemin = require("imagemin");
const imageminWebp = require("imagemin-webp");

export default {
  name: "DragAndDrop",
  data() {
    return {
      dragAndDropCapable: false,
      files: [],
      doneFiles: [],
      isDraggedOn: false,
      done: false,
      outputFolder: ""
    };
  },
  mounted() {
    document
      .getElementById("outputFolderSelector")
      .addEventListener("click", () => {
        document.getElementById("outputFolderInput").click();
      });

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

      this.$refs.fileform.addEventListener("drop", e => {
        for (let i = 0; i < e.dataTransfer.files.length; i++) {
          this.files.push(e.dataTransfer.files[i]);
          console.log(e.dataTransfer.files[i]);
        }
      });
    }
  },
  methods: {
    processFile(event) {
      console.log(event.target.files);
      this.outputFolder = event.target.files[0].path;
    },
    determineDragAndDropCapable() {
      var div = document.createElement("div");
      return (
        ("draggable" in div || ("ondragstart" in div && "ondrop" in div)) &&
        "FormData" in window &&
        "FileReader" in window
      );
    },
    startConvert(files) {
      console.log(files);
      files.forEach(el => {
        this.convertImages(el, (err, file) => {
          if (err) {
            console.log("ERROR");
            console.log(err);
            return;
          }
          console.log("FILE CONVERSION DONE");
          this.doneFiles.push(file);
          if (this.doneFiles.length === this.files.length) {
            this.done = true;
            setTimeout(() => {
              this.done = false;
              this.files = [];
              this.doneFiles = [];
            }, 1000);
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
  transition: 0.4s;
}

.drop-files {
  width: 100%;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
</style>
