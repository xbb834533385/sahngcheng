<template>
  <div class="detail-container">
    <swiper indicator-dots autoplay circular>
      <swiper-item v-for="(item, index) in goodsDetail.pics" :key="item.goods_id">
        <img :src="item.pics_mid_url" mode="aspectFill">
      </swiper-item>
    </swiper>
    <!-- 商品信息 -->
    <div class="goods-info">
      <div class="top">
        ￥{{goodsDetail.goods_price}}
      </div>
      <div class="mid">
        <div class="left">
          {{goodsDetail.goods_name}}
        </div>
        <div class="right">
          <span class="iconfont icon-shoucang"></span>
          收藏
        </div>
      </div>
      <div class="bottom">
        快递：&nbsp;&nbsp;&nbsp;&nbsp;免运费
      </div>
    </div>
  </div>
</template>

<script>
// 导入ajax
import hxios from '../../utils/index.js'
export default {
  data() {
    return {
      goodsDetail:{}
    }
  },
  async onLoad(options){
    let goods_id = options.goods_id;
    let res = await hxios.get({
      url:`api/public/v1/goods/detail?goods_id=${goods_id}`
    })
    this.goodsDetail = res.data.message;
  }
};
</script>

<style lang="scss">
$uRed: #ff2d4a;
  swiper {
    height: 720rpx;
    swiper-item{
      img{
        display: block;
        height: 100%;
        width: 100%;
      }
    }
  }
  .goods-info{
    background-color: #fff;
    padding-left: 16rpx;
    .top{
      font-size: 46rpx;
      color: $uRed;
      padding: 40rpx 0;
    }
    .mid{
      display: flex;
      .left {
        flex: 1;
        font-size: 30rpx;
        overflow: hidden;
        text-overflow: ellipsis;
        display: -webkit-box;
        -webkit-box-orient: vertical;
        -webkit-line-clamp:2;
        padding-right: 80rpx;
      }
      .right {
        width: 140rpx;
        border-left: 1rpx solid #ddd;
        text-align: center;
        font-size: 24rpx;
        color: #999;
        span{
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
</style>
