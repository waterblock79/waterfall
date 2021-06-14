<template>
  <v-app>
    <div class="ma-12 text-center font-weight-light">
	<a href="https://waterfall.js.org/" target="_blank">
      <v-avatar
        ><v-img
  max-height="900"
  max-width="1500"
  src="https://upload.cc/i1/2021/05/23/82jZQR.png"
      ></v-img></v-avatar
      ></a>
      <h1 class="font-weight-light">Waterfall Project</h1>
      <p></p>
      <p>对新手友好的 Android 设备管理工具</p>
    </div>
    <v-divider></v-divider>
    <v-subheader class="ml-3">Path</v-subheader>
    <v-list class="ml-3">
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title>ADB Path</v-list-item-title>
          <v-text-field
            dense
            v-model="adbPath"
            @change="ChangeADBPath"
          ></v-text-field>
        </v-list-item-content>
      </v-list-item>
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title>FASTBOOT Path</v-list-item-title>
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
