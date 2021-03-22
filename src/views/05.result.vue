<template>
  <div class="result-container">
    <div class="title-wrap">
      <h2 class="title">{{ this.$route.query.q }}</h2>
      <span class="sub-title">找到 {{ count }} 个结果</span>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲" name="songs">
        <table class="el-table">
          <thead>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr
              class="el-table__row"
              v-for="(item, index) in songsList"
              :key="index"
              @dblclick="playMusic(item.id)"
            >
              <td>{{ (index + 1) + (page - 1)*10 }}</td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{ item.name }}</span>
                    <span
                      class="iconfont icon-mv"
                      v-if="item.mvid != 0"
                      @click="toMV(item.mvid)"
                    ></span>
                  </div>
                  <span v-if="item.alias.length != 0">{{ item.alias[0] }}</span>
                </div>
              </td>
              <td>{{ item.artists[0].name }}</td>
              <td>{{ item.album.name }}</td>
              <td>{{ item.duration }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="歌单" name="lists">
        <div class="items">
          <div
            class="item"
            v-for="(item, index) in playlists"
            :key="index"
            @click="toPlaylists(item.id)"
          >
            <div class="img-wrap">
              <div class="num-wrap">
                播放量:
                <span class="num">{{ item.playCount }}</span>
              </div>
              <img :src="item.coverImgUrl" alt="" />
              <span class="iconfont icon-play"></span>
            </div>
            <p class="name">{{ item.name }}</p>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div
            class="item"
            v-for="(item, index) in mv"
            :key="index"
            @click="toMV(item.id)"
          >
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{ item.playCount }}</div>
              </div>
              <span class="time">{{ item.duration }}</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{ item.name }}</div>
              <div class="singer">{{ item.artistName }}</div>
            </div>
          </div>
        </div>
      </el-tab-pane>
      <!-- 分页器 -->
      <!-- total是总条数，current-page是当前页，page-size是每页条数 -->
      <!-- @current-change是页码改变事件 -->
      <el-pagination
        @current-change="handleCurrentChange"
        background
        layout="prev, pager, next"
        :total="count"
        :current-page="page"
        :page-size="10"
      >
      </el-pagination>
    </el-tabs>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "result",
  data() {
    return {
      activeIndex: "songs",
      page: 1,
      songsList: [],
      //保存歌单
      playlists: [],
      mv: [],
      //搜索结果条数
      count: 0,
    };
  },
  watch: {
    activeIndex: function () {
      this.page = 1;
      switch (this.activeIndex) {
        case "songs":
          this.getSongList();
          break;
        case "lists":
          this.getPlayList();
          break;
        case "mv":
          this.getMv();
          break;
        default:
          break;
      } 
    },
  },
  created() {
    this.getSongList();
  },
  methods: {
    playMusic(id) {
      axios({
        url: "https://autumnfish.cn/song/url",
        method: "get",
        params: {
          id: id,
        },
      }).then((res) => {
        // console.log(res.data.data[0].url);
        let url = res.data.data[0].url;
        //将歌曲链接传递给父组件
        this.$parent.musicUrl = url;
        // console.log(this.$parent.musicUrl);
      });
    },
    toPlaylists(id) {
      this.$router.push("/playlist?q=" + id);
    },
    toMV(id) {
      this.$router.push("/mv?q=" + id);
    },
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`);
      this.page = val;
      switch (this.activeIndex) {
        case "songs":
          this.getSongList();
          break;
        case "lists":
          this.getPlayList();
          break;
        case "mv":
          this.getMv();
          break;
        default:
          break;
      }
    },
    // 获取歌曲列表
    getSongList() {
      axios({
        url: "https://autumnfish.cn/search",
        method: "get",
        params: {
          keywords: this.$route.query.q,
          limit: 10,
          type: 1,
          offset: (this.page - 1) * 10
        },
      }).then((res) => {
        this.songsList = res.data.result.songs;
        this.count = res.data.result.songCount;
        for (let i = 0; i < this.songsList.length; i++) {
          let time = this.songsList[i].duration;
          let minutes = parseInt(time / 1000 / 60);
          let seconds = parseInt((time / 1000) % 60);
          if (minutes < 10) {
            minutes = "0" + minutes;
          }
          if (seconds < 10) {
            seconds = "0" + seconds;
          }
          this.songsList[i].duration = minutes + ":" + seconds;
        }
        // console.log(this.songsList);
      });
    },
    // 获取歌单列表
    getPlayList() {
      axios({
        url: "https://autumnfish.cn/search",
        method: "get",
        params: {
          keywords: this.$route.query.q,
          type: 1000, // type:type,
          // 获取的数据量
          limit: 10, 
          offset: (this.page - 1) * 10
        },
      }).then((res) => {
        this.playlists = res.data.result.playlists;
        // 总数
        this.count = res.data.result.playlistCount;
        // 处理 播放次数
        for (let i = 0; i < this.playlists.length; i++) {
          if (this.playlists[i].playCount > 100000) {
            this.playlists[i].playCount =
              parseInt(this.playlists[i].playCount / 10000) + "万";
          }
        }
      });
    },
    // 获取MV列表
    getMv() {
      axios({
        url: "https://autumnfish.cn/search",
        method: "get",
        params: {
          keywords: this.$route.query.q,
          type: 1004, // type:type,
          // 获取的数据量
          limit: 8,
          offset: (this.page - 1) * 8
        },
      }).then((res) => {
        this.mv = res.data.result.mvs;
        // 总数
        this.count = res.data.result.mvCount;
        // 处理数据
        for (let i = 0; i < this.mv.length; i++) {
          // 时间
          let min = parseInt(this.mv[i].duration / 1000 / 60);
          let sec = parseInt((this.mv[i].duration / 1000) % 60);
          if (min < 10) {
            min = "0" + min;
          }
          if (sec < 10) {
            sec = "0" + sec;
          }
          this.mv[i].duration = min + ":" + sec;

          // 播放次数
          if (this.mv[i].playCount > 100000) {
            this.mv[i].playCount =
              parseInt(this.mv[i].playCount / 10000) + "万";
          }
        }
      });
    },
  },
};
</script>

<style >
</style>
