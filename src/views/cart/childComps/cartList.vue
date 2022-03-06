<template>
  <div class="cart-list">
    <!-- <li v-for="(item, index) in cartList" :key="index">{{ item }}</li> -->
    <scroll class="content" ref="scroll">
      <cart-list-item
        v-for="(item, index) in cartList"
        :key="index"
        :product="item"
      />
    </scroll>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import Scroll from "../../../components/common/scroll/Scroll.vue";
import CartListItem from "./CartListItem.vue";

export default {
  name: "cartList",
  data() {
    return {};
  },
  components: {
    Scroll,
    CartListItem,
  },
  computed: {
    ...mapGetters(["cartList"]),
  },
  activated() {
    // 处于活跃时刷新状态
    this.$refs.scroll.refresh();
  },
};
</script>

<style scoped>
.cart-list {
  height: calc(100% - 44px - 49px - 40px);
}
.content {
  height: 100%;
  overflow: hidden;
}
</style>