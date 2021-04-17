<template>
  <div class="main">
    <div class="ml-8 mt-3 mb-5 mr-8">
      <span
        style="font-size:1.4em"
        v-for="(item, index) in dirName.split('/')"
        :key="index"
      >
        <span>{{ item }}</span>
        <template v-if="index + 1 != dirName.split('/').length">
          <span class="ma-2 color-gray">/</span>
        </template>
      </span>
      <button class="pull-right color-white mr-8 primary ">
        <input
          class="ghost"
          type="file"
          multiple="multiple"
          @change="PushFiles($event)"
        />
        <span class="material-icons center-y mr-1">file_upload</span>传入文件
      </button>
    </div>
    <ul class="mr-3">
      <li @click="ChangeDir('../')">
        <span class="ml-1 font-mono">../</span>
      </li>
      <li
        v-for="(item, index) in dirFiles"
        :key="index"
        @click="
          if (item.type == 'file') {
            OpenFile(item.name);
          }
          if (item.type == 'dir' || item.type == 'link') {
            ChangeDir(item.name);
          }
        "
        @contextmenu.prevent="
          contextmenu.show = true;
          contextmenu.item = item;
        "
        :title="item.link"
      >
        <span class="material-icons mr-3 center-y">
          <template v-if="item.type == 'dir'">folder</template>
          <template v-if="item.type == 'file'">insert_drive_file</template>
          <template v-if="item.type == 'link'">insert_link</template>
        </span>
        <span>{{ item.name }}</span>
      </li>
      <!-- Context Menu -->
      <transition name="ot15">
        <div class="dialog opa-trans-15" v-show="contextmenu.show">
          <div class="dialog-overlay" @click="contextmenu.show = false" />
          <div class="dialog-content card" style="width:50%;">
            <p class="card-title">{{ contextmenu.item.name }}</p>
            <!-- Size -->

            <p v-show="contextmenu.item.type == 'file'">
              <span class="material-icons center-y">save</span>
              <b class="mx-2">占用空间</b>
              <span
                >{{ contextmenu.item.size }} 字节（
                {{ (contextmenu.item.size / 1000 / 1024).toFixed(2) }} MB
                ）</span
              >
            </p>

            <!-- Date -->

            <p>
              <span class="material-icons center-y">access_time</span>
              <b class="mx-2">最后编辑</b>
              <span>{{ contextmenu.item.date }}</span>
            </p>

            <!-- Permissions -->

            <p>
              <span class="material-icons center-y">verified_user</span>
              <b class="mx-2">权限</b>
              <span>{{ contextmenu.item.permissions }}</span>
            </p>

            <!-- Active -->
            <li
              class="pl-4"
              @click="
                copyFile = {};
                copyFile.path = `${dirName}/${contextmenu.item.name}`;
                copyFile.name = `${contextmenu.item.name}`;
              "
            >
              <span class="material-icons mr-2 center-y">content_copy</span>复制
            </li>
            <li
              class="pr-4"
              @click="
                showRemoveFile = true;
                removeFileName = contextmenu.item;
              "
              v-show="contextmenu.item.type != 'link'"
            >
              <span class="material-icons mr-2 center-y">delete</span>删除
            </li>
          </div>
          <!-- RemoveFile Alert -->
          <transition name="ot15">
            <div class="dialog opa-trans-15" v-show="showRemoveFile">
              <div class="dialog-overlay" />
              <div class="dialog-content card" style="width:60%">
                <p class="card-title">您确定吗</p>
                <p class="card-content">
                  永久的删除 {{ removeFileName.name }} ？
                </p>
                <div class="card-active">
                  <button class="color-primary" @click="showRemoveFile = false">
                    取消
                  </button>
                  <button
                    class="color-error"
                    @click="
                      RemoveFile(
                        `${dirName}/${removeFileName.name}`,
                        `${removeFileName.type}`
                      )
                    "
                  >
                    删除
                  </button>
                </div>
              </div>
            </div>
          </transition>
        </div>
      </transition>
    </ul>
    <!-- Upload File Dialog -->
    <transition name="ot15">
      <div class="dialog opa-trans-15" v-show="showPushFile">
        <div class="dialog-overlay" />
        <div class="dialog-content card">
          <div class="card-title">推送文件</div>
          <p class="card-content font-mono">
            {{ pushFilesOutput }}
          </p>
          <div class="card-active">
            <button class="color-primary" @click="showPushFile = false">
              关闭
            </button>
          </div>
        </div>
      </div>
    </transition>
    <!-- Copy -->
    <transition name="ot15">
      <div class="bottom-active opa-trans-15" v-show="copyFile != false">
        已选择 {{ copyFile.name }}
        <button
          class="pull-right button-icon"
          @click="
            PasteFile(copyFile.path, `${dirName}/${copyFile.name}`);
            copyFile = false;
          "
        >
          <span class="material-icons center-y">content_paste</span>
        </button>
        <button class="pull-right button-icon" @click="copyFile = false">
          <span class="material-icons center-y">clear</span>
        </button>
      </div>
    </transition>
  </div>
</template>

<script>
const exec = require("child_process").exec;
const execSync = require("child_process").execSync;
const spawn = require("child_process").spawn;
const fs = require("fs");

export default {
  name: "FileManager",

  props: ["deviceName"],

  data: function() {
    return {
      adb: localStorage.getItem("ADBPath"),
      fastboot: localStorage.getItem("FastbootPath"),
      dirName: "/storage",
      dirFiles: [],
      pushFilesChosen: [],
      pushFilesOutput: "",
      showPushFile: false,
      contextmenu: { show: false, item: { name: null } },
      copyFile: false,
      showRemoveFile: false,
      removeFileName: { name: null, path: null },
    };
  },

  created: function() {
    this.ChangeDir("./sdcard0");
  },

  methods: {
    GetFileList: function(dirName) {
      exec(
        `${this.adb} -s ${this.deviceName} shell "cd ${dirName} && ls -l"`,
        (stdout, stderr, err) => {
          if (stdout == null) stdout = stderr;
          if (err) {
            alert(`Σ(っ °Д °;)っ⚠\n${err}`);
            return;
          }

          this.dirFiles = [];
          this.dirName = dirName;

          stdout.split("\n").forEach((item) => {
            // Filter and Init
            if (item.split(" ")[0] == "total" || item == "") return;

            let splited = item.split(" ").filter((p) => {
              return p != "";
            });
            let tmp = {};

            /* Output Like:
                drwxrwx--x  57 system system     4096 2021-04-17 08:51 data
             >  d?????????   ? ?      ?             ?                ? data_mirror
            */
            if (item.indexOf("?????????") != -1) {
              tmp.name = splited.pop();
              if (item[0] == "-") tmp.type = "file";
              else if (item[0] == "d") tmp.type = "dir";
              else if (item[0] == "l") tmp.type = "link";
              else tmp.type = "unknown";
              tmp.date = "I dont know";
              tmp.size = "I dont know too";
              tmp.permissions = "Why do you ask that?";
              this.dirFiles.push(tmp);
              return;
            }

            // Type
            if (item[0] == "-") tmp.type = "file";
            else if (item[0] == "d") tmp.type = "dir";
            else if (item[0] == "l") tmp.type = "link";
            else tmp.type = "unknown";

            // Name
            tmp.name = item.substring(item.lastIndexOf(":") + 4);
            /*                               ↑ ↑ ↑                 FileName
                                                               ↓   |———————|
             >  drwxrwx---  2 root everybody 3488 2021-04-17 12:01 File Name 
                                                               |___|
                                                               4 str
            */
            if (tmp.type == "link") {
              tmp.name = splited[splited.length - 3];
              tmp.link = splited[splited.length - 1];
            }

            // Date

            tmp.date = `${splited[5]} ${splited[6]}`;

            // Size

            tmp.size = `${splited[4]}`;

            // Permissions

            tmp.permissions = `${splited[0]}`;

            // Push
            this.dirFiles.push(tmp);
          });
        }
      );
    },
    ChangeDir: function(p) {
      exec(
        `${this.adb} -s ${this.deviceName} shell "cd ${this.dirName} && cd ${p} && pwd"`,
        (stdout, stderr, err) => {
          if (stdout == null) stdout = stderr;
          if (err) {
            alert(`Σ(っ °Д °;)っ⚠\n${err}`);
            return;
          }
          stdout = stdout.replace("\n", "").replace("\r", "");
          this.GetFileList(stdout);
        }
      );
    },
    PushFiles: function(event) {
      let files = event.target.files;
      if (files.length == 0) return;
      this.pushFilesOutput = "";
      this.showPushFile = true;
      console.log(files);
      files.forEach((item) => {
        let push = spawn(
          this.adb,
          [
            `-s ${this.deviceName}`,
            `push "${item["path"]}" "${this.dirName}/${item["name"]}"`,
          ],
          {
            shell: process.platform === "win32",
          }
        );
        push.stdout.on("data", (data) => {
          this.pushFilesOutput += `${data.toString()}\n\n`;
        });
        push.stderr.on("data", (err) => {
          alert(`Σ(っ °Д °;)っ⚠\n${err.toString()}`);
          return;
        });
      });
      this.ChangeDir(".");
    },
    PasteFile: function(from, to) {
      exec(
        `${this.adb} -s ${this.deviceName} shell "cp '${from}' '${to}'"`,
        (stdout, stderr, err) => {
          if (err) {
            alert(`Σ(っ °Д °;)っ⚠\n${err}`);
            return;
          }
          if (stdout == null) stdout = stderr;
          this.ChangeDir(".");
        }
      );
    },
    RemoveFile: function(path, type) {
      let rf;
      if (type == "dir") rf = "-rf";
      else rf = "";
      exec(
        `${this.adb} -s ${this.deviceName} shell "rm ${rf} '${path}'"`,
        (stdout, stderr, err) => {
          if (err) {
            alert(`Σ(っ °Д °;)っ⚠\n${err}`);
            return;
          }
          if (stdout == null) stdout = stderr;
          this.showRemoveFile = false;
          this.contextmenu.show = false;
          this.ChangeDir(".");
        }
      );
    },
    OpenFile: function(p) {
      let pullandopen = (f) => {
        console.log(f);
        let pull = spawn(
          this.adb,
          [`-s ${this.deviceName}`, `pull "${this.dirName}/${f}" "tmp/${f}"`],
          {
            shell: process.platform === "win32",
          }
        );
        pull.stdout.on("data", () => {
          execSync(`start "" "tmp\\${f}"`);
        });
        pull.stderr.on("data", (err) => {
          alert(`Σ(っ °Д °;)っ⚠\n${err.toString()}`);
          return;
        });
      };
      fs.access("tmp/", function(err) {
        if (err) {
          fs.mkdir("tmp", (err) => {
            if (err) {
              alert(`Σ(っ °Д °;)っ⚠\n${err.toString()}`);
              return;
            }
            pullandopen(p);
          });
        } else {
          pullandopen(p);
        }
      });
    },
  },
};
</script>

<style>
.ot15-enter-active .ot15-leave-active {
  transition: opacity 0.15s;
}

.ot15-enter,
.ot15-leave-to {
  opacity: 0;
}
</style>
