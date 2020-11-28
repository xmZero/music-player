<template>
  <div class="home">
    <div class="content-wrap">
      <div class="song-img">
        <div class="img-wrapper">
          <img :src="imgSrc" alt="" />
        </div>
      </div>
      <div class="song-info">
        <h1 class="song-name">{{ currentSong.song }}</h1>
        <h3 class="singer-name">{{ currentSong.singer }}</h3>
        <h3 class="album-name">{{ currentSong.album }}</h3>
      </div>
      <div class="play-area">
        <div class="cur-time">{{ curTimeString }}</div>
        <div class="pro-wrapper">
          <div class="pro-bottom"></div>
          <div class="pro-top" :style="processorStyle">
            <span class="slide-point"> </span>
          </div>
        </div>
        <div class="all-time">{{ allTime }}</div>
      </div>
      <div class="play-control">
        <div
          class="btn-wrap like-btn"
          @click="changeLikeStatus"
          :class="{ liked: currentSong.isLike }"
        ></div>
        <div class="btn-wrap prev-btn" @click="prevSong"></div>
        <div
          class="btn-wrap play-btn"
          :class="{ playing: isPlaying }"
          @click="changePlayStatus"
        ></div>
        <div class="btn-wrap next-btn" @click="nextSong"></div>
        <div class="btn-wrap list-btn" @click="showPlayList"></div>
      </div>
    </div>
    <div id="like-tip" v-show="isShowLikeTip">
      {{ currentSong.isLike ? "已添加为我喜欢的歌曲" : "取消喜欢成功" }}
    </div>
    <div class="play-list" :class="{show:isShowPlayList}">
      <div class="line-head">播放列表</div>
      <ul class="play-list-wrap">
        <li v-for="(song, index) in songList" :key="song.song" @click="selectSong(index)">
          <h3>{{song.song}}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span>{{song.singer}}</span></h3>
        </li>
      </ul>
      <div class="close-btn" @click="closePlayList">关闭</div>
    </div>
  </div>
</template>
<script>
import axios from "axios";

export default {
  name: "Home",
  data() {
    return {
      songList: [
        {
          image: "song_1.jpg",
          audio: "song_1.mp3",
          song: "加载中",
          album: "加载中",
          singer: "加载中",
          duration: 0,
          isLike: false,
        },
      ],
      curIndex: 0,
      isPlaying: false,
      lastDuration: 0,
      playSongDuration: 0,
      frameId: null,
      isShowLikeTip: false,
      isShowPlayList: false
    };
  },
  created() {
    axios
      .get("/mock/data.json")
      .then((res) => {
        this.songList = res.data;
      })
      .catch((error) => {
        console.log("error: " + error);
      });
  },
  methods: {
    nextSong() {
      let index = this.curIndex + 1;
      this.changeSong(index != this.songList.length ? index : 0);
    },
    prevSong() {
      let index = this.curIndex - 1;
      this.changeSong(index != -1 ? index : this.songList.length - 1);
    },
    changeSong(index) {
      this.audio.pause();
      this.curIndex = index;
      this.audio.load();
      if (this.isPlaying) {
        this.audio.play();
        this.startProcessor();
      }
    },
    changePlayStatus() {
      if (this.isPlaying) {
        this.audio.pause();
        this.lastDuration = this.playSongDuration;
      } else {
        this.audio.play();
        this.startProcessor();
      }
      this.isPlaying = !this.isPlaying;
    },
    startProcessor() {
      let startTime = Date.now();
      let self = this;
      let render = function () {
        if (self.isPlaying) {
          let curPlayTime = (Date.now() - startTime) / 1000;
          console.log(curPlayTime);
          self.playSongDuration = curPlayTime + self.lastDuration;
          console.log("playSongDuration:" + self.playSongDuration);
          window.requestAnimationFrame(render);
        }
      };
      window.requestAnimationFrame(render);
    },
    formatTime(duration) {
      let time = Math.round(duration);
      var minute = Math.floor(time / 60);
      var second = time - minute * 60;
      if (minute < 10) {
        minute = "0" + minute;
      }
      if (second < 10) {
        second = "0" + second;
      }
      return minute + ":" + second;
    },
    changeLikeStatus() {
      this.currentSong.isLike = !this.currentSong.isLike;
      this.isShowLikeTip = true;
      let self = this;
      setTimeout(() => {
        self.isShowLikeTip = false;
      }, 1000);
    },
    showPlayList() {
        this.isShowPlayList = true;
        let self = this;
        setTimeout(() => {
            self.isShowPlayList = false;
        }, 20000);
    },
    closePlayList(){
        this.isShowPlayList = false;
    },
    selectSong(index){
        this.isPlaying = true;
        this.changeSong(index);
        this.closePlayList();

    }
  },
  computed: {
    currentSong() {
      return this.songList[this.curIndex];
    },
    allTime() {
      return this.formatTime(this.currentSong.duration);
    },
    imgSrc() {
      return "/source/" + this.currentSong.image;
    },
    audio() {
      let audio = new Audio("/source/" + this.currentSong.audio);
      return audio;
    },
    processorStyle() {
      let percent = (this.playSongDuration / this.currentSong.duration) * 100;
      return {
        transform: `translateX(${percent}%)`,
      };
    },
    curTimeString() {
      return this.formatTime(this.playSongDuration);
    },
  },
};
</script>


<style lang="less" scoped>
* {
  margin: 0;
  padding: 0;
}
ul li {
  list-style: none;
}
div,
h1,
h2,
h3,
h4,
h5,
h6,
p,
a {
  font: 14px/20px Helvetica, sans-serif, arial;
}
body {
  background-repeat: no-repeat;
  background-size: cover;
  background-position: 50%;
}
.content-wrap {
  height: 100vh;
  width: 100vw;
  padding-top: 60px;
  box-sizing: border-box;
  background-color: rgba(0, 0, 0, 0.3);
  // 歌曲图片样式
  .song-img {
    width: 50%;
    height: 0;
    padding-top: 50%;
    position: relative;
    margin: 0 auto 40px;
    .img-wrapper {
      position: absolute;
      left: 0;
      top: 0;
      right: 0;
      bottom: 0;
      img {
        width: 100%;
        height: 100%;
      }
    }
  }
  // 歌曲信息样式
  .song-info {
    height: 126px;
    width: 100%;
    text-align: center;
    .song-name {
      // 字数较多的时候打点展示
      text-overflow: ellipsis;
      overflow: hidden;
      white-space: nowrap;

      margin: 0 40px;
      font-size: 24px;
      line-height: 36px;
      color: rgba(255, 255, 255, 0.86);
      margin-bottom: 8px;
    }
    .singer-name,
    .album-name {
      font-size: 12px;
      line-height: 16px;
      color: rgba(255, 255, 255, 0.6);
      margin-bottom: 6px;
    }
  }
  // 进度条样式
  .play-area {
    display: flex;
    .cur-time,
    .all-time {
      width: 60px;
      height: 40px;
      color: #fff;
      text-align: center;
      line-height: 40px;
    }
    .pro-wrapper {
      flex: 1;
      position: relative;
      overflow: hidden;
      padding-left: 3px;
      padding-right: 3px;
      .pro-bottom,
      .pro-top {
        width: 100%;
        height: 1px;
        position: absolute;
        top: 20px;
      }
      .pro-bottom {
        background-color: rgba(255, 255, 255, 0.6);
      }
      .pro-top {
        transform: translateX(-100%);
        background-color: #fff;
        position: relative;
        .slide-point {
          height: 21px;
          width: 21px;
          position: absolute;
          top: -10px;
          left: -10px;

          // 水平垂直居中,flex应该设置在父元素上
          display: flex;
          // 水平居中
          align-items: center;
          // 垂直居中
          justify-content: center;

          &::after {
            height: 5px;
            width: 5px;
            content: "";
            background-color: #fff;
            border-radius: 50%;
          }
        }
      }
    }
  }
  // 控制区域样式
  .play-control {
    position: absolute;
    bottom: 0;
    height: 60px;
    width: 100%;
    background-color: rgba(0, 0, 0, 0.2);
    display: flex;
    .btn-wrap {
      flex: 1;
      background-repeat: no-repeat;
      background-size: 20px 20px;
      background-position: 50%;
    }
    .like-btn {
      background-image: url(/img/icon-like.png);
      &.liked {
        background-image: url(/img/icon-like-solid.png);
      }
    }
    .prev-btn {
      background-image: url(/img/icon-prev.png);
    }
    .play-btn {
      background-image: url(/img/icon-play.png);
      &.playing {
        background-image: url(/img/icon-pause.png);
      }
    }
    .next-btn {
      background-image: url(/img/icon-next.png);
    }
    .list-btn {
      background-image: url(/img/icon-list.png);
    }
  }
}

.play-list {
  position: fixed;
  bottom: 0;
  width: 100%;
  background-color: #fff;
  transform: translateY(100%);
  transition: transform 200ms ease-in;
  &.show {
    transform: translateY(0);
  }
  .line-head,
  .close-btn {
    height: 50px;
    line-height: 50px;
    text-align: center;
    font-size: 16px;
    color: rgba(0, 0, 0, 0.86);
  }
  .close-btn {
    border-top: 1px solid rgba(0, 0, 0, 0.1);
  }
  li {
    height: 20px;
    padding: 10px;
    border-top: 1px solid rgba(0, 0, 0, 0.1);
    h3 {
      color: rgba(0, 0, 0, 0.86);
      span {
        font-size: 12px;
        color: rgba(0, 0, 0, 0.6);
      }
    }
    &.playing {
      h3 {
        color: rgba(255, 0, 0, 0.86);
        span {
          color: rgba(255, 0, 0, 0.6);
        }
      }
    }
  }
}

#like-tip {
  width: 200px;
  height: 50px;
  position: absolute;
  left: 50%;
  top: 50%;
  margin-top: -25px;
  margin-left: -100px;
  text-align: center;
  line-height: 50px;
  font-weight: 900;
  color: #fff;
  background-color: rgba(150, 0, 0, 0.8);
}
</style>


