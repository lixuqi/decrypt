<template>
  <div class="col">
    <div class="row">
      <el-input size="small" v-model="path">
        <template #suffix>
          <span :class="[{'icon-success':fileList.length>0,'icon-complete':complete}]" v-if="!!path" class="el-icon-circle-check icon-default"></span>
        </template>
      </el-input>
      <el-button size="small" ref="select" type="info" @click="selectDirectory">浏览</el-button>
    </div>
    <div class="row row-btn">
      <el-button :disabled="!path" size="small" v-loading.fullscreen.lock="fullscreenLoading" element-loading-text="加载文件..." element-loading-spinner="el-icon-loading" element-loading-background="rgba(0, 0, 0, 0.8)" type="primary" @click="loadFile">读取</el-button>
      <el-button :disabled="fileList.length==0" size="small" type="primary" @click="handleFiles">开始</el-button>
    </div>
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
      path: '',
      fileList: [],
      complete: false
    }
  },
  mounted() {
  },
  methods: {
    selectDirectory() {
      dialog.showOpenDialog({
        properties: ["openDirectory"],
      }, (res) => {
        if (res) {
          this.path = res[0];
        } else {
          this.path = "";
        }
        this.complete = false;
        this.fileList = [];
      })
    },
    loadFile() {
      if (!this.path) return;
      this.fullscreenLoading = true;
      setTimeout(() => {
        this.handleFilesList(this.path).then((list) => {
          this.fullscreenLoading = false;
          this.fileList = list;
          this.complete = false;
        });
      }, 50);
    },
    handleFiles() {
      const loading = this.$loading({
        lock: true,
        text: 'Loading...',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.8)'
      });
      let list = this.fileList;
      let errFile = [];
      setTimeout(() => {
        for (let i = 0; i < list.length; i++) {
          try {
            let data = fs.readFileSync(list[i]);
            fs.unlinkSync(list[i]);
            fs.writeFileSync(list[i], data, {});
          } catch (e) {
            errFile.push(list[i])
          }
        }
        dialog.showMessageBox({
          type: 'none',
          buttons: ['确定'],
          title: '已完成',
          message: errFile.length > 0 ? `全部完成` : `失败数量：${errFile.length} / ${list.length}`
        }, () => {
          if (errFile.length > 0) fs.writeFileSync('errFile.log', errFile.join(','), {});
          this.complete = true;
          loading.close()
        })
      }, 50);
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
.col {
  margin: 10px;
  margin-right: 0;
}
.row {
  display: flex;
  margin-bottom: 10px;
  flex-direction: row;
}
.row > * {
  display: inline-block;
  margin: 0 !important;
  margin-right: 10px !important;
}
.row-btn button {
  width: 50%;
}
.icon-default {
  line-height: 32px;
  font-size: 16px;
}
.icon-success {
  color: #409eff;
}
.icon-complete {
  color: #67c23a;
}
</style>
