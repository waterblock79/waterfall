<template>
  <div class="app">
    <pre style="width:100%;max-height:30em;overflow: scroll;font-size: 85%;" class="mx-3 font-mono">{{ shellLog }}</pre>
    <div class="flex mx-3">
      <input class="flex-grow" type="text" ref="command"/>
      <button @click="RunCommand()" class="ml-3 color-white primary">执行</button>
    </div>
  </div>
</template>

<script>
const exec = require("child_process").exec;
const execSync = require("child_process").execSync;

export default {
  name: "AboutProject",

  data: function () {
    return {
      adbPath: localStorage.getItem("ADBPath"),
      fastbootPath: localStorage.getItem("FastbootPath"),
      shellLog: "",
    };
  },

  methods: {
    RunCommand: function () {
      let command = this.$refs["command"].value;
      if(!(command.indexOf("adb") != -1 || command.indexOf("fastboot") != -1)){
          if(confirm("确定执行一个非 ADB 或 FASTBOOT 命令吗？") == false){
              return;
          }
      }
      exec(command,(stdout,stderr)=>{
          this.shellLog += `\n${execSync("cd").toString().replace("\n","")} >> ${this.$refs["command"].value}\n`;
          if(stdout != null)
            this.shellLog += stdout;
          if(stderr != null)
            this.shellLog += stderr;
      });
      command = ""
    },
  },
};
</script>
