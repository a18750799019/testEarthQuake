<template>

  <el-dialog v-bind="$attrs"
             v-dialog-drag
             :visible="dialogVisible"
             @close="close"
             :close-on-click-modal="false"
             :width="width||'80%'">
    <span slot="title"
          class="head">{{title}}</span>
    <slot></slot>
  </el-dialog>

</template>
<script>

import "@/util/ds-directives-dialog-drag";
export default {
  name: "Dialog",
  props: ["title", "dialogVisible", "width"],
  methods: {
    close () {
      this.$emit("close", true);
    }
  }
}
</script>
<style lang="scss" scoped>
::v-deep .el-dialog {
  border-radius: 5px;
}
::v-deep .el-dialog__header {
  background: #e4f1ff;
  height: 42px;
  line-height: 42px;
  padding: 0 0 0 10px;
  text-align: left;
  border-radius: 5px 5px 0 0;
  border-bottom: 1px solid #a6e1ff;
  .el-dialog__headerbtn {
    top: 13px;
    .el-dialog__close {
      top: 10px;
    }
  }
}
::v-deep .el-dialog__body {
  padding: 0;
  border-radius: 0 0 5px 5px;
}

::v-deep .el-dialog__wrapper {
  overflow: hidden;
}
</style>
