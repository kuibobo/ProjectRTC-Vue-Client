<template>
  <div class="webrtc">
    <div id="videos">
      <video id="me" autoplay></video>
    </div>

    <div class="remoteStreams ng-scope" ng-controller="RemoteStreamsController as rtc">
      <h2>Sky Remote Streams</h2>

      <div id="remoteVideosContainer"></div>

      <table>
        <thead>
        <tr>
          <th>Stream</th><th>1-way</th><th>2-way</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="item in users">
          <td><a>{{item.userId}}</a></td>
          <td><input type="button" @click="handleInvite(item)" value="邀请"><input type="button" @click="handleCall(item)" value="呼叫"></td>
        </tr>
        </tbody>
      </table>

      <li style="width: 240px">
        <input type="button" @click="handleLoadData" value="刷新"/>
      </li>
      <div class="remoteStreams">

        <p class="footer">pierre@chabardes.net</p>

      </div>
    </div>

    <div class="localStream ng-scope" ng-controller="LocalStreamController as localStream">
      <p>Username: <input ng-model="localStream.name" class="ng-pristine ng-untouched ng-valid"></p>
      <video id="localVideo" muted="muted" autoplay="true"></video>
      <li>

      </li>
      <div ng-show="localStream.cameraIsOn" class="ng-hide">
        <p>Share this link:</p>
        <a class="ng-binding"></a>
      </div>
    </div>
  </div>
</template>

<script>
import { SkyRTC } from './SkyRTC-client'

export default {
  name: "HelloWord",
  data() {
    return {
      camera: {},
      users: [],
      isShowVideo: false,
      rtc : undefined,
      user: 'Chrome'
    };
  },
  watch: {
  },
  mounted() {
    let me = this
    this.rtc = SkyRTC();
    this.rtc.connect("ws://127.0.0.1:5000/ws/" + this.user + "/1", this.user);
    this.rtc.on("connected", function (socket) {
      console.info('connected', socket)
      //创建本地视频流
      me.rtc.createStream({
        "video": true,
        "audio": true
      });
    });
    this.rtc.on("stream_created", function (stream) {
      console.info('stream_created')
      document.getElementById('me').srcObject = stream;
      document.getElementById('me').play();
      // 设置本地不播放自己的声音
      document.getElementById('me').volume = 0.0;
    });
    this.rtc.on('pc_add_stream', function (stream, socketId) {
      console.info('pc_add_stream')
      var newVideo = document.createElement("video");
      var videos = document.getElementById("videos");
      var id = "other-" + socketId;
      newVideo.setAttribute("class", "other");
      newVideo.setAttribute("autoplay", "autoplay");
      newVideo.setAttribute("id", id);
      videos.appendChild(newVideo);
      me.rtc.attachStream(stream, id);
    });
    this.rtc.on('remove_peer', function (socketId) {
      var video = document.getElementById('other-' + socketId);
      if (video) {
        video.parentNode.removeChild(video);
      }
    });

    this.rtc.on('new_peer', function (socketId) {
      console.info('new_peer');

      //me.rtc.createPeerConnections();
      //me.rtc.addStreams();
      //me.rtc.addDataChannels();
      me.rtc.sendOffers()
    });

  },
  methods: {
    handleLoadData() {
      const axios = require('axios').default;
      let me = this
      axios.get('http://127.0.0.1:5000/userList')
          .then(function (resp) {
            me.users = resp.data
          })
    },

    handleInvite(user) {
      let me = this
      this.rtc.on("__peers", function (data) {
        // 发送邀请
        me.rtc.sendInvite(me.user, user.userId)
      });
      this.rtc.createRoom(me.user)
    },

    handleCall(user) {

    },
  }
};
</script>

<style rel="stylesheet/scss" lang="scss">
body {
  padding: 30px;
  font: 14px "Lucida Grande", Helvetica, Arial, sans-serif;
}

a {
  color: #00b7ff;
  text-decoration: none;
}

table, tr, td, th {margin: 0;border: 0;padding: 5px;border-collapse: collapse;}

li {
  list-style-type: none;
  border: 2px solid black;
  border-radius: 5px;
  text-align: center;
  box-shadow: 3px 3px 3px #888;
  padding: 5px 5px 5px 5px;
}

li:hover {
  cursor: pointer;
  background-color: #aaa;
}

.localStream {
  display: inline-block;
  top: 15px;
  right: 15px;
  float: right;
}

.localStream video {
  border: 5px solid rgb(119, 119, 119);
  outline: 0px none;
  height: 240px;
  width: 320px;
}

.remoteStreams{
  display: inline-block;
  top: 15px;
  right: 15px;
  float: left;
  text-align: center;
}

.remoteStreams td:hover {
  cursor: pointer;
  background-color: #aaa;
}

.remoteStreams .selected {
  background-color: #444 !important;
  text-shadow:none;
  border-right-color: #aaa;
  border-left: none;
  border-radius: 3px 3px 3px 3px;
  box-shadow:inset 1px 2px 6px #070707;
}

.footer {
  clear: both;
}
</style>
