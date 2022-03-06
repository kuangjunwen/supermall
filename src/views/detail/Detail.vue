<template>
  <div id="detail">
    <detail-nav-bar @titleClick="titleClick" class="detail-nav" ref="nav" />
    <scroll class="conten" ref="scroll" :probeType="3" @scroll="contentScroll">
      <detail-swiper :top-images="topImages"></detail-swiper>
      <detail-base-info :goods="goods" />
      <detail-shop-info :shop="shop" />
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad" />
      <detail-param-info ref="params" :param-info="paramInfo" />
      <detail-comment-info ref="comment" :comment-info="commentInfo" />
      <goods-list ref="recommend" :goods="recommends" />
    </scroll>
    <detail-bottom-bar @addCart="addToCart" />
    <!-- <back-top @click.native="backClick" v-show="isShowBackTop" /> -->
    <!-- <toast :message="message" :show="show" /> -->
  </div>
</template>

<script>
import {
  getDetail,
  Goods,
  Shop,
  GoodsParm,
  getRecommend,
} from "network/detail";
import { debounce } from "common/utils";

import Scroll from "../../components/common/scroll/Scroll.vue";

import DetailNavBar from "./childComps/DetailNavBar.vue";
import DetailSwiper from "./childComps/DetailSwiper.vue";
import DetailBaseInfo from "./childComps/DetailBaseInfo.vue";
import DetailShopInfo from "./childComps/DetailShopInfo.vue";
import DetailGoodsInfo from "./childComps/DetailGoodsInfo.vue";
import DetailParamInfo from "./childComps/DetailParamInfo.vue";
import DetailCommentInfo from "./childComps/DetailCommentInfo.vue";
import GoodsList from "../../components/content/goods/GoodsList.vue";
import DetailBottomBar from "./childComps/DetailBottomBar.vue";

import { backTopMixin } from "common/mixin";
import { mapActions } from "vuex";
// import Toast from "../../components/common/toast/Toast.vue";

export default {
  name: "Detail",
  data() {
    return {
      iid: null,
      topImages: [],
      goods: {},
      shop: {},
      detailInfo: {},
      paramInfo: {},
      commentInfo: {},
      recommends: [],
      themTopYs: [],
      getThemeTopY: {},
      currentIndex: 0,
      // isShowBackTop: fasle,
    };
  },
  mixins: [backTopMixin],
  created() {
    // 1.保存传入的iid
    this.iid = this.$route.params.iid;
    // 2.根据iid请求详情数据
    getDetail(this.iid).then((res) => {
      // console.log(res);
      // 1.获取顶部的图片轮播数据
      const data = res.result;
      this.topImages = data.itemInfo.topImages;

      // 2.获取商品信息
      this.goods = new Goods(
        data.itemInfo,
        data.columns,
        data.shopInfo.services
      );

      // 3.创建店铺信息的对象
      this.shop = new Shop(data.shopInfo);

      // 4.保存商品的详情数据
      this.detailInfo = data.detailInfo;

      // 5.获取参数信息
      this.paramInfo = new GoodsParm(
        data.itemParams.info,
        data.itemParams.rule
      );

      // 6.取出评论信息
      if (data.rate.cRate !== 0) {
        this.commentInfo = data.rate.list[0];
      }

      // this.$nextTick(() => {
      //   this.themTopYs = [];
      //   this.themTopYs.push(0);
      //   this.themTopYs.push(this.$refs.params.$el.offsetTop);
      //   this.themTopYs.push(this.$refs.comment.$el.offsetTop);
      //   this.themTopYs.push(this.$refs.recommend.$el.offsetTop);
      //   console.log(this.themTopYs);
      // });
    });

    // 3.请求推荐数据
    getRecommend().then((res) => {
      // console.logs(res);
      this.recommends = res.data.list;
    });

    // 4.给getThemeTopY赋值
    this.getThemeTopY = debounce(() => {
      this.themTopYs = [];
      this.themTopYs.push(0);
      this.themTopYs.push(this.$refs.params.$el.offsetTop);
      this.themTopYs.push(this.$refs.comment.$el.offsetTop);
      this.themTopYs.push(this.$refs.recommend.$el.offsetTop);
      // console.log(this.themTopYs);
    }, 200);
  },
  mounted() {},
  methods: {
    ...mapActions(["addCart"]),
    imageLoad() {
      this.$refs.scroll.refresh();
      this.getThemeTopY();
    },
    titleClick(index) {
      console.log(index);
      this.$refs.scroll.scrollTo(0, -this.themTopYs[index], 100);
    },
    contentScroll(position) {
      // 1.获取y值
      const positioniY = -position.y;

      // 2.positionY和主题中值进行对比
      let length = this.themTopYs.length;
      for (let i = 0; i < length; i++) {
        if (
          this.currentIndex !== i &&
          ((i < length - 1 &&
            positioniY >= this.themTopYs[i] &&
            positioniY < this.themTopYs[i + 1]) ||
            (i === length - 1 && positioniY >= this.themTopYs[i]))
        ) {
          this.currentIndex = i;
          // console.log(this.currentIndex);
          this.$refs.nav.currentIndex = this.currentIndex;
        }
      }

      // 3.是否显示回到顶部
      this.isShowBackTop = -position.y > 1000;
    },
    addToCart() {
      // 1.获取购物车需要展示的信息
      const product = {};
      product.image = this.topImages[0];
      product.title = this.goods.title;
      product.desc = this.goods.desc;
      product.price = this.goods.realPrice;
      product.iid = this.iid;

      // 2.将商品添加到购物车里面
      // this.$store.commit("addCart", product);
      this.addCart(product).then((res) => {
        // this.show = true;
        // this.message = res;

        // setTimeout(() => {
        //   this.show = false;
        //   this.message = "";
        // }, 1500);

        this.$toast.show(res, 1000);
        // console.log(this.$toast);
      });
      // this.$store.dispatch("addCart", product).then((res) => {
      //   console.log(res);
      // });
    },
  },
  components: {
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    Scroll,
    DetailGoodsInfo,
    DetailParamInfo,
    DetailCommentInfo,
    GoodsList,
    DetailBottomBar,
    // Toast,
    // BackTop,
  },
};
</script>

<style>
#detail {
  position: relative;
  z-index: 9;
  background-color: #fff;
  height: 100vh;
}
.detail-nav {
  position: relative;
  z-index: 9;
  background-color: #fff;
}
.conten {
  height: calc(100% - 44px - 49px);
}
</style>