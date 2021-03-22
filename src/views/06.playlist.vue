<template>
  <div class="playlist-container">
    <div class="top-wrap">
      <div class="img-wrap">
        <img :src="playlist.coverImgUrl" alt="" />
      </div>
      <div class="info-wrap">
        <p class="title">{{ playlist.name }}</p>
        <div class="author-wrap">
          <img class="avatar" :src="playlist.creator.avatarUrl" alt="" />
          <span class="name">{{ playlist.creator.nickname }}</span>
          <span class="time">{{ getYMDHMS(Number(playlist.createTime)) }} 创建</span>
        </div>
        <div class="play-wrap">
          <span class="iconfont icon-circle-play" @click="playMusic(tracks[0].id)"></span>
          <span class="text">播放全部</span>
        </div>
      </div>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲列表" name="1">
        <table class="el-table playlit-table">
          <thead>
            <th></th>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr class="el-table__row" v-for="(item, index) in tracks" :key="index">
              <td>{{ index + 1 }}</td>
              <td>
                <div class="img-wrap">
                  <img :src="item.al.picUrl" alt="" />
                  <span class="iconfont icon-play" @click="playMusic(item.id)"></span>
                </div>
              </td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{ item.al.name }}</span>
                    <span v-if="item.mv != 0" class="iconfont icon-mv" @click="playMv(item.mv)"></span>
                  </div>
                  <span>{{ item.alia[0] }}</span>
                </div>
              </td>
              <td>{{ item.ar[0].name }}</td>
              <td>{{ item.name }}</td>
              <td>{{ getDuration(item.dt) }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="评论" name="2">
        <!-- 精彩评论 -->
        <div class="comment-wrap" v-if="hotComment.length != 0">
          <p class="title">
            精彩评论<span class="number"></span>
          </p>
          <div class="comments-wrap">
            <div class="item" v-for="(item, index) in hotComment" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <!-- 评论 -->
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{ item.user.nickname }}:</span>
                  <span class="comment">{{ item.content }}</span>
                </div>
                <!-- 评论的回复 -->
                <div class="re-content" v-if="item.beReplied.length != 0">
                  <span class="name"
                    >{{ item.beReplied[0].user.nickname }}：</span
                  >
                  <span class="comment">{{ item.beReplied[0].content }}</span>
                </div>
                <div class="date">{{ getYMDHMS(Number(item.time)) }}</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 最新评论 -->
        <div class="comment-wrap">
          <p class="title">最新评论<span class="number">({{ total }})</span></p>
          <div class="comments-wrap">
            <div class="item" v-for="(item, index) in comments" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{ item.user.nickname }}：</span>
                  <span class="comment">{{ item.content }}</span>
                </div>
                <div class="re-content" v-if="item.beReplied.length != 0">
                  <span class="name">{{ item.beReplied[0].user.nickname }}：</span>
                  <span class="comment">{{ item.beReplied[0].content }}</span>
                </div>
                <div class="date">{{ getYMDHMS(Number(item.time)) }}</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 分页器 -->
        <el-pagination
          @current-change="handleCurrentChange"
          background
          layout="prev, pager, next"
          :total="total"
          :current-page="page"
          :page-size="10"
        >
        </el-pagination>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "playlist",
  data() {
    return {
      activeIndex: "1",
      // 总条数
      total: 0,
      // 页码
      page: 1,
      // 歌单详情数据
      playlist: {},
      // 前10首歌曲列表
      tracks: [], 
      // 热门评论
      hotComment: [],
      // 热门评论的个数
      hotCount: 0,
      // 普通评论
      comments: [],
    };
  },
  methods: {
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`);
      this.page = val;
      axios({
      url:"https://autumnfish.cn/comment/playlist",
      method:"get",
      params:{
        id:this.$route.query.q,
        limit:10,
        // 根据页码计算
        offset:(this.page-1)*10
      }
    }).then(res=>{
      // console.log(res)
      // 总个数
      this.total = res.data.total
      // 评论数据
      this.comments = res.data.comments
    })
    },
    //将时间戳转换为年月日时分秒格式
    getYMDHMS(timestamp){
      let time = new Date(timestamp)
      let year = time.getFullYear()
      let month = time.getMonth() + 1
      let date = time.getDate()
      let hours = time.getHours()
      let minute = time.getMinutes()
      let second = time.getSeconds()

      if (month < 10) { month = '0' + month }
      if (date < 10) { date = '0' + date }
      if (hours < 10) { hours = '0' + hours }
      if (minute < 10) { minute = '0' + minute }
      if (second < 10) { second = '0' + second }
      return year + '-' + month + '-' + date + ' ' + hours + ':' + minute + ':' + second
    },
    //将歌曲的时长转换为分秒格式
    getDuration(duration){
      let minutes = parseInt(duration / 1000 / 60);
      if (minutes < 10) {
          minutes = '0' + minutes;
      }
      let seconds = parseInt(duration / 1000 % 60);
      if (seconds < 10) {
          seconds = '0' + seconds;
      }
      return (`${minutes}:${seconds}`);
    },
    playMusic: function(id){
      axios({
        url:"https://autumnfish.cn/song/url",
        method:"get",
        params:{
          id: id
        }
      }).then(res => {
        // 将播放链接传递给父组件
        this.$parent.musicUrl = res.data.data[0].url;
        // console.log(res);
      })
    },
    playMv: function(mvid){
      this.$router.push('/mv?q=' + mvid);
    },
  },
  created() {
    // 获取歌单详情
    axios({
      url: "https://autumnfish.cn/playlist/detail",
      method: "get",
      params: {
        id: this.$route.query.q,
      },
    }).then((res) => {
      // console.log(res.data.playlist.trackIds)
      this.playlist = res.data.playlist;
      this.tracks = res.data.playlist.tracks;
    });

    // 获取 评论
    axios({
      url: "https://autumnfish.cn/comment/hot",
      method: "get",
      params: {
        id: this.$route.query.q,
        // 传递类型
        type: 2,
      },
    }).then((res) => {
      // console.log(res);
      this.hotComment = res.data.hotComments;
      // 保存个数
      this.hotCount = res.data.total;
    });

    // 获取 最新评论
    axios({
      url: "https://autumnfish.cn/comment/playlist",
      method: "get",
      params: {
        id: this.$route.query.q,
        limit: 10,
        offset: 0,
      },
    }).then((res) => {
      // console.log(res)
      // 总个数
      this.total = res.data.total;
      // 评论数据
      this.comments = res.data.comments;
    });
  },
};
</script>

<style >
</style>
