<template>
  <div class="webrtc">
    <div class="remoteStreams ng-scope" ng-controller="RemoteStreamsController as rtc">
      <h2>WebRTC Remote Streams</h2>

      <div id="remoteVideosContainer"></div>

      <table>
        <thead>
        <tr>
          <th>Stream</th><th>1-way</th><th>2-way</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="item in streams">
          <td><a>{{item.name}}</a></td>
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
        <input type="button" @click="handleShowCam" v-if="!isShowVideo" value="开始"/>
        <input type="button" @click="handleCloseCam" v-else value="关闭" />
      </li>
      <div ng-show="localStream.cameraIsOn" class="ng-hide">
        <p>Share this link:</p>
        <a class="ng-binding"></a>
      </div>
    </div>
  </div>
</template>

<script>
import { PeerManager, Peer, requestUserMedia, attachMediaStream } from './webrtc-client'

export default {
  name: "HelloWord",
  data() {
    return {
      camera: {},
      streams: [],
      isShowVideo: false,
      client : undefined,
    };
  },
  watch: {
  },
  mounted() {
    console.info("created")
    let me = this
    me.camera.preview = window.document.getElementById('localVideo');
    me.client = new PeerManager('http://127.0.0.1:3001/');

    var mediaConfig = {
      audio:true,
      video: {
        mandatory: {},
        optional: []
      }
    };

    me.$root.eventHub.$on('cameraIsOn', (data)=>{
      console.info('cameraIsOn', data)
      me.isShowVideo = data
    })

    me.camera.start = function(){
      return requestUserMedia(mediaConfig)
          .then(function(stream){
            attachMediaStream(me.camera.preview, stream);
            me.client.setLocalStream(stream);
            me.camera.stream = stream;
            me.$root.eventHub.$emit('cameraIsOn', true)
          })
          .catch(Error('Failed to get access to local media.'));
    };
    me.camera.stop = function(){
      return new Promise(function(resolve, reject){
        try {
          me.camera.stream.getTracks().forEach(t => t.stop())
          me.camera.preview.src = '';
          resolve();
        } catch(error) {
          reject(error);
        }
      })
          .then(function(result){
            me.$root.eventHub.$emit('cameraIsOn', false)
          });
    };
  },
  methods: {
    getStreamById (id) {
      let me = this
      for (var i = 0; i < me.streams.length; i++) {
        if (me.streams[i].id === id) {
          return me.streams[i];
        }
      }
    },

    handleLoadData() {
      const axios = require('axios').default;

      let me = this
// Make a request for a user with a given ID
      axios.get('http://127.0.0.1:3002/clients')
          .then(function (resp) {
            // handle success
            for(var i=0; i<resp.data.data.length;i++) {
              var stream = me.getStreamById(resp.data.data[i].id);
              resp.data.data[i].isPlaying = (!!stream) ? stream.isPLaying : false;
            }
            me.streams = resp.data.data
            console.log(resp.data.data);
          })
    },

    handleShowCam() {
      let me = this
      me.camera.start().then(function(result) {
        console.info('readyToStream')
        me.client.send('readyToStream', { name: 'ChromeX' });
      })
          .catch(function(err) {
            console.log(err);
          });
    },
    handleCloseCam() {
      let me = this
      me.camera.stop()
          .then(function(result){
            me.client.send('leave');
            me.client.setLocalStream(null);
          })
          .catch(function(err) {
            console.log(err);
          });
    },
    handleInvite(stream) {
      let me = this
      this.client.invite(stream.id, function() {
        me.handleCall(stream)
      }, function() {
        alert('reject')
      }, function() {
        stream.isPlaying = false
      });
    },

    handleCall(stream) {
      let me = this
      me.camera.start()
          .then(function(result) {
            me.client.toggleLocalStream(stream.id);
            if(stream.isPlaying){
              me.client.peerRenegociate(stream.id);
            } else {
              me.client.peerInit(stream.id);
            }
            stream.isPlaying = stream.isPlaying == undefined ? true : !stream.isPlaying;
          })
          .catch(function(err) {
            console.log(err);
          });
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
