<template>
  <div class="app">
    <div class="ma-10 text-center font-weight-light">
      <img src="../assets/icon.png" style="width: 5em; height: 5em" />
      <h2 class="font-weight-light">Waterfall Project <sub>Alpha</sub></h2>
      <p>By waterblock79</p>
      <p>更美观、更纯净、更强大的开源安卓设备管理器。</p>
    </div>
    <hr class="mx-3"/>
    <div class="mx-4">
      <sub>Settings</sub>
      <div style="height: 0.5em"></div>
      <div class="mx-1">
        <sub class="color-black">ADB.exe 路径</sub><br />
        <input
          type="text"
          class="mb-4 mt-1"
          v-model="adbPath"
          @change="ChangeADBPath"
          style="width: 100%"
          spellcheck="false"
        /><br />
        <sub class="color-black">FASTBOOT.exe 路径</sub><br />
        <input
          type="text"
          class="mb-4 mt-1"
          v-model="fastbootPath"
          @change="ChangeFBPath"
          style="width: 100%"
          spellcheck="false"
        /><br />
      </div>
    </div>
  </div>
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
