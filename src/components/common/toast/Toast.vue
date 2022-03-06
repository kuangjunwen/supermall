<template>
  <div class="toast" v-show="isShow">
    <div>{{ message }}</div>
  </div>
</template>

<script>
export default {
  name: "Toast",
  props: {
    // message: {
    //   type: String,
    //   default: "",
    // },
    // show: {
    //   type: Boolean,
    //   default: false,
    // },
  },
  data() {
    return {
      message: "",
      isShow: false,
    };
  },
  methods: {
    show(message, duration = 1500) {
      this.isShow = true;
      this.message = message;
      setTimeout(() => {
        this.isShow = false;
        this.message = "";
      }, duration);
    },
  },

  components: {},
};
</script>

<style>
.toast {
  position: fixed;
  top: 50%;
  left: 50%;
  /* 根据自身宽度减去一半 */
  transform: translate(-50%, -50%);
  padding: 8px 10px;

  z-index: 999;

  border-radius: 10px;
  color: #fff;
  background-color: rgba(0, 0, 0, 0.75);
}
</style>