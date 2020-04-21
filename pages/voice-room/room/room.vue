<template>
<view>
<!--miniprogram/pages/room/room.wxml-->
<!-- 在模版上放置标签 -->
<view class="page-room">
  <view class="bg">
    <view class="title" :style="'padding-top:' + ((headerHeight + statusBarHeight)/2 - 12) + 'px'">
      <text>房间号：{{roomID}}</text>
    </view>
    <view class="user-container">
      <view class="own" v-if="role === 'presenter'" @tap="handleRoleChange">
        <view class="avatar">
          <image src="/static/images/avatar0_100.png" class="avatar-image"></image>
        </view>
        <view class="userID">{{userID}}（自己）</view>
      </view>
      <view class="own" v-if="role === 'audience'">
        <view class="avatar" @tap="handleRoleChange">
        </view>
        <view class="userID">虚位以待</view>
      </view>
      <view class="users">
        <view class="user" v-for="(item, userID) in userList" :key="userID" v-if="item.hasMainAudio">
          <view v-if="role === 'presenter'" class="avatar" :style="'border: 4px solid rgba(0, 110, 255, ' + (item.volume / 100) + ')'">
            <image :src="'../../../images/avatar' + (index + 1) + '_100.png'" class="avatar-image"></image>
          </view>
          <view class="userID">{{item.userID}}</view>
        </view>
        <view class="user" v-for="(item, index) in [1,2,3,4,5,6]" :key="index" v-if="index < 6 - userList.length">
          <view class="avatar-fake"></view>
          <view class="userID">虚位以待</view>
        </view>
      </view>
    </view>
    <view class="role-choose" v-if="showRolePanel">
      <view class="role-panel-text" v-if="role === 'presenter'">确定下麦吗？</view>
      <view class="role-panel-text" v-if="role === 'audience'">确定上麦吗？</view>
      <view class="handle-btn">
        <view class="role-btn confirm" @tap="confirmRoleChange">确认</view>
        <view class="role-btn" @tap="handleRoleChange">取消</view>
      </view>
    </view>
    <view class="setting">
      <view class="btn-normal" @tap="handlePublishAudio" v-if="role === 'presenter'">
        <image :src="presenterConfig.enableMic? '../../../images/audio-true.png': '../../../images/audio-false.png'"></image>
      </view>
      <view class="btn-normal" @tap="handleMuteAllAudio">
        <image :src="!presenterConfig.muteAllAudio? '../../../images/voice.png': '../../../images/voice-false.png'"></image>
      </view>
    </view>
  </view>
  <trtc-room id="trtc-component" :config="rtcConfig"></trtc-room>
</view>
<cover-image class="close" :style="'top:' + ((headerHeight + statusBarHeight) - 34) + 'rpx'" src="../../../images/back.png" @tap="onBack"></cover-image>
</view>
</template>

<script>
import { genTestUserSig } from "../../../debug/GenerateTestUserSig";
const app = getApp().globalData;
import trtcRoom from "../../components/trtc-room/trtc-room";

export default {
  data() {
    return {
      roomID: 0,
      // 房间号
      role: '',
      userList: [],
      // 主播list
      userID: '',
      debugMode: false,
      initialRole: '',
      showRolePanel: false,
      presenterConfig: {
        enableMic: true,
        muteAllAudio: false
      },
      headerHeight: getApp().globalData.headerHeight,
      statusBarHeight: getApp().globalData.statusBarHeight,
      rtcConfig: {
        sdkAppID: '',
        // 必要参数 开通实时音视频服务创建应用后分配的 sdkAppID
        userID: '',
        // 必要参数 用户 ID 可以由您的帐号系统指定
        userSig: '',
        // 必要参数 身份签名，相当于登录密码的作用
        template: '' // 必要参数 组件模版，支持的值 1v1 grid custom ，注意：不支持动态修改, iOS 不支持 pusher 动态渲染

      }
    };
  },

  components: {
    trtcRoom
  },
  props: {},

  /**
   * 生命周期函数--监听页面加载
   * @param {*} options 配置项
   */
  onLoad: function (options) {
    console.log('room onload', options);
    wx.setKeepScreenOn({
      keepScreenOn: true
    }); // 获取 rtcroom 实例

    this.trtcComponent = this.selectComponent('#trtc-component');
    Object.getOwnPropertyNames(options).forEach(key => {
      if (options[key] === 'true') {
        options[key] = true;
      }

      if (options[key] === 'false') {
        options[key] = false;
      }
    }); // 监听TRTC Room 关键事件

    this.bindTRTCRoomEvent();
    this.setData({
      roomID: parseInt(options.roomID),
      role: options.role,
      initialRole: options.role,
      userID: options.userID,
      debugMode: options.debugMode
    }, () => {
      this.getSignature(options.userID);
    });
  },

  /**
   * 生命周期函数--监听页面初次渲染完成
   */
  onReady: function () {
    console.log('room ready');
    wx.setKeepScreenOn({
      keepScreenOn: true
    });
  },

  /**
   * 生命周期函数--监听页面显示
   */
  onShow: function () {
    console.log('room show');
    wx.setKeepScreenOn({
      keepScreenOn: true
    });
  },

  /**
   * 生命周期函数--监听页面隐藏
   */
  onHide: function () {
    console.log('room hide');
  },

  /**
   * 生命周期函数--监听页面卸载
   */
  onUnload: function () {
    console.log('room unload');
    wx.setKeepScreenOn({
      keepScreenOn: false
    });
  },
  methods: {
    enterRoom: function ({
      sdkAppID,
      userSig
    }) {
      const template = 'custom';
      const roomID = this.roomID;
      console.log('* room enterRoom', roomID, template);
      this.rtcConfig = {
        sdkAppID: sdkAppID,
        // 您的实时音视频服务创建应用后分配的 sdkAppID
        userID: this.userID,
        userSig: userSig,
        template: template,
        // 1v1 grid custom
        debugMode: this.debugMode // 非必要参数，打开组件的调试模式，开发调试时建议设置为 true

      };
      this.setData({
        rtcConfig: this.rtcConfig
      }, () => {
        // 进房前决定是否推送视频或音频
        if (this.role === 'presenter') {
          this.trtcComponent.publishLocalAudio();
        } // roomID 取值范围 1 ~ 4294967295


        this.trtcComponent.enterRoom({
          roomID: roomID
        });
      });
    },
    getSignature: function (userID) {
      console.log('* room getSignature', userID);
      const Signature = genTestUserSig(userID);
      this.enterRoom({
        sdkAppID: Signature.sdkAppID,
        userSig: Signature.userSig
      });
    },
    bindTRTCRoomEvent: function () {
      const TRTC_EVENT = this.trtcComponent.EVENT; // 初始化事件订阅

      this.trtcComponent.on(TRTC_EVENT.LOCAL_JOIN, event => {
        console.log('* room LOCAL_JOIN', event); // 进房成功，触发该事件后可以对本地视频和音频进行设置
        // if (this.data.role === 'presenter') {
        //   this.trtcComponent.publishLocalAudio()
        // }
      });
      this.trtcComponent.on(TRTC_EVENT.LOCAL_LEAVE, event => {
        console.log('* room LOCAL_LEAVE', event);
      });
      this.trtcComponent.on(TRTC_EVENT.ERROR, event => {
        console.log('* room ERROR', event);
      }); // 远端用户进房

      this.trtcComponent.on(TRTC_EVENT.REMOTE_USER_JOIN, event => {
        console.log('* room REMOTE_USER_JOIN', event, this.trtcComponent.getRemoteUserList());
        const userList = this.trtcComponent.getRemoteUserList();
        this.handleOnUserList(userList).then(() => {
          console.log(this.userList);
        });
      }); // 远端用户退出

      this.trtcComponent.on(TRTC_EVENT.REMOTE_USER_LEAVE, event => {
        console.log('* room REMOTE_USER_LEAVE', event, this.trtcComponent.getRemoteUserList());
        const userList = this.trtcComponent.getRemoteUserList();
        this.handleOnUserList(userList).then(() => {
          console.log(this.userList);
        });
      }); // 远端用户推送音频

      this.trtcComponent.on(TRTC_EVENT.REMOTE_AUDIO_ADD, event => {
        if (this.userList.length < 6) {
          // 订阅音频
          const data = event.data; // 如果不订阅就不会自动播放音频

          const userList = this.trtcComponent.getRemoteUserList();
          this.handleOnUserList(userList).then(() => {
            console.log(this.userList);
          });
          console.log('* room REMOTE_AUDIO_ADD', event, this.trtcComponent.getRemoteUserList());
          this.trtcComponent.subscribeRemoteAudio({
            userID: data.userID
          });
        }
      }); // 远端用户取消推送音频

      this.trtcComponent.on(TRTC_EVENT.REMOTE_AUDIO_REMOVE, event => {
        console.log('* room REMOTE_AUDIO_REMOVE', event, this.trtcComponent.getRemoteUserList());
        const userList = this.trtcComponent.getRemoteUserList();
        this.handleOnUserList(userList).then(() => {
          console.log(this.userList);
        });
      });
      this.trtcComponent.on(TRTC_EVENT.REMOTE_AUDIO_VOLUME_UPDATE, event => {
        const userID = event.data.target.dataset.userid;
        const volume = event.data.detail.volume;
        this.userList.forEach(item => {
          if (item.userID === userID) {
            item.volume = volume;
          }
        });
        this.setData({
          userList: this.userList
        });
      });
    },
    handleOnUserList: function (userList) {
      return new Promise((resolve, reject) => {
        const newUserList = [];
        let index = 0;
        const oldUserList = this.userList;
        userList.forEach(item => {
          if (item.hasMainAudio) {
            const user = this.judgeWhetherExist({
              userID: item.userID,
              streamType: 'main'
            }, oldUserList);
            index += 1;

            if (user) {
              // 已存在
              newUserList.push(Object.assign(user, {
                index: index
              }));
            } else {
              newUserList.push({
                userID: item.userID,
                streamType: 'main',
                index: index,
                hasMainAudio: item.hasMainAudio,
                volume: 0
              });
            }
          }
        });
        this.setData({
          userList: newUserList
        }, () => {
          console.log('handleOnUserList newUserList', newUserList);
          resolve();
        });
      });
    },
    judgeWhetherExist: function (target, userList) {
      userList.forEach(item => {
        if (target.userID === item.userID && target.streamType === item.streamType) {
          return item;
        }
      });
      return false;
    },

    handlePublishAudio() {
      if (this.presenterConfig.enableMic) {
        this.trtcComponent.unpublishLocalAudio();
        const config = this.presenterConfig;
        config.enableMic = false;
        this.setData({
          presenterConfig: config
        }, () => {
          wx.showToast({
            title: '您已关闭麦克风',
            icon: 'none',
            duration: 500
          });
        });
      } else {
        this.trtcComponent.publishLocalAudio();
        const config = this.presenterConfig;
        config.enableMic = true;
        this.setData({
          presenterConfig: config
        }, () => {
          wx.showToast({
            title: '您已开启麦克风',
            icon: 'none',
            duration: 500
          });
        });
      }
    },

    handleMuteAllAudio() {
      if (this.presenterConfig.muteAllAudio) {
        this.userList.forEach(item => {
          this.trtcComponent.subscribeRemoteAudio({
            userID: item.userID
          });
        });
        const config = this.presenterConfig;
        config.muteAllAudio = false;
        this.setData({
          presenterConfig: config
        }, () => {
          wx.showToast({
            title: '取消禁音成功',
            icon: 'none',
            duration: 500
          });
        });
      } else {
        this.userList.forEach(item => {
          this.trtcComponent.unsubscribeRemoteAudio({
            userID: item.userID
          });
        });
        const config = this.presenterConfig;
        config.muteAllAudio = true;
        this.setData({
          presenterConfig: config
        }, () => {
          wx.showToast({
            title: '禁音成功',
            icon: 'none',
            duration: 500
          });
        });
      }
    },

    /**
     * 返回上一页
     */
    onBack: function () {
      wx.navigateBack({
        delta: 1
      });
    },

    handleRoleChange() {
      if (this.initialRole !== 'presenter') {
        this.setData({
          showRolePanel: !this.showRolePanel
        });
      }
    },

    confirmRoleChange() {
      if (this.role === 'audience') {
        this.trtcComponent.publishLocalAudio().then(() => {
          this.setData({
            role: 'presenter',
            showRolePanel: false
          });
        }).catch(() => {
          wx.showToast({
            icon: 'none',
            title: '上麦失败'
          });
        });
      } else {
        this.trtcComponent.unpublishLocalAudio().then(() => {
          this.setData({
            role: 'audience',
            showRolePanel: false
          });
        }).catch(() => {
          wx.showToast({
            icon: 'none',
            title: '下麦失败'
          });
        });
      }
    }

  }
};
</script>
<style>
@import "./room.css";
</style>