<template name="grid">
  <view class="template-grid">
    <view class="grid-scroll-container" @touchstart="_handleGridTouchStart" @touchend="_handleGridTouchEnd">
      <!-- <view id="grid-container-id" class="grid-container {{visibleStreamList.length < 4 ? 'stream-' + visibleStreamList.length : visibleStreamList.length%2 == 0? 'stream-odd':'stream-even'}}"> -->
      <view id="grid-container-id" :class="'grid-container ' + (visibleStreamList.length < 4 ? 'stream-' + visibleStreamList.length : 'stream-3')">
        
        <view :class="'view-container pusher-container ' + (pusher.isVisible && ((gridCurrentPage === 1 && gridPlayerPerPage > 3) || gridPlayerPerPage < 4)?'':'none')">
          <live-pusher class="pusher" :url="pusher.url" :mode="pusher.mode" :autopush="pusher.autopush" :enable-camera="pusher.enableCamera" :enable-mic="pusher.enableMic" :muted="!pusher.enableMic" :enable-agc="pusher.enableAgc" :enable-ans="pusher.enableAns" :enable-ear-monitor="pusher.enableEarMonitor" :auto-focus="pusher.enableAutoFocus" :zoom="pusher.enableZoom" :min-bitrate="pusher.minBitrate" :max-bitrate="pusher.maxBitrate" :video-width="pusher.videoWidth" :video-height="pusher.videoHeight" :beauty="pusher.beautyLevel" :whiteness="pusher.whitenessLevel" :orientation="pusher.videoOrientation" :aspect="pusher.videoAspect" :device-position="pusher.frontCamera" :remote-mirror="pusher.enableRemoteMirror" :local-mirror="pusher.localMirror" :background-mute="pusher.enableBackgroundMute" :audio-quality="pusher.audioQuality" :audio-volume-type="pusher.audioVolumeType" :audio-reverb-type="pusher.audioReverbType" :waiting-image="pusher.waitingImage" :debug="debug" @statechange="_pusherStateChangeHandler" @netstatus="_pusherNetStatusHandler" @error="_pusherErrorHandler" @bgmstart="_pusherBGMStartHandler" @bgmprogress="_pusherBGMProgressHandler" @bgmcomplete="_pusherBGMCompleteHandler"></live-pusher>
          <view class="no-video" v-if="!pusher.enableCamera">
            <image class="image" src="/static/components/trtc-room/template/grid/static/mute-camera-white.png"></image>
          </view>
          <view class="no-audio" v-if="!pusher.enableMic">
            <image class="image" src="/static/components/trtc-room/template/grid/static/mute-mic-white.png"></image>
          </view>
          <view class="operation-bar">
            <view class="btn-normal" @tap="_switchMemberListPanel">
              <image class="btn-image" src="/static/components/trtc-room/template/grid/static/list.png"></image>
            </view>
            <view class="btn-normal" @tap="_switchSettingPanel">
              <image class="btn-image" src="/static/components/trtc-room/template/grid/static/setting.png"></image>
            </view>
            <view class="btn-normal btn-hangup" @tap="_hangUp">
              <image class="btn-image" src="/static/components/trtc-room/template/grid/static/hangup.png"></image>
            </view>
            <view class="btn-normal" @tap="_toggleIMPanel">
            <image class="btn-image" src="/static/components/trtc-room/template/grid/static/im.png"></image>
          </view>
          </view>
        </view>

        <view v-for="(item, streamID) in visibleStreamList" :key="streamID" :class="'view-container player-container ' + (item.isVisible?'':'none')" :id="'player-'+item.streamID" :data-userid="item.userID" :data-streamtype="item.streamType" @tap="_doubleTabToggleFullscreen">
          <live-player class="player" :id="item.streamID" :data-userid="item.userID" :data-streamid="item.streamID" :data-streamtype="item.streamType" :src="item.src" mode="RTC" :autoplay="item.autoplay" :mute-audio="item.muteAudio" :mute-video="item.muteVideo" :orientation="item.orientation" :object-fit="item.objectFit" :background-mute="item.enableBackgroundMute" :min-cache="item.minCache" :max-cache="item.maxCache" :sound-mode="item.soundMode" :enable-recv-message="item.enableRecvMessage" :auto-pause-if-navigate="item.autoPauseIfNavigate" :auto-pause-if-open-native="item.autoPauseIfOpenNative" :debug="debug" @statechange="_playerStateChange" @fullscreenchange="_playerFullscreenChange" @netstatus="_playerNetStatus" @audiovolumenotify="_playerAudioVolumeNotify"></live-player>
          <view class="no-video" v-if="item.muteVideo">
            <image class="image" src="/static/components/trtc-room/template/grid/static/display-pause-white.png"></image>
            <view class="text">
              <p>{{item.userID}}</p>
            </view>
          </view>
          <view class="no-video" v-if="!item.hasVideo && !item.muteVideo">
            <image class="image" src="/static/components/trtc-room/template/grid/static/mute-camera-white.png"></image>
            <view class="text">
              <p>{{item.userID}}</p>
            </view>
            <view class="text">
              <p>对方摄像头未打开</p>
            </view>
          </view>
          <view class="no-audio" v-if="!item.hasAudio">
            <image class="image" src="/static/components/trtc-room/template/grid/static/mute-mic-white.png"></image>
          </view>
          <view class="audio-volume" v-if="item.hasAudio">
            <image class="image" src="/static/components/trtc-room/template/grid/static/micro-open.png"></image>
            <view class="audio-active" :style="'height:' + item.volume + '%'">
              <image class="image" src="/static/components/trtc-room/template/grid/static/audio-active.png"></image>
            </view>
          </view>
          <view class="operation-bar">
            <view class="btn-normal" @tap.stop="_handleSubscribeRemoteAudio" :data-user-i-d="item.userID" :data-stream-type="item.streamType">
              <image class="btn-image" :src="item.muteAudio? './static/speaker-false.png': './static/speaker-true.png'"></image>
            </view>
            <view class="btn-normal" @tap.stop="_handleSubscribeRemoteVideo" :data-user-i-d="item.userID" :data-stream-type="item.streamType">
              <image class="btn-image" :src="item.muteVideo? './static/display-pause-false.png': './static/display-play-true.png'"></image>
            </view>
            <view class="btn-normal" @tap="_toggleFullscreen" :data-user-i-d="item.userID" :data-stream-type="item.streamType">
              <image class="btn-image" src="/static/components/trtc-room/template/grid/static/fullscreen.png"></image>
            </view>
          </view>
          <!-- <progress class="volume-progress" percent="{{item.volume}}" stroke-width="4"/> -->
        </view>
        <view v-for="(item, id) in gridPagePlaceholderStreamList" :key="id" class="view-container player-container player-placeholder">
          <image class="image" src="/static/components/trtc-room/template/grid/static/mute-camera-white.png"></image>
        </view>
      </view>
    </view>
    <!-- <view class="slide-up-tips">
      <image class="image" src="./static/slide-up.png"></image>
      <view class="text">
        <p>上滑查看更多</p>
      </view>
    </view> -->
    <view class="pages-container" v-if="gridPageCount > 1">
      <view v-for="(item, index) in gridPageCount" :key="index" :class="'page-item ' + (index+1 === gridCurrentPage? 'current':'')"></view>
    </view>
    <view :class="'panel memberlist-panel ' + (panelName === 'memberlist-panel' ? '' : 'none')">
      <view @tap="_handleMaskerClick" class="close-btn">X</view>
      <view class="panel-header">成员列表</view>
      <view class="panel-body">
        <view class="panel-tips" v-if="streamList.length === 0">暂无成员</view>
        <scroll-view class="scroll-container" scroll-y="true">
          <view class="member-item" v-for="(item, streamID) in streamList" :key="streamID">
            <view class="member-id">{{item.userID}}</view>
            <view class="member-btns">
              <button class="btn" hover-class="btn-hover" :data-userid="item.userID" :data-streamtype="item.streamType" data-key="objectFit" data-value="fillCrop|contain" @tap="_setPlayerProperty">{{item.objectFit === 'fillCrop'? '填充':'适应'}}</button>
              <button class="btn" hover-class="btn-hover" :data-userid="item.userID" :data-streamtype="item.streamType" data-key="orientation" data-value="vertical|horizontal" @tap="_setPlayerProperty">{{item.orientation === 'vertical'? '竖屏':'横屏'}}</button>
              <button class="btn" hover-class="btn-hover" :data-userid="item.userID" :data-streamtype="item.streamType" @tap="_switchStreamType" v-if="item.streamType === 'main'">{{item._definitionType === 'small'? '小画面':'主画面'}}</button>
              <button class="btn" hover-class="btn-hover" :data-userid="item.userID" :data-streamtype="item.streamType" @tap="_handleSnapshotClick">截屏</button>
            </view>
          </view>
        </scroll-view>
      </view>
    </view>
    <view :class="'panel setting-panel ' + (panelName === 'setting-panel' ? '' : 'none')">
      <view @tap="_handleMaskerClick" class="close-btn">X</view>
      <view class="panel-header">推流设置</view>
      <view class="panel-body">
        <scroll-view class="scroll-container" scroll-y="true">
        <view class="setting-option">
          <view class="label">启用摄像头</view>
          <view class="btn-normal" @tap="_toggleVideo">
            <image class="btn-image" :src="pusher.enableCamera? './static/camera-true.png': './static/camera-false.png'"></image>
          </view>
        </view>
        <view class="setting-option">
          <view class="label">启用麦克风</view>
          <view class="btn-normal" @tap="_toggleAudio">
            <image class="btn-image" :src="pusher.enableMic? './static/audio-true.png': './static/audio-false.png'"></image>
          </view>
        </view>
        <view class="setting-option">
          <view class="label">切换摄像头</view>
          <view class="btn-normal" @tap="switchCamera">
            <image class="btn-image" src="/static/components/trtc-room/template/grid/static/switch.png"></image>
          </view>
        </view>
        <view class="setting-option">
          <view class="label">开启美颜</view>
          <switch class="setting-switch" color="#006eff" :checked="pusher.beautyLevel == 9 ? true: false" data-key="beautyLevel" data-value="9|0" data-value-type="number" @change="_setPuserProperty"></switch>
        </view>
        <view class="setting-option">
          <view class="label">开启AGC</view>
          <switch class="setting-switch" color="#006eff" :checked="pusher.enableAgc" data-key="enableAgc" data-value="true" data-value-type="boolean" @change="_setPuserProperty"></switch>
        </view>
        <view class="setting-option">
          <view class="label">开启ANS</view>
          <switch class="setting-switch" color="#006eff" :checked="pusher.enableAns" data-key="enableAns" data-value="true" data-value-type="boolean" @change="_setPuserProperty"></switch>
        </view>
        <view class="setting-option">
          <view class="label">开启横屏推流</view>
          <switch class="setting-switch" color="#006eff" :checked="pusher.videoOrientation === 'vertical' ? false: true" data-key="videoOrientation" data-value="horizontal|vertical" data-value-type="string" @change="_setPuserProperty"></switch>
        </view>
        </scroll-view>
      </view>
    </view>
    <view :class="'masker ' + (panelName =='' ? 'none' : '')" @tap="_handleMaskerClick"></view>
  </view>
</template>