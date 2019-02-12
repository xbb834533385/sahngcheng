<template>
  <div class="detail-container">
    <swiper indicator-dots autoplay circular>
      <swiper-item v-for="(item, index) in goodsDetail.pics" :key="item.goods_id">
        <img :src="item.pics_mid_url" mode="aspectFill" @click="showPreviewImg(item.pics_big_url)">
      </swiper-item>
    </swiper>
    <!-- 商品信息 -->
    <div class="goods-info">
      <div class="top">￥{{goodsDetail.goods_price}}</div>
      <div class="mid">
        <div class="left">{{goodsDetail.goods_name}}</div>
        <div class="right">
          <span class="iconfont icon-shoucang"></span>
          收藏
        </div>
      </div>
      <div class="bottom">快递：&nbsp;&nbsp;&nbsp;&nbsp;免运费</div>
    </div>
    <!-- 选项 -->
    <div class="options">
      <div class="option">
        促销
        <span class="red">满300减30</span>
      </div>
      <div class="option">
        已选
        <span>黑色/s/1件</span>
      </div>
    </div>
    <div class="options">
      <div class="option" @click="chooseAddress">
        送至
        <span>
          {{address}}
          <span class="iconfont icon-jiantouyou"></span>
        </span>
      </div>
    </div>
    <!-- tab -->
    <div class="tab-bar">
      <div class="item" :class="{active:index==0}" @click="index=0">图文介绍</div>
      <div class="item" :class="{active:index==1}" @click="index=1">规格参数</div>
    </div>
    <div class="tab-content">
      <div class="item" v-show="index==0" v-html="goodsDetail.goods_introduce"></div>
      <div class="item" v-show="index==1">
        <div class="attrs" v-for="(item, index) in goodsDetail.attrs" :key="item.attr_id">
          <div class="attr-name">{{item.attr_name}}</div>
          <div class="attr-value">{{item.attr_value}}</div>
        </div>
      </div>
    </div>
    <!-- 底部操纵区域 -->
    <div class="bottom-box">
      <div class="service">
        <span class="iconfont icon-kefu"></span>
        联系客服
      </div>
      <div class="cart">
        <span class="iconfont icon-gouwuche" @click="toCart"></span>
        购物车
      </div>
      <button>加入购物车</button>
      <button>立即购买</button>
    </div>
  </div>
</template>

<script>
// 导入ajax
import hxios from "../../utils/index.js";
export default {
  data() {
    return {
      // 商品详情
      goodsDetail: {},
      // 送货地址
      address: "",
      index: 0
    };
  },
  async onLoad(options) {
    let goods_id = options.goods_id;
    let res = await hxios.get({
      url: `api/public/v1/goods/detail?goods_id=${goods_id}`
    });
    this.goodsDetail = res.data.message;
    // 获取缓存
    wx.getStorage({
      key: "address",
      success: res => {
        this.address = res.data.provinceName + " " + res.data.cityName + ' ' + res.data.countyName;
      }
    });
  },
  methods: {
    // 显示大图预览 微信自带的方法
    showPreviewImg(pics_big_url) {
      // 创建一个新数组 根据返回内容
      let urls = this.goodsDetail.pics.map(v => v.pics_big_url);
      // 微信自带的api
      wx.previewImage({
        urls,
        current: pics_big_url
      });
    },
    chooseAddress() {
      // 使用微信自带的api
      wx.chooseAddress({
        success: res => {
          console.log(res.provinceName);
          console.log(res.cityName);
          console.log(res.countyName);
          this.address =
            res.provinceName + " " + res.cityName + " " + res.countyName;
          // 缓存起来
          wx.setStorage({
            key: "address",
            data: res
          });
        }
      });
    },
    // 去购物车 因为购物车是用tabbar管理的页面 所以不能用navigateTo跳转
    toCart(){
      wx.switchTab({url:"/pages/cart/main"})
    }
  }
};
</script>

<style lang="scss">
$uRed: #ff2d4a;
.detail-container {
  background-color: #eee;
  margin-bottom: 90rpx;
}
swiper {
  height: 720rpx;
  swiper-item {
    img {
      display: block;
      height: 100%;
      width: 100%;
    }
  }
}
.goods-info {
  background-color: #fff;
  padding-left: 16rpx;
  .top {
    font-size: 46rpx;
    color: $uRed;
    padding: 40rpx 0;
  }
  .mid {
    display: flex;
    .left {
      flex: 1;
      font-size: 30rpx;
      overflow: hidden;
      text-overflow: ellipsis;
      display: -webkit-box;
      -webkit-box-orient: vertical;
      -webkit-line-clamp: 2;
      padding-right: 80rpx;
    }
    .right {
      width: 140rpx;
      border-left: 1rpx solid #ddd;
      text-align: center;
      font-size: 24rpx;
      color: #999;
      span {
        display: block;
      }
    }
  }
  .bottom {
    font-size: 28rpx;
    color: #999;
    padding: 30rpx 0;
  }
}
.options {
  margin-top: 20rpx;
  background-color: #fff;
  .option {
    height: 100rpx;
    line-height: 100rpx;
    padding-left: 15rpx;
    font-size: 30rpx;
    span {
      color: #998d92;
    }
    span.red {
      color: $uRed;
    }
  }
}
// tab 
.tab-bar {
  display: flex;
  height: 100rpx;
  background-color: #fff;
  .item{
    flex: 1;
    line-height: 100rpx;
    font-size: 26rpx;
    text-align: center;
    &.active {
      color: $uRed;
      border-bottom: 10rpx solid $uRed;
    }
  }
}
.tab-content{
  background-color: #fff;
  .item {
    padding: 10rpx;
    .attrs{
      display: flex;
      text-align: center;
      height: 88rpx;
      line-height: 88rpx;
      font-size: 22rpx;
      margin-top: -1rpx;
      .attr-name{
        flex: 1;
        border: 1rpx solid #ccc;
      }
      .attr-value {
        flex:1;
        border: 1rpx solid #ccc;
        margin-left: -1rpx;
      }
    }
  }
}
.bottom-box {
  display: flex;
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 90rpx;
  text-align: center;
  background-color: #fff;
  .iconfont {
    display: block;
  }
  .service{
    flex: 2;
    font-size: 18rpx;
    padding-top: 15rpx;
  }
  .cart{
    flex: 2;
    font-size: 18rpx;
    padding-top: 15rpx;
  }
  button {
    flex: 3;
    background-color: #ffb400;
    color: #fff;
    border-radius: 0;
    border: 0;
    &:last-child{
      background-color: $uRed;
    }
  }
}
</style>
