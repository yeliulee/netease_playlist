<template>
  <div id="app" class="app-wrapper">
    <div class="player-wrapper">
      <div
        class="container-background"
        :style="{ 'background-image': 'url(' + musicImage + ')' }"
      ></div>
      <div class="container-mask"></div>
      <div class="container-player">
        <div :class="'container-music-info ' + playerStatus">
          <div class="container-music-name">{{ musicInfo.name }}</div>
          <div class="container-artist-name">{{ musicInfo.artist }}</div>
          <div class="container-music-time">
            <div class="container-music-current-time">
              {{ musicInfo.current }}
            </div>
            <div class="container-music-duration">
              {{ musicInfo.durationFormat }}
            </div>
          </div>
          <div
            class="container-music-duration-flow"
            @mousemove="showInstantTime"
            @mouseout="hideInstantTime"
            @click="setProgress"
          >
            <div class="container-music-instant-time" :style="instantTimeStyle">
              {{ instantTimeFormat }}
            </div>
            <div
              class="container-control-hover"
              :style="{ width: hoverWidth }"
            ></div>
            <div
              class="container-music-active-bar"
              :style="{ width: activeLength }"
            ></div>
          </div>
        </div>
        <div class="container-player-main">
          <div :class="'container-album ' + playerStatus">
            <img
              :src="musicImage"
              alt="歌曲图片"
              :class="'container-music-image ' + playerStatus"
            />
            <div class="container-dot"></div>
          </div>
          <div class="container-controls">
            <i class="control-btn fa fa-backward" @click="playControl"></i>
            <i
              :class="'control-btn fa ' + playIcon"
              @click="playControl('play')"
            ></i>
            <i class="control-btn fa fa-forward" @click="playControl"></i>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      mid: 868773387,
      musicImage:
        "https://p2.music.126.net/iAaRt_l_ussYqkLVuDEpqg==/2931298001578529.jpg",
      musicInfo: {
        url: "",
        name: "走在冷风中",
        artist: "周二珂",
        current: "00:00",
        duration: 0,
        durationFormat: "00:00"
      },
      activeLength: "0%",
      musicStatus: "pause",
      audio: new Audio(),
      hoverWidth: "0%",
      instantTimeStyle: "",
      instantTimeFormat: "00:00",
      musicProgress: 0
    };
  },
  computed: {
    playerStatus: function() {
      if (this.musicStatus === "play") {
        return "active";
      } else {
        return "";
      }
    },
    playIcon: function() {
      if (this.musicStatus === "pause") {
        return "fa-play";
      } else {
        return "fa-pause";
      }
    }
  },
  methods: {
    playControl: function(type) {
      if (type === "play") {
        if (this.musicStatus === "pause") {
          this.audio.play();
          this.musicStatus = "play";
        } else {
          this.audio.pause();
          this.musicStatus = "pause";
        }
      } else {
        this.fetchMusic();
      }
    },
    timeFormat: function(seconds) {
      let durationMinutes = Math.floor(seconds / 60);
      let durationSeconds = Math.floor(seconds - durationMinutes * 60);
      if (durationMinutes < 10) {
        durationMinutes = "0" + durationMinutes;
      }
      if (durationSeconds < 10) {
        durationSeconds = "0" + durationSeconds;
      }
      return durationMinutes + ":" + durationSeconds;
    },
    fetchMusic: function() {
      // mid 歌单ID
      fetch(
        "https://api.uomg.com/api/rand.music?mid=" + this.mid + "&format=json"
      )
        .then(res => {
          return res.text();
        })
        .then(res => {
          let data = JSON.parse(res).data;
          this.musicInfo.url = data.url.replace("http://", "https://");
          this.musicInfo.name = data.name;
          this.musicInfo.artist = data.artistsname;
          this.musicImage = data.picurl.replace("http://", "https://");
          this.audio.src = this.musicInfo.url;
          this.audio.onloadedmetadata = () => {
            this.musicInfo.durationFormat = this.timeFormat(
              this.audio.duration
            );
            this.musicInfo.duration = this.audio.duration;
          };
          if (this.musicStatus === "play") {
            setTimeout(() => {
              this.audio.play();
            }, 300);
          }
        });
    },
    showInstantTime: function(e) {
      this.hoverWidth = e.layerX + "px";
      this.musicProgress =
        this.musicInfo.duration * (e.layerX / e.target.offsetWidth);
      this.instantTimeFormat = this.timeFormat(this.musicProgress);
      this.instantTimeStyle =
        "left:" + e.layerX + "px" + ";display:block;margin-left:-21px";
    },
    hideInstantTime: function() {
      this.instantTimeStyle = "";
      this.hoverWidth = "0px";
      this.musicProgress = 0;
    },
    setProgress: function() {
      this.audio.currentTime = this.musicProgress;
    }
  },
  watch: {
    audio: function(audio) {
      console.log(JSON.stringify(audio));
    }
  },
  mounted() {
    this.audio.loop = false;
    this.fetchMusic();
    // 实时监听播放进度调整播放条和时间
    this.audio.ontimeupdate = () => {
      let currentSeconds = this.audio.currentTime;
      let playProgress = (currentSeconds / this.musicInfo.duration) * 100;
      if (!isNaN(currentSeconds)) {
        this.musicInfo.current = this.timeFormat(currentSeconds);
        this.activeLength = playProgress + "%";
      } else {
        this.musicInfo.current = "00:00";
      }
      if (playProgress === 100) {
        this.activeLength = "0%";
        this.musicStatus = "pause";
        this.musicInfo.current = "00:00";
      }
    };
  }
};
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
  background-repeat: no-repeat;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.app-wrapper {
  display: flex;
  -webkit-box-flex: 1;
  flex: 1 1 auto;
  backface-visibility: hidden;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
  flex-direction: column;
  max-height: 100vh;
  min-height: 100vh;
  max-width: 100%;
}
.player-wrapper {
  height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.container-background {
  z-index: 1;
  top: -35px;
  left: -35px;
  right: -35px;
  bottom: -35px;
  position: fixed;
  filter: blur(35px);
  background-size: cover;
  background-attachment: fixed;
  background-position: 50%;
  background-repeat: no-repeat;
}
.container-mask {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 2;
  background-color: rgba(255, 255, 255, 0.45);
}

@keyframes rotateAlbum {
  0% {
    transform: rotateZ(0deg);
  }
  100% {
    transform: rotateZ(360deg);
  }
}

.container-player {
  z-index: 3;
  width: 430px;
  height: 100px;
  position: relative;
}
.container-player-main {
  position: relative;
  height: inherit;
  background-color: #fff;
  box-shadow: 0 30px 80px #656565;
  border-radius: 15px;
  z-index: 2;
}
.container-music-info {
  position: absolute;
  top: 0;
  right: 15px;
  left: 15px;
  padding: 13px 22px 15px 184px;
  background-color: #fff7f7;
  border-radius: 15px 15px 0 0;
  transition: 0.3s ease top;
  z-index: 1;
}
.container-music-name {
  color: #54576f;
  font-size: 17px;
  font-weight: bold;
}
.container-artist-name {
  color: #acaebd;
  font-size: 13px;
  margin: 2px 0 8px 0;
  flex-wrap: nowrap;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.container-music-time {
  height: 12px;
  margin-bottom: 3px;
  overflow: hidden;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}
.container-music-current-time,
.container-music-duration {
  color: #f86d92;
  font-size: 11px;
  background-color: transparent;
  border-radius: 10px;
  transition: 0.3s ease all;
}
.container-music-duration-flow {
  position: relative;
  height: 4px;
  border-radius: 4px;
  background-color: #ffe8ee;
  cursor: pointer;
}
.container-music-instant-time {
  background-color: #3b3d50;
  position: absolute;
  top: -29px;
  color: #fff;
  font-size: 12px;
  white-space: pre;
  padding: 5px 6px;
  border-radius: 4px;
  display: none;
}
.container-control-hover {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  opacity: 0.2;
  z-index: 2;
  background-color: #3b3d50;
}
.container-music-active-bar {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 0;
  background-color: #fd6d94;
  transition: 0.2s ease width;
  z-index: 1;
  height: 4px;
  border-radius: 4px;
}
.active.container-music-info {
  top: -92px;
}
.container-album {
  position: absolute;
  top: -40px;
  width: 115px;
  height: 115px;
  margin-left: 40px;
  transform: rotateZ(0);
  transition: 0.3s ease all;
  box-shadow: 0 0 0 10px #fff;
  border-radius: 50%;
  overflow: hidden;
}
.container-album:before {
  content: "";
  position: absolute;
  top: 50%;
  right: 0;
  left: 0;
  width: 20px;
  height: 20px;
  margin: -10px auto 0 auto;
  background-color: #d6dee7;
  border-radius: 50%;
  box-shadow: inset 0 0 0 2px #fff;
  z-index: 2;
}
.container-album.active {
  top: -60px;
  box-shadow: 0 0 0 4px #fff7f7, 0 30px 50px -15px #afb7c1;
}
.container-dot {
  position: absolute;
  top: 50%;
  right: 0;
  left: 0;
  height: 13px;
  color: #1f1f1f;
  font-size: 13px;
  font-family: Helvetica, serif;
  text-align: center;
  font-weight: bold;
  line-height: 1;
  padding: 6px;
  margin: -12px auto 0 auto;
  background-color: rgba(255, 255, 255, 0.19);
  opacity: 0;
  z-index: 2;
  transition: 0.1s linear all;
}
.container-music-image {
  display: block;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  opacity: 1;
}
.container-music-image.active {
  z-index: 1;
  -webkit-animation: rotateAlbum 3s linear 0s infinite forwards;
  animation: rotateAlbum 3s linear 0s infinite forwards;
}
.container-controls {
  width: 250px;
  height: 100%;
  margin: 0 5px 0 141px;
  float: right;
  overflow: hidden;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-around;
}
.control-btn .control-btn i {
  transition: 0.2s ease all;
}
.control-btn {
  width: 26px;
  height: 26px;
  padding: 25px;
  display: block;
  color: #d6dee7;
  text-align: center;
  line-height: 1;
  font-size: 26px;
  transition: 0.3s ease all;
  cursor: pointer;
  border-radius: 5px;
}
.control-btn:hover {
  color: #fff;
  background-color: #d6d6de;
}

@media screen and (max-width: 640px) {
  .container-player {
    width: 350px;
  }
  .container-controls {
    width: 225px;
  }
  .container-album {
    width: 100px;
    height: 100px;
    top: -20px;
    margin-left: 25px;
  }
  .container-album.active {
    top: -45px;
  }
  .container-music-info {
    padding-left: 125px;
  }
}
</style>
