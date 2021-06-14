<template>
  <div>
    <div class="flex" style="width: 100%">
      <div
        class="error color-white mx-5 pa-3"
        style="width: 100%; height: 30%; opacity: 0.9; border-radius: 7px"
      >
        <span class="material-icons mr-1">warning</span>
        刷机时会丢失您的数据和手机原本带有的系统！若刷机不当可能会导致手机<b>损坏</b>！
      </div>
      <button
        class="color-white primary ma-5 pa-3 flex-grow"
        style="height: 100%"
        @click="OpenFileSelect('flashsystem')"
      >
        <span class="material-icons mr-3">android</span>
        刷入 System 分区<br />
        <small>该分区常常用于存储手机操作系统相关文件</small>
      </button>
      <input
        type="file"
        class="ghost"
        @change="Flash('system', $event)"
        ref="flashsystem"
      />
      <button
        class="color-white primary ma-5 pa-3 flex-grow"
        style="height: 100%"
        @click="OpenFileSelect('flashrecovery')"
      >
        <span class="material-icons mr-3">device_hub</span>
        刷入 Recovery 分区<br />
        <small>Recovery 中您可以进行刷写系统等高级操作。</small>
      </button>
      <input
        type="file"
        class="ghost"
        @change="Flash('recovery_ramdisk', $event)"
        ref="flashrecovery"
      />
    </div>
    <div class="flex" style="width: 100%">
      <button
        class="color-white primary ma-3 pa-2 flex-grow"
        style="height: 100%"
        @click="OpenFileSelect('flashboot')"
      >
        <span class="material-icons mr-2">sd_card</span>
        刷入 Boot 分区<br />
      </button>
      <input
        type="file"
        class="ghost"
        @change="Flash('boot', $event)"
        ref="flashboot"
      />
      <button
        class="color-white primary ma-3 pa-2 flex-grow"
        style="height: 100%"
        @click="OpenFileSelect('flashkernel')"
      >
        <span class="material-icons mr-2">memory</span>
        刷入 Kernel 分区<br />
      </button>
      <input
        type="file"
        class="ghost"
        @change="Flash('kernel', $event)"
        ref="flashkernel"
      />
      <button
        class="color-white gray ma-3 pa-2 flex-grow"
        style="height: 100%"
        @click="RebootToSystem()"
      >
        <span class="material-icons mr-2">phone_android</span>
        重启到系统<br />
      </button>
    </div>
    <transition name="dialog">
      <div class="dialog" v-show="showFlash">
        <div class="dialog-overlay" />
        <div class="dialog-content card">
          <span class="card-title">Fastboot flash</span>
          <pre class="card-content font-mono word-break-all">{{
            flashLog
          }}</pre>
          <div class="card-active">
            <button class="color-primary" @click="showFlash = false">
              关闭
            </button>
          </div>
        </div>
      </div>
    </transition>
    <hr class="mx-2 my-2" />
    <div class="mx-4">
      <sub
        >特别镜像<span class="color-error" style="opacity: 0.8"
          >（请确保<b>设备版本为 8.0 以上</b>！）</span
        ></sub
      ><br />
      <button class="mt-4 mb-1 primary color-white" style="width: 100%" @click="OpenDownload('7to')">
        <span class="material-icons mr-2 center-y">change_history</span>下载
        Magisk Recovery</button
      ><br />
      <sub class="color-black"
        >（感谢酷安 @麦麦观饭 ）由 Magisk 修补过的 Recovery 镜像，同时适用于
        Android 11 及以上的 GSI，刷入后，通过 Recovery 方式打开手机，即可进入 Magisk 修补版本系统！</sub
      ><br />
      <button class="mt-4 mb-1 primary color-white" style="width: 100%" @click="OpenDownload('magisk')">
        <span class="material-icons mr-2 center-y">developer_board</span>下载
        华为设备奇兔 Recovery</button
      ><br />
      <sub class="color-black"
        >（感谢酷安 @麦麦观饭 ）为华为设备适配的通用 奇兔 Recovery</sub
      ><br />
    </div>
  </div>
</template>

<script>
const exec = require("child_process").exec;

export default {
  name: "Fastboot",

  props: ["deviceName"],

  data: function () {
    return {
      adbPath: localStorage.getItem("ADBPath"),
      fastbootPath: localStorage.getItem("FastbootPath"),
      showFlash: false,
      flashLog: "",
    };
  },

  created: function () {
    console.log();
  },

  methods: {
    OpenFileSelect: function (name) {
      this.$refs[name].click();
    },
    Flash: function (name, e) {
      // Get file selected
      let file = e.target.files;
      if (file.length <= 0) return;
      // Remove file select
      for (let index in this.$refs) {
        //Match refs="flashxxxxxxx" in <button> and <input type="file">
        if (index.indexOf("flash") == -1) return;
        this.$refs[index].file = [];
      }
      // FLASH
      this.showFlash = true;
      this.flashLog = `刷写 ${file[0].name} 中... \n这大约会耗费 ${Math.floor(
        file[0].size / 20000000
      )} 秒\n`;
      /*
      Name       Size             Time      Size÷Time
      System     2470769208       103.025   23982229
      Recovery   15998976         0.673     23772624
      Kernel     25165824         1.207     20849895
      */
      exec(
        `${this.fastbootPath} -s ${this.deviceName} flash ${name} ${file[0].path}`,
        (stdout, stderr, err) => {
          if (stdout == null) stdout = stderr;
          if (stdout == null || stdout == "") stdout = err.toString();
          if (stdout.toString().indexOf("FAILED") != -1) {
            this.flashLog += `出错了？！`;
            if (stdout.toString().indexOf("sparse flash write failure") != -1) {
              this.flashLog += `或许是您的该分区容量较小，不足以刷进这个镜像？`;
            }
            this.flashLog += `\n`;
          }
          this.flashLog += `${stdout}\n`;
        }
      );
    },
    OpenDownload : function (x) {
      if(x == "7to"){
        exec(`explorer "https://waterfall.js.org/images/kirin7to.img"`)
      }
      if(x == "magisk"){
        exec(`explorer "https://waterfall.js.org/images/magisk.img"`)
      }
    },
    RebootToSystem: function () {
      exec(`${this.fastbootPath} -s ${this.deviceName} reboot`, {
        timeout: 10000,
      });
    },
  },
};
</script>
