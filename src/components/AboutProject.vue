<template>
  <v-app>
    <div class="ma-10 text-center font-weight-light">
      <v-avatar
        ><v-icon x-large color="primary"
          >mdi-android-debug-bridge</v-icon
        ></v-avatar
      >
      <h2 class="font-weight-light">Waterfall Project <sub>Alpha</sub></h2>
      <p>By waterblock79</p>
      <p>更美观、更纯净、更强大的开源安卓设备管理器。</p>
    </div>
    <v-divider></v-divider>
    <v-subheader class="ml-3">Settings</v-subheader>
    <v-list class="ml-3">
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title>ADB.exe 路径</v-list-item-title>
          <v-text-field
            dense
            v-model="adbPath"
            @change="ChangeADBPath"
          ></v-text-field>
        </v-list-item-content>
      </v-list-item>
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title>FASTBOOT.exe 路径</v-list-item-title>
          <v-text-field
            dense
            v-model="fastbootPath"
            @change="ChangeFBPath"
          ></v-text-field>
        </v-list-item-content>
      </v-list-item>
    </v-list>
  </v-app>
</template>

<script>
export default {
  name: "AboutProject",

  data: function () {
    return {
      adbPath: localStorage.getItem("ADBPath"),
      fastbootPath: localStorage.getItem("FastbootPath"),
    };
  },

  methods: {
    ChangeADBPath: function () {
      localStorage.setItem("ADBPath", this.adbPath);
      return;
    },
    ChangeFBPath: function () {
      localStorage.setItem("FastbootPath", this.fastbootPath);
      return;
    },
  },
};
</script>
