<template>
  <div class="check-button" :class="{ check: isChecked }">
    <img src="~assets/img/common/checkbutton.svg" alt="" />
  </div>
</template>

<script>
export default {
  name: "CheckButton",
  props: {
    isChecked: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {};
  },
  components: {},
};
</script>

<style>
.check-button {
  border-radius: 50%;
  border: 1px solid #aaa;
}
.check {
  border-color: red;
  background-color: red;
}
.check-button img {
  width: 21px;
}
</style>