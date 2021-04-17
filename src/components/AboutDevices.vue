<template>
  <v-app>
    <v-card class="d-flex pa-5" outlined tile>
      <v-avatar size="81">
        <v-icon size="81">mdi-cellphone</v-icon>
      </v-avatar>
      <v-card-title>{{
        deviceInfo.find((info) => info.Name == "ro.product.model").Value
      }}</v-card-title>
      <v-card-text>
        <v-chip class="ma-2">
          <v-avatar left>
            <!--BatteryLevel Between 94 and 5-->
            <template v-if="5 < this.batteryInfo.level < 95">
              <v-icon>
                mdi-battery-{{ Math.round(batteryInfo.level / 10) * 10 }}
              </v-icon>
            </template>
            <!--BatteryLevel 100-->
            <template v-if="this.batteryInfo.level >= 95">
              <v-icon>mdi-battery</v-icon>
            </template>
            <!--BatteryLevel < 5-->
            <template v-if="this.batteryInfo.level < 5">
              <v-icon>mdi-battery-outline</v-icon>
            </template>
          </v-avatar>
          {{ batteryInfo.level }}%
        </v-chip>
        <v-chip class="ma-2">
          <v-avatar left>
            <v-icon>mdi-memory</v-icon>
          </v-avatar>
          {{
            (
              GNACTG(RAMInfo["MemTotal"]) -
              (GNACTG(RAMInfo["MemAvailable"]) + GNACTG(RAMInfo["MemFree"]))
            ).toFixed(1)
            /*Mem Usage*/
          }}
          /
          {{
            GNACTG(RAMInfo["MemTotal"]).toFixed(1)
            /*Mem Total*/
          }}
          GB
        </v-chip>
        <!--Storage-->
        <v-chip class="ma-2">
          <v-avatar left>
            <v-icon>mdi-sd</v-icon>
          </v-avatar>
          {{ storageInfo[2] }} / {{ storageInfo[1] }}
        </v-chip>
        <!--Screen-->
        <v-chip class="ma-2">
          <v-avatar left>
            <v-icon>mdi-cellphone-screenshot</v-icon>
          </v-avatar>
          {{ screenInfo["Physical size"] }}
        </v-chip>
        <!--CPU-->
        <v-chip class="ma-2">
          <v-avatar left>
            <v-icon>mdi-developer-board</v-icon>
          </v-avatar>
          {{ deviceInfo.find((info) => info.Name == "ro.product.board").Value }}
        </v-chip>
      </v-card-text>
    </v-card>
    <v-card outlined tile>
      <div class="d-flex">
        <v-menu offset-y>
          <template v-slot:activator="{ on, attrs }">
            <v-chip v-bind="attrs" v-on="on" label class="ml-3 mt-2">
              {{ showInfo }}
            </v-chip>
          </template>
          <v-list>
            <v-list-item @click="showInfo = 'Battery'"> 电池 </v-list-item>
            <v-list-item @click="showInfo = 'RAM'"> 内存 </v-list-item>
            <v-list-item @click="showInfo = 'Storage'"> 存储空间 </v-list-item>
            <v-list-item @click="showInfo = 'Screen'"> 屏幕 </v-list-item>
          </v-list>
        </v-menu>
      </div>
      <div>
        <!--Battery-->
        <template v-if="showInfo == 'Battery'">
          <v-list>
            <v-list-item
              v-for="(item, index) in batteryInfoFormatted"
              :key="index"
            >
              <v-list-item-content>
                <v-list-item-title>{{ item }}</v-list-item-title>
                <v-list-item-subtitle>{{ index }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </template>
        <!--RAM-->
        <template v-if="showInfo == 'RAM'">
          <v-list>
            <v-list-item v-for="(item, index) in RAMInfoFormatted" :key="index">
              <v-list-item-content>
                <v-list-item-title>{{ item }}</v-list-item-title>
                <v-list-item-subtitle>{{ index }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </template>
        <!--Storage-->
        <template v-if="showInfo == 'Storage'">
          <v-list>
            <v-list-item
              v-for="(item, index) in storageInfoFormatted"
              :key="index"
            >
              <v-list-item-content>
                <v-list-item-title>{{ item }}</v-list-item-title>
                <v-list-item-subtitle>{{ index }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </template>
        <!--Screen-->
        <template v-if="showInfo == 'Screen'">
          <v-list>
            <v-list-item
              v-for="(item, index) in screenInfoForamtted"
              :key="index"
            >
              <v-list-item-content>
                <v-list-item-title>{{ item }}</v-list-item-title>
                <v-list-item-subtitle>{{ index }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </template>
      </div>
    </v-card>
    <v-list two-line dense>
      <v-subheader inset>Your Android Devices</v-subheader>

      <v-list-item v-for="(item, index) in deviceInfo" :key="index">
        <v-list-item-avatar>
          <v-icon class="grey lighten-1" dark>
            mdi-cellphone-information
          </v-icon>
        </v-list-item-avatar>

        <v-list-item-content>
          <v-card-title>{{ item.Value }}</v-card-title>
          <v-card-subtitle>{{ item.Name }}</v-card-subtitle>
        </v-list-item-content>
      </v-list-item>
    </v-list>
  </v-app>
</template>

<script>
const execSync = require("child_process").execSync;
import YAML from "yamljs";

export default {
  name: "AboutDevices",

  props: ["deviceName"],

  data: function() {
    return {
      adb: localStorage.getItem("ADBPath"),
      fastboot: localStorage.getItem("FastbootPath"),
      deviceInfo: [],
      //===
      batteryInfoFormatted: [],
      batteryInfo: [],
      RAMInfoFormatted: [],
      RAMInfo: [],
      storageInfoFormatted: [],
      storageInfo: [],
      screenInfoForamtted: [],
      screenInfo: [],
      showInfo: "Battery",
    };
  },

  created: function() {
    this.GetProp();
    this.GetBattery();
    this.GetRAM();
    this.GetStorage();
    this.GetScreen();
  },

  methods: {
    GetProp: function() {
      execSync(`${this.adb} -s ${this.deviceName} shell getprop`)
        .toString()
        .split("\n")
        .forEach((item) => {
          if (!item.match("product")) return;
          let infoName = "";
          infoName = item.substring(item.indexOf("[")+1, item.indexOf("]"));
          let infoValue = "";
          infoValue = item.substring(item.lastIndexOf("[")+1, item.lastIndexOf("]"));
          this.deviceInfo.push({ 'Name': infoName, 'Value': infoValue });
          return;
        });
      return;
    },
    GetBattery: function() {
      this.batteryInfo = YAML.parse(
        execSync(
          `${this.adb} -s ${this.deviceName} shell dumpsys battery`
        ).toString()
      )["Current Battery Service state"];
      this.batteryInfoFormatted = {
        电池技术: this.batteryInfo["technology"],
        电池温度: `${this.batteryInfo["temperature"] / 10} °C`,
        电池电压: `${this.batteryInfo["voltage"] / 1000} V`,
      };
    },
    GetRAM: function() {
      this.RAMInfo = YAML.parse(
        execSync(
          `${this.adb} -s ${this.deviceName} shell cat /proc/meminfo`
        ).toString()
      );
      this.RAMInfoFormatted = {
        内存总数: this.RAMInfo["MemTotal"],
        剩余可用内存: this.RAMInfo["MemAvailable"],
        空闲内存: this.RAMInfo["MemFree"],
      };
    },
    GNACTG: function(p) {
      return Math.round(p.match(/[0-9]+/) / 100000) / 10;
      //Get Num And Convert To GB
    },
    GetStorage: function() {
      this.storageInfo = execSync(
        `${this.adb} -s ${this.deviceName} shell df -h /sdcard`
      )
        .toString()
        .split("\n")[1]
        .split(" ")
        .filter((x) => {
          return x != "";
        });
      let rootStorageInfo = execSync(
        `${this.adb} -s ${this.deviceName} shell df -h /`
      )
        .toString()
        .split("\n")[1]
        .split(" ")
        .filter((x) => {
          return x != "";
        });
      this.storageInfoFormatted = {
        可用总容量: this.storageInfo[1],
        已用容量: this.storageInfo[2],
        剩余容量: this.storageInfo[3],
        路径: this.storageInfo[5],
        Root目录总容量: rootStorageInfo[1],
        Root目录已用容量: rootStorageInfo[2],
        Root目录剩余容量: rootStorageInfo[3],
      };
    },
    GetScreen: function() {
      this.screenInfo = YAML.parse(
        execSync(
          `${this.adb} -s ${this.deviceName} shell wm density && ${this.adb} -s ${this.deviceName} shell wm size`
        ).toString()
      );
      this.screenInfoForamtted = {
        DPI: this.screenInfo["Physical density"],
        屏幕分辨率: this.screenInfo["Physical size"],
      };
    },
  },
};
</script>
