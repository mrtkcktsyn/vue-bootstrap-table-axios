<template>
  <div id="app">
    <app-datatable  />
  </div>
</template>

<script>
import Datatable from "./components/Datatable.vue";

export default {
  name: "App",

  components: {
    appDatatable: Datatable,
  },

  data: () => ({
    items: [],
  }),
};
</script>

<style lang="scss">
div {
  box-sizing: border-box;
}
</style>
