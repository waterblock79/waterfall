<template>
  <v-app>
  </v-app>
</template>

<script>
export default {
  name: "FileManager",

  data: function () {
    return {
      adbPath: localStorage.getItem("ADBPath"),
      fastbootPath: localStorage.getItem("FastbootPath")
    };
  },

  methods: {
  }
};
</script>
