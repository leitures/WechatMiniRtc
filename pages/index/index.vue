<template>
<view>
<!--index.wxml-->
<view class="container">
  <view class="title" :style="'padding-top:' + ((headerHeight + statusBarHeight)/2 - 12) + 'px'">
    <text>腾讯实时音视频</text>
  </view>
   <view class="tips">
    <text>以下将展示小程序实时音视频能力，由腾讯云提供技术支持</text>
  </view>
  <view class="guide-box">
    <view class="single-box" v-for="(item, index) in entryInfos" :key="index" :id="index" @tap="handleEntry">
      <image class="icon" mode="aspectFit" :src="item.icon" role="img"></image>
      <view class="label">{{item.title}}</view>
      <view class="desc">{{item.desc}}</view>
    </view>
  </view>
</view>

<view class="logo-box">
  <image class="logo" src="/static/images/logo.png"></image>
</view>
</view>
</template>

<script>
// index.js
// const app = getApp().globalData;
const app = getApp().globalData;

export default {
  data() {
    return {
      template: '1v1',
      headerHeight: getApp().globalData.headerHeight,
      statusBarHeight: getApp().globalData.statusBarHeight,
      entryInfos: [{
        icon: "/static/images/call.png",
        title: "语音聊天室",
        desc: "<trtc-room>",
        navigateTo: "../voice-room/join-room/joinRoom"
      }, {
        icon: "/static/images/doubleroom.png",
        title: "双人通话",
        desc: "<trtc-room>",
        navigateTo: "../videocall/videocall"
      }, {
        icon: "/static/images/multiroom.png",
        title: "多人会议",
        desc: "<trtc-room>",
        navigateTo: "../meeting/meeting"
      }]
    };
  },

  components: {},
  props: {},
  onLoad: function () {},
  methods: {
    selectTemplate: function (event) {
      console.log('index selectTemplate', event);
      this.setData({
        template: event.detail.value
      });
    },
    handleEntry: function (e) {
      let url = this.entryInfos[e.currentTarget.id].navigateTo;
      wx.navigateTo({
        url: url
      });
    }
  }
};
</script>
<style>
@import "./index.css";
</style>