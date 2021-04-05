<template>
  <v-app>
    <div class="d-flex ml-3 mt-3">
      <v-icon>mdi-magnify</v-icon>
      <v-text-field
        class="mx-3"
        placeholder="搜索指定应用包名"
        v-model="searchInfo"
      ></v-text-field>
      <v-dialog v-model="showInstall" width="500">
        <template v-slot:activator="{ on, attrs }">
          <v-btn
            class="mt-3 mr-3"
            @click="
              showInstall = true;
              installingApp = false;
            "
            color="primary"
            v-bind="attrs"
            v-on="on"
            depressed
          >
            <v-icon class="mr-2">mdi-plus-box-multiple</v-icon>
            安装应用
          </v-btn>
        </template>

        <v-card>
          <v-card-title>安装应用</v-card-title>

          <v-card-text>
            <template v-if="!installingApp">
              <v-file-input
                v-model="installAppChosen"
                accept=".apk"
                label="选择 APK 文件"
                multiple
                show-size
                counter
              />
            </template>
            <template v-else>
              <div>
                {{ installAppProcess["Installed"] }} /
                {{ installAppProcess["All"] }}
                <v-progress-linear
                  :value="
                    (this.installAppProcess['Installed'] /
                      this.installAppProcess['All']) *
                    100
                  "
                  color="blue lighten-1"
                ></v-progress-linear>
                <v-alert
                  color="white"
                  style="font-family: Consolas"
                  class="mt-3"
                  >{{ installAppOutput }}</v-alert
                >
              </div>
            </template>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" text @click="showInstall = false">
              关闭
            </v-btn>
            <template v-if="!installingApp">
              <v-btn
                color="primary"
                text
                @click="InstallAPK"
                :disabled="installAppChosen.length < 1"
              >
                安装
              </v-btn>
            </template>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>
    <div class="d-flex align-start">
      <v-list dense class="mx-3" width="50%">
        <v-subheader>Appliction</v-subheader>
        <v-list-item-group v-model="appSelected" color="primary" no-action>
          <v-list-item
            v-for="(item, i) in appList"
            :key="i"
            :disabled="!item.match(searchInfo)"
            @click="GetAPPInfo(item)"
          >
            <v-list-item-icon>
              <v-icon>mdi-application</v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title>{{ item }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list-item-group>
      </v-list>
      <template v-if="appSelected != null">
        <v-card
          outlined
          style="border: none"
          width="50%"
          height="100%"
          fluid
          class="mt-4"
        >
          <v-card-title class="mt-5 ml-5">{{
            appList[appSelected]
          }}</v-card-title>
          <div class="mx-7">
            <v-dialog v-model="showSaveApk" width="500">
              <template v-slot:activator="{ on, attrs }">
                <v-btn
                  block
                  depressed
                  color="primary"
                  @click="SaveAPK(appList[appSelected])"
                  v-bind="attrs"
                  v-on="on"
                >
                  <v-icon class="mr-2">mdi-widgets-outline</v-icon>
                  提取
                </v-btn>
              </template>
              <v-card>
                <v-card-title>提取应用 APK 文件</v-card-title>
                <v-card-text>
                  <b>
                    文件将保存到
                    <code>{{
                      require("child_process").execSync("echo %cd%")
                    }}</code>
                  </b>
                  <p style="font-family: Consolas">{{ saveApkInfo }}</p>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn
                    class="mt-3 mx-5"
                    @click="showSaveApk = false"
                    color="primary"
                    text
                    >关闭</v-btn
                  ></v-card-actions
                >
              </v-card>
            </v-dialog>
            <v-dialog v-model="showUninstall" width="500">
              <template v-slot:activator="{ on, attrs }">
                <v-btn
                  block
                  depressed
                  color="error"
                  class="my-3"
                  @click="
                    showUninstall = true;
                    agreeUninstall = false;
                    uninstalling = false;
                  "
                  v-bind="attrs"
                  v-on="on"
                >
                  <v-icon class="mr-2">mdi-delete-outline</v-icon>
                  卸载<b>并清除数据</b>
                </v-btn>
              </template>
              <v-card>
                <v-card-title>卸载应用</v-card-title>
            <template v-if="!uninstalling">
              <v-card-text>
                <b> 您将在卸载后失去您的应用和应用信息，依然继续？ </b>
                <v-checkbox
                  v-model="agreeUninstall"
                  label="我知晓并依然准备继续"
                  color="error"
                ></v-checkbox>
              </v-card-text>
            </template>
            <template v-if="uninstalling">
              <v-card-text>
                <p style="font-family: Consolas">{{ uninstallAppInfo }}</p>
              </v-card-text>
            </template>
            <v-card-actions>
                <v-spacer></v-spacer>
                <template v-if="!uninstalling"><v-btn
                  color="error"
                  text
                  :disabled="!agreeUninstall"
                  @click="UninstallAPK(appList[appSelected])"
                >
                  卸载
                </v-btn></template>
                <v-btn
                    @click="showUninstall = false"
                    color="primary"
                    text
                    >关闭</v-btn
                  >
              </v-card-actions>
              </v-card>
            </v-dialog>
          </div>
          <v-card-text>
            <div>
              <v-icon class="mx-3">mdi-source-branch</v-icon>
              <span class="mr-2">版本</span> {{ appInfoFormatted["version"] }}
            </div>
            <div>
              <v-icon class="mx-3">mdi-dev-to</v-icon>
              <span class="mr-2">SDK 版本</span>
              {{ appInfoFormatted["targetSdk"] }}
            </div>
            <div>
              <v-icon class="mx-3">mdi-lock</v-icon>
              <span class="mr-2">已请求权限</span>
              <v-list dense max-width="5em">
                <v-list-item
                  v-for="(item, i) in appInfoFormatted['permissions']"
                  :key="i"
                >
                  <span class="ml-3 text-caption">{{ item }}</span>
                </v-list-item>
              </v-list>
            </div>
          </v-card-text>
        </v-card>
      </template>
    </div>
  </v-app>
</template>

<script>
const execSync = require("child_process").execSync;
const exec = require("child_process").exec;

export default {
  name: "ApkManager",

  props: ["deviceName"],

  data: function () {
    return {
      adb: localStorage.getItem("ADBPath"),
      fastboot: localStorage.getItem("FastbootPath"),
      appList: [],
      appSelected: null,
      searchInfo: "",
      //=== INSTALL APP ===
      showInstall: false,
      installingApp: false,
      installAppProcess: { Installed: 0, All: 1 },
      installAppOutput: "",
      installAppChosen: [],
      //=== APP INFO ===
      appInfoFormatted: {},
      showSaveApk: false,
      saveApkInfo: "",
      showUninstall: false,
      uninstallAppInfo: "",
      agreeUninstall: false,
      uninstalling: false,
    };
  },

  created: function () {
    this.GetAPKList();
  },

  methods: {
    GetAPKList: function () {
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
    GetAPPInfo: function (p) {
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
    InstallAPK: function () {
      this.installingApp = true;
      this.installAppProcess.Installed = 0;
      this.installAppProcess.All = this.installAppChosen.length;
      this.installAppOutput = "";
      this.installAppChosen.forEach((item) => {
        exec(
          `${this.adb} -s ${this.deviceName} install ${item.path}`,
          (stdout, stderr) => {
            if (stdout == null) stdout = stderr;
            this.installAppOutput += "\n" + stdout;
            this.installAppProcess.Installed++;
          }
        );
      });
      this.GetAPKList();
    },
    SaveAPK: function (p) {
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
    UninstallAPK: function (p) {
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
    DisableAPP: function (p) {
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
