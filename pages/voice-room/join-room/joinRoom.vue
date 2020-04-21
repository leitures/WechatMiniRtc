<template>
<view>
<view class="container-box">
  <view class="title" :style="'padding-top:' + ((headerHeight + statusBarHeight)/2 - 12) + 'px'">
    <view>语音聊天室</view>
  </view>
  <view class="input-box">
    <input type="number" :value="roomID" maxlength="20" @input="bindRoomID" placeholder="请输入房间号" placeholder-style="color:#ffffff;opacity: 0.55;"></input>
  </view>
  <view class="input-box">
    <input :value="userID" maxlength="20" @input="bindUserID" placeholder="请输入用户名" placeholder-style="color:#ffffff;opacity: 0.55;"></input>
  </view>
  <radio-group class="radio-group" @change="roleChange">
    <label class="radio">
       <radio color="#006eff" value="presenter" checked="true" id="presenter"></radio>主播（可发言）</label>
    <label class="radio">
      <radio color="#006eff" value="audience" id="audience"></radio>观众（仅收听）</label>
  </radio-group>
  <view class="choice-content">
    <label class="label" for="switchDebugMode">
      <text>调试模式</text>
      <switch id="switchDebugMode" color="#006eff" :checked="debugMode" @change="switchDebugMode"></switch>
    </label>
  </view>
</view>
<view class="bottom-btn">
  <button class="btn" @tap="joinRoom" hover-class="none">进入房间</button>
</view>
<cover-image class="close" :style="'top:' + ((headerHeight + statusBarHeight) - 34) + 'rpx'" src="../../../images/back.png" @tap="onBack"></cover-image>
</view>
</template>

<script>
const app = getApp().globalData;

export default {
  data() {
    return {
      roomID: '',
      userID: '',
      role: 'presenter',
      // 用户角色 presenter audience
      tapTime: '',
      headerHeight: getApp().globalData.headerHeight,
      statusBarHeight: getApp().globalData.statusBarHeight,
      lc: '◀︎',
      debugMode: false,
      streamList: [],
      debug: ""
    };
  },

  components: {},
  props: {},

  /**
   * 生命周期函数--监听页面加载
   * @param {*} options 配置项
   */
  onLoad: function (options) {},

  /**
   * 生命周期函数--监听页面初次渲染完成
   */
  onReady: function () {},

  /**
   * 生命周期函数--监听页面显示
   */
  onShow: function () {
    wx.setKeepScreenOn({
      keepScreenOn: true
    });
  },

  /**
   * 生命周期函数--监听页面隐藏
   */
  onHide: function () {},

  /**
   * 生命周期函数--监听页面卸载
   */
  onUnload: function () {},
  methods: {
    // 绑定输房间号入框
    bindRoomID: function (e) {
      this.setData({
        roomID: e.detail.value
      });
    },
    bindUserID: function (e) {
      this.setData({
        userID: e.detail.value
      });
    },
    roleChange: function (e) {
      this.setData({
        role: e.detail.value
      });
    },
    switchDebugMode: function (event) {
      this.setData({
        debugMode: event.detail.value
      });
    },
    radioDebugChange: function (e) {
      this.setData({
        debug: e.detail.value
      });
    },
    // 进入rtcroom页面
    joinRoom: function () {
      // 防止两次点击操作间隔太快
      const nowTime = new Date();

      if (nowTime - this.tapTime < 1000) {
        return;
      }

      if (!this.roomID) {
        wx.showToast({
          title: '请输入房间号',
          icon: 'none',
          duration: 2000
        });
        return;
      }

      if (!this.userID) {
        wx.showToast({
          title: '请输入用户名',
          icon: 'none',
          duration: 2000
        });
        return;
      }

      const reg = /^[0-9a-zA-Z]*$/;

      if (this.userID.match(reg) === null) {
        wx.showToast({
          icon: 'none',
          title: '用户名为英文加数字'
        });
        return;
      }

      if (/^\d*$/.test(this.roomID) === false) {
        wx.showToast({
          title: '只能为数字',
          icon: 'none',
          duration: 2000
        });
        return;
      }

      const url = '../room/room?&roomID=' + this.roomID + '&debugMode=' + this.debugMode + '&userID=' + this.userID + '&role=' + this.role;
      wx.navigateTo({
        url: url
      });
      this.setData({
        'tapTime': nowTime
      });
    },
    onBack: function () {
      wx.navigateBack({
        delta: 1
      });
    }
  }
};
</script>
<style>
@import "./joinRoom.css";
</style>