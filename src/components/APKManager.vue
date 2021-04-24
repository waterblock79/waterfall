<template>
  <div class="main">
    <div class="flex ml-5 mt-3">
      <span class="material-icons center-y">search</span>
      <input
        type="text"
        class="center-y ml-3 mr-3 flex-grow"
        placeholder="搜索应用包名..."
      />
      <button class="mr-3 center-y primary color-white">
        <input
          type="file"
          class="ghost"
          accept=".apk"
          @change="
            InstallAPK($event);
          "
        />
        <span class="material-icons mr-2 center-y">library_add</span>
        安装应用
      </button>
    </div>
    <transition name="dialog">
      <div class="dialog" v-show="showInstall">
        <div class="dialog-overlay" />
        <div class="dialog-content card">
          <span class="card-title">安装应用</span>
          <div class="card-content flex">
            <span class="color-gray" style="width:100%"
              >{{ installAppProcess.Installed }} /
              {{ installAppProcess.All }}</span
            >
            <progress
              style="width:100%;"
              :value="installAppProcess.Installed"
              :max="installAppProcess.All"
            />
            <pre class="font-mono">{{ installAppOutput }}</pre>
          </div>
          <div class="card-active">
            <button class="color-primary" @click="showInstall = false">
              关闭
            </button>
          </div>
        </div>
      </div>
    </transition>
    <div class="d-flex align-start">
      <ul style="width:50%">
      <sub>Appliction</sub>
        <li
          v-for="(item, i) in appList"
          :key="i"
          @click="
            appSelected = i;
            GetAPPInfo(item);
          "
          class="dense mt-1"
          :selected="appSelected == i"
          :disabled="!item.match(searchInfo)"
        >
          <span class="material-icons mr-4 color-gray">apps</span>
          <span class="color-black font-smaller">{{ item }}</span>
        </li>
      </ul>
      <template v-if="appSelected != null">
        <div class="mt-4 card" style="box-shadow:none">
          <p class="mt-5 ml-5 card-title">{{ appList[appSelected] }}</p>
          <div class="flex">
            <button
              class="primary color-white flex-grow mx-7"
              style="width:100%"
              @click="SaveAPK(appList[appSelected])"
            >
              <span class="material-icons mr-2">widgets</span>
              提取
            </button>
            <button
              class="flex-grow error color-white mx-7 my-3"
              @click="
                showUninstall = true;
                agreeUninstall = false;
                uninstalling = false;
              "
            >
              <span class="material-icons mr-2">delete</span>
              卸载<b>并清除数据</b>
            </button>
          </div>
          <div class="mx-7">
            <transition name="dialog">
              <div class="dialog" v-show="showSaveApk">
                <div class="dialog-overlay" />
                <div class="dialog-content card">
                  <span class="card-title">提取 APK 文件</span>
                  <div class="card-content">
                    <b>
                      文件将保存到
                      <code>{{
                        require("child_process").execSync("echo %cd%")
                      }}</code>
                    </b>
                    <p style="font-family: Consolas">{{ saveApkInfo }}</p>
                  </div>
                  <div class="card-active">
                    <button class="color-primary" @click="showSaveApk = false">
                      关闭
                    </button>
                  </div>
                </div>
              </div>
            </transition>
            <transition name="dialog">
              <div class="dialog" v-show="showUninstall">
                <div class="dialog-overlay" />
                <div class="dialog-content card">
                  <span class="card-title">卸载应用</span>
                  <template v-if="!uninstalling">
                    <div class="card-content">
                      <b> 您将在卸载后失去您的应用和应用信息，依然继续？ </b>
                      <br />
                      <input
                        type="checkbox"
                        v-model="agreeUninstall"
                        class="center-y"
                      />
                      我已知晓并仍然准备继续
                    </div>
                  </template>
                  <template v-if="uninstalling">
                    <div class="card-content">
                      <p style="font-family: Consolas">
                        {{ uninstallAppInfo }}
                      </p>
                    </div>
                  </template>
                  <div class="card-active">
                    <template v-if="!uninstalling"
                      ><button
                        class="color-error"
                        type="button"
                        :disabled="!agreeUninstall"
                        @click="UninstallAPK(appList[appSelected])"
                      >
                        卸载
                      </button></template
                    >
                    <button
                      class="color-primary"
                      @click="showUninstall = false"
                      color="primary"
                      text
                    >
                      关闭
                    </button>
                  </div>
                </div>
              </div>
            </transition>
          </div>
          <div>
            <div class="mx-3">
              <span class="mx-3 material-icons color-gray">update</span>
              <span>版本</span> {{ appInfoFormatted["version"] }}
            </div>
            <div class="my-2 mx-3">
              <span class="mx-3 material-icons color-gray">developer_mode</span>
              <span>SDK 版本</span>
              {{ appInfoFormatted["targetSdk"] }}
            </div>
            <div class="mx-3">
              <span class="mx-3 material-icons color-gray">verified_user</span>
              <span class="mr-2">已请求权限</span>
              <div class="ml-8 mt-3 font-mono" v-for="(item, i) in appInfoFormatted['permissions']"
                  :key="i">
                  {{ item }}
                  <br/>
              </div>
            </div>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script>
const execSync = require("child_process").execSync;
const exec = require("child_process").exec;

export default {
  name: "ApkManager",

  props: ["deviceName"],

  data: function() {
    return {
      adb: localStorage.getItem("ADBPath"),
      fastboot: localStorage.getItem("FastbootPath"),
      appList: [],
      appSelected: null,
      searchInfo: "",
      //=== INSTALL APP ===
      showInstall: false,
      installAppProcess: { Installed: 0, All: 1 },
      installAppOutput: "",
      installAppChosen: [],
      //=== APP INFO ===
      appInfoFormatted: {},
      showSaveApk: false,
      saveApkInfo: "",
      //=== UNINSTALL APP ===
      showUninstall: false,
      uninstallAppInfo: "",
      agreeUninstall: false,
      uninstalling: false,
    };
  },

  created: function() {
    this.GetAPKList();
  },

  methods: {
    GetAPKList: function() {
      this.appList = [];
      exec(
        `${this.adb} -s ${this.deviceName} shell pm list package`,
        (stdout, stderr) => {
          if (stdout == null) stdout = stderr;
          stdout
            .split("\n")
            .forEach((item) => this.appList.push(item.substring(8)));
        }
      );
    },
    GetAPPInfo: function(p) {
      this.appInfoFormatted = {};
      exec(
        `${this.adb} -s ${this.deviceName} shell dumpsys package ${p}`,
        (stdout, stderr) => {
          if (stdout == null) stdout = stderr;
          let appInfoFormatted = {};
          stdout.split("\n").forEach((item) => {
            if (item.match("versionName")) {
              //                                                         Get Version (like 1.1.0)
              appInfoFormatted["version"] = item.split("=")[1];
            }
            if (item.match("versionCode")) {
              item.split(" ").forEach((item) => {
                if (item.match("targetSdk")) {
                  //                                                     Get Target SDK (like 30)
                  appInfoFormatted["targetSdk"] = item.split("=")[1];
                }
              });
            }
          });
          //                                                             Get App Permissions what it requested.
          let permissions = stdout.substring(
            stdout.indexOf("requested permissions"),
            stdout.indexOf("install permissions")
          );
          appInfoFormatted["permissions"] = permissions.split("\n").slice(1);
          this.appInfoFormatted = appInfoFormatted;
        }
      );
    },
    InstallAPK: function(e) {
      let file = e.target.files;
      this.showInstall = true;
      this.installAppProcess.Installed = 0;
      this.installAppProcess.All = file.length;
      this.installAppOutput = "";
      file.forEach((item) => {
        exec(
          `${this.adb} -s ${this.deviceName} install ${item.path}`,
          (stdout, stderr) => {
            if (stdout == null) stdout = stderr;
            this.installAppOutput += "\n" + stdout;
            this.installAppProcess.Installed++;
          }
        );
      });
      while(this.installAppProcess.Installed==this.installAppProcess.All){
        this.GetAPKList();
        return;
      }
    },
    SaveAPK: function(p) {
      this.showSaveApk = true;
      this.saveApkInfo = "";
      let apkPath = execSync(
        `${this.adb} -s ${this.deviceName} shell pm path ${p}`
      )
        .toString()
        .split(":")[1];
      exec(
        `${this.adb} -s ${this.deviceName} pull ${apkPath} %cd%`,
        (stdout, stderr) => {
          if (stdout == null) stdout = stderr;
          this.saveApkInfo += stdout;
        }
      );
    },
    UninstallAPK: function(p) {
      this.uninstallAppInfo = "";
      this.uninstalling = true;
      exec(
        `${this.adb} -s ${this.deviceName} uninstall ${p}`,
        (stdout, stderr) => {
          if (stdout == null) stdout = stderr;
          this.uninstallAppInfo += stdout;
          this.GetAPKList();
        }
      );
    },
    DisableAPP: function(p) {
      exec(
        `${this.adb} -s ${this.deviceName} shell pm disable-user ${p}`,
        (stdout, stderr) => {
          if (stdout == null) stdout = stderr;
          this.showDOEApp = true;
          this.DOEAppInfo = stdout;
        }
      );
    },
  },
};
</script>
