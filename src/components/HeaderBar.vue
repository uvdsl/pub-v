<template>
  <Toolbar>
    <template #left>
      <h1>RefBee</h1>
    </template>
    <!-- <template #right>  </template> -->
  </Toolbar>
</template>

<script lang="ts">
import { defineComponent } from "vue";
export default defineComponent({
  name: "HeaderBar",
  components: {},
  setup() {},
});
</script>

<style lang="scss">
.p-toolbar-group-left {
  h1 {
    margin-left: 10px;
    font-size: 150%;
    color: var(--primary-color);
  }
}
</style>
