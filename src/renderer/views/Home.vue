<template>
  <div class="row">
    <el-input size="small" v-model="path" />
    <el-button size="small" ref="select" type="primary" @click="selectDirectory">浏览</el-button>
    <el-button size="small" v-loading.fullscreen.lock="fullscreenLoading" :element-loading-text="loadingText" ref="start" type="primary" @click="decryptAllFiles">开始</el-button>
  </div>
</template>

<script>
const fs = require("fs");
const { remote } = require("electron");
const dialog = remote.dialog
export default {
  name: 'home',
  data() {
    return {
      fullscreenLoading: false,
      loadingText: '',
      path: ''
    }
  },
  mounted() {
  },
  methods: {
    selectDirectory() {
      dialog.showOpenDialog({
        properties: ["openDirectory"],
      },(res)=>{
        if (res.canceled) {
          this.path = "";
        } else {
          this.path = res[0];
        }
      })
    },
    decryptAllFiles() {
      if (!this.path) return;
      this.loadingText = "加载文件...";
      this.fullscreenLoading = true;
      setTimeout(() => {
        this.handleFilesList(this.path).then((list) => {
          this.$nextTick(() => {
            this.handleFiles(list);
          })
        });
      }, 50);
    },
    handleFiles(list) {
      for (let i = 0; i < list.length; i++) {
        let data = fs.readFileSync(list[i]);
        fs.writeFileSync(list[i], data, {});
      }
      console.log('complete')
      this.fullscreenLoading = false;
    },
    handleFilesList(path) {
      return new Promise((resolve, reject) => {
        var list = [];
        async function getAllFiles(filePath) {
          if (!filePath) return;
          let exists = fs.existsSync(filePath);
          if (exists) {
            let files = fs.readdirSync(filePath);
            for (let i = 0; i < files.length; i++) {
              let file = files[i];
              let currentFilePath = filePath + "\\" + file;
              let stats = fs.lstatSync(currentFilePath);
              if (stats.isDirectory()) {
                await getAllFiles(currentFilePath);
              } else {
                list.push(currentFilePath);
              }
            }
            if (filePath == path) {
              resolve(list);
            }
          } else {
            console.warn(`指定的目录${filePath}不存在！`);
          }
        }
        getAllFiles(path);
      });
    }
  }
}
</script>

<style scoped>
.row {
  display: flex;
  flex-direction: row;
}
.row button {
  display: inline-block;
  margin: 0!important;
  margin-left: 10px!important;
}
</style>
