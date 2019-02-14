<template>
  <div class="cart-container">
    <!-- 空车 -->
    <div class="empty" v-show="!isfull">
      <i class="iconfont icon-gouwuche"></i>
      <p>你还没有添加任何商品</p>
    </div>
    <!-- 非空车 -->
    <div class="full" v-show="isfull">
      <!-- 收货人信息 -->
      <div class="buyer-box" @click="chooseaddress">
        <div class="item">
          <div class="left">收货人：{{buyer}}</div>
          <div class="right">
            {{mobile}}
            <span class="iconfont icon-jiantouyou"></span>
          </div>
        </div>
        <div class="item">收货地址：{{address}}</div>
        <div class="line"></div>
      </div>
      <!-- 商品列表 -->
      <div class="goods-box">
        <div class="title">
          <span class="iconfont icon-dianpu"></span> 优购生活馆
        </div>
        <div class="items">
          <div class="item" v-for="(item, index) in goodsList" :key="item.goods_id">
            <div class="item-left">
              <input
                type="radio"
                :checked="item.isSelected"
                @click="item.isSelected=!item.isSelected"
              >
            </div>
            <div class="item-right">
              <div class="goods-left">
                <img :src="item.goods_small_logo">
              </div>
              <div class="goods-right">
                <div class="name">{{item.goods_name}}</div>
                <div class="bottom">
                  <div class="left">
                    ￥
                    <span>{{item.goods_price}}</span>.00
                  </div>
                  <div class="right">
                    <span @click="item.goods_number>1?item.goods_number--:item.goods_number">-</span>
                    <span>{{item.goods_number}}</span>
                    <span @click="item.goods_number++">+</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!-- 底部操纵区域 -->
      <div class="bottom-box">
        <div class="left">
          <input type="radio" :checked="isCheckAll" @click="checkAll">全选
        </div>
        <div class="right">
          <div class="total-price">
            <div class="top">
              ￥
              <span>{{totalPrice}}</span>.00
            </div>
            <div class="bottom">商家包邮</div>
          </div>
          <button>结算({{totalNum}})</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// 导入hxios
import hxios from "../../utils/index.js";
export default {
  data() {
    return {
      // 是否有商品
      isfull: false,
      // 商品
      goodsList: [],
      // 收货人
      buyer: "",
      // 手机
      mobile: "",
      // 收货地址
      address: ""
    };
  },
  methods: {
    // 获取地址
    chooseaddress() {
      wx.chooseAddress({
        success: res => {
          this.buyer = res.userName;
          this.address =
            res.provinceName +
            " " +
            res.cityName +
            " " +
            res.countyName +
            " " +
            res.detailInfo;
          this.mobile = res.telNumber;
          // 存到缓存里面
          wx.setStorage({
            key: "address",
            data: res
          });
        }
      });
    },
    // 全选方法
    checkAll(){
      // 把所有商品的选中状态 变得和自己一样
      this.isCheckAll = !this.isCheckAll;
    } 
  },
  // 计算属性
  computed: {
    // 全选
    isCheckAll:{
      // 赋值
      // 不会自动触发，因此添加了一个点击的事件来触发set方法
      set(value){
        this.goodsList.forEach(v=>{
          v.isSelected = value;
        })
      },
      get() {
        // 返回布尔值 是否全部被选中了 
        // 遍历每一个goodsList找到isSelected属性，当全部都为true时全选
        return this.goodsList.every(v => {
          return v.isSelected == true;
        });
      }
    },
    // 总数
    totalNum(){
      let num = 0;
      this.goodsList.forEach(v=>{
        // 如果isSelected属性为true就累加goods_number
        if(v.isSelected){
          num += v.goods_number;
        }
      })
      return num;
    },
    // 总金额
    totalPrice(){
      let price = 0;
      this.goodsList.forEach(v=>{
        if(v.isSelected){
          price += v.goods_number*v.goods_price;
        }
      })
      return price;
    }
  },
  // 侦听器
  watch: {
    // 修改个数并保存到缓存中，复杂数据为了能改变保存到缓存中 用到了deep深监听
    goodsList:{
      handler:function (val,oldVal) {
        // 同步更新到缓存中
        let data = {};
        this.goodsList.forEach(v=>{
          data[v.goods_id] = v.goods_number;
        })
        // 覆盖缓存
        wx.setStorage({
          key: 'cart',
          data
        });
      },
      deep:true
    }
  },
  onShow() {
    // 读取缓存
    wx.getStorage({
      key: "cart",
      success: async res => {
        // console.log(res.data)
        // 修改为有商品
        this.isfull = true;
        // 获取缓存中的商品id
        let ids = "";
        for (const key in res.data) {
          ids += key;
          ids += ",";
        }
        // 去掉拼接完成ids的最后一个，号
        ids = ids.slice(0, -1);
        // console.log(ids);

        // 调用接口
        let results = await hxios.get({
          url: `api/public/v1/goods/goodsList?goods_ids=${ids}`
        });
        // 添加字段 个数 是否选中
        results.data.message.forEach(v => {
          // 个数
          v["goods_number"] = res.data[v.goods_id];
          // 是否选中
          v["isSelected"] = false;
        });
        // 把数据保存起来
        this.goodsList = results.data.message;
      },
      fail: () => {},
      complete: () => {}
    });
    // 读取地址缓存
    let address = wx.getStorageSync("address");
    if (address) {
      // console.log("有地址");
      // 赋值
      this.buyer = address.userName;
      this.mobile = address.telNumber;
      this.address =
        address.provinceName +
        " " +
        address.cityName +
        " " +
        address.countyName +
        " " +
        address.detailInfo;
    } else {
      //没有地址
      console.log("没有地址");
    }
  }
}
</script>

<style lang="scss">
$uRed: #ff2d4a;
page {
  height: 100%;
}
.cart-container {
  height: 100%;
  background-color: #eee;
  .empty {
    height: 100%;
    padding-top: 100rpx;
    box-sizing: border-box;
    text-align: center;
    i {
      font-size: 100rpx;
      color: #f9e24d;
      width: 180rpx;
      height: 180rpx;
      margin: 0 auto;
      border-radius: 50%;
      background: linear-gradient(
        to bottom,
        #e89eab,
        #e59b96,
        #d54e55,
        #32462f
      );
      line-height: 180rpx;
    }
    p {
      font-size: 26rpx;
      margin-top: 50rpx;
    }
  }
  .full {
    padding-bottom: 100rpx;
    .buyer-box {
      background-color: #fff;
      .item {
        height: 100rpx;
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 0 30rpx 0 20rpx;
        font-size: 30rpx;
      }
      .line {
        background-image: linear-gradient(
          -45deg,
          blue 20%,
          white 20%,
          white 30%,
          red 30%,
          red 70%,
          white 70%,
          white 80%,
          blue 80%
        );
        background-size: 160rpx 12rpx;
        height: 12rpx;
      }
    }
    .goods-box {
      background-color: #fff;

      .title {
        height: 90rpx;
        border-bottom: 1rpx solid #eee;
        line-height: 90rpx;
        padding-left: 30rpx;
        font-size: 30rpx;
      }
      .items {
        .item {
          height: 210rpx;
          display: flex;
          .item-left {
            width: 85rpx;
            display: flex;
            justify-content: center;
            align-items: center;
          }
          .item-right {
            flex: 1;
            display: flex;
            border-bottom: 1rpx solid #eee;
            .goods-left {
              display: flex;
              align-items: center;
              margin-right: 20rpx;
              img {
                width: 160rpx;
                height: 160rpx;
                display: block;
              }
            }
            .goods-right {
              padding: 20rpx 18rpx 20rpx 0;
              display: flex;
              flex-direction: column;
              justify-content: space-between;
              .name {
                overflow: hidden;
                text-overflow: ellipsis;
                display: -webkit-box;
                -webkit-box-orient: vertical;
                -webkit-line-clamp: 2;
              }
              .bottom {
                display: flex;
                justify-content: space-between;
                .left {
                  color: $uRed;
                  font-size: 20rpx;
                  display: flex;
                  align-items: flex-end;
                  span {
                    font-size: 30rpx;
                  }
                }
                .right {
                  span {
                    display: inline-block;
                    width: 60rpx;
                    height: 50rpx;
                    border: 4rpx solid #666;
                    text-align: center;
                    line-height: 50rpx;
                    &:nth-child(2) {
                      border: none;
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
    // 底部操纵区域布局
    .bottom-box {
      height: 100rpx;
      display: flex;
      justify-content: space-between;
      padding-left: 30rpx;
      align-items: center;
      position: fixed;
      bottom: 0;
      left: 0;
      width: 100%;
      background-color: white;
      .left {
        font-size: 30rpx input {
          margin-right: 30rpx;
        }
      }
      .right {
        display: flex;
        .total-price {
          display: flex;
          flex-direction: column;
          justify-content: space-around;
          .top {
            color: #eb4450;
            font-size: 28rpx;
            // 伪元素
            &::before {
              content: "合计:";
              color: black;
            }
            span {
              font-size: 30rpx;
            }
          }
          .bottom {
            font-size: 28rpx;
            color: #ccc;
          }
        }
        button {
          margin-left: 20rpx;
          width: 230rpx;
          background-color: #eb4450;
          color: white;
          text-align: center;
          line-height: 100rpx;
          border-radius: 0;
          &.disabled {
            background-color: #ccc;
            color: black;
          }
        }
      }
    }
  }
}
</style>
