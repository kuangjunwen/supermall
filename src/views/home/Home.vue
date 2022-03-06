<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>

    <tab-control
      :titles="['流行', '新款', '精选']"
      @tabClick="tabClick"
      ref="tabControl"
      class="tab-control"
      v-show="false"
    />

    <scroll
      class="content"
      ref="scroll"
      :probe-type="3"
      @scroll="contentScroll"
      :pull-up-load="true"
      @pullingUp="loadMore"
    >
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad" />
      <recommend-view :recommends="recommends" />
      <featrue-view />
      <tab-control
        :titles="['流行', '新款', '精选']"
        @tabClick="tabClick"
        ref="tabControl"
      />
      <goods-list :goods="showGoods" />
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop" />
  </div>
</template>

<script>
// 导入一些方法
import { getHomeMultidata, getHomeGoods } from "network/home";
import { debounce } from "common/utils";

// 导入子组件
import HomeSwiper from "./childComps/HomeSwiper.vue";
import RecommendView from "./childComps/RecommendView.vue";
import FeatrueView from "./childComps/FeatrueView.vue";

// 导入公用的组件
import NavBar from "components/common/navbar/NavBar.vue";
import TabControl from "../../components/content/tabControl/TabControl.vue";
import GoodsList from "../../components/content/goods/GoodsList.vue";
import Scroll from "../../components/common/scroll/Scroll.vue";
import BackTop from "../../components/content/backTop/BackTop.vue";

export default {
  name: "Home",
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] },
      },
      currentType: "pop",
      isShowBackTop: false,
      tabOffsetTop: 0,
      isTabFixed: false,
      saveY: 0,
    };
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list;
    },
  },
  activated() {
    this.$refs.scroll.refresh();
    this.$refs.scroll.scrollTo(0, this.saveY, 10);
  },
  deactivated() {
    this.saveY = this.$refs.scroll.getScrollY();
  },
  components: {
    NavBar,
    HomeSwiper,
    RecommendView,
    FeatrueView,
    TabControl,
    GoodsList,
    Scroll,
    BackTop,
  },
  created() {
    // 1.请求多个数据
    // 这里写主要逻辑，具体操作放methods方法里
    this.getHomeMultidata();
    // 2.请求商品数据,调用方法，传入类型
    this.getHomeGoods("pop");
    this.getHomeGoods("new");
    this.getHomeGoods("sell");
  },
  mounted() {
    const refresh = debounce(this.$refs.scroll.refresh, 200);
    // 1.监听item中图片加载完成
    this.$bus.$on("itemImageLoad", () => {
      refresh();
    });

    // 2.获取tabControl的offsetTop
    // 所有组件都有一个属性$el: 用于获取组件中的元素
  },
  methods: {
    // 事件监听相关的方法
    tabClick(index) {
      switch (index) {
        case 0:
          this.currentType = "pop";
          break;
        case 1:
          this.currentType = "new";
          break;
        case 2:
          this.currentType = "sell";
          break;
      }
    },

    backClick() {
      // 返回顶部方法，第三个参数是多少毫秒之内完成，默认为0，写了数值能有动画效果
      this.$refs.scroll.scrollTo(0, 0);
    },
    contentScroll(position) {
      // 因为y轴是负值，所以要先变成正数才能进行判断
      // 显示隐藏回到顶部组件
      // 1.判断BackTop是否显示
      this.isShowBackTop = -position.y > 1000;
      // 2.决定tabControl是否吸顶(position: fixed)
      this.isTabFixed = -position.y > this.tabOffsetTop;
    },
    loadMore() {
      this.getHomeGoods(this.currentType);
    },
    swiperImageLoad() {
      this.tabOffsetTop = this.$refs.tabControl.$el.offsetTop;
    },
    // 网络请求相关的方法
    getHomeMultidata() {
      getHomeMultidata().then((res) => {
        // this.result = res;
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      });
    },
    getHomeGoods(type) {
      // 让传入类型的页码加1
      const page = this.goods[type].page + 1;
      getHomeGoods(type, page).then((res) => {
        // console.log(res);
        // 把请求到的数据存储在变量中
        this.goods[type].list.push(...res.data.list);
        // 储存完后，页码加1
        this.goods[type].page += 1;
        // 调用完成加载数据方法
        this.$refs.scroll.finishPullUp();
      });
    },
  },
};
// scoped的作用是给style加上作用域
</script>


<style scoped>
#home {
  /* padding-top: 44px; */
  height: 100vh;
}
.home-nav {
  background-color: var(--color-tint);
  color: #fff;

  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  z-index: 9;
}
img {
  width: 100%;
}
.content {
  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
  right: 0;
}
.tab-control {
  position: relative;

  z-index: 9;
}
.fixed {
  position: fixed;
  top: 44px;
  left: 0;
  right: 0;
}
</style>