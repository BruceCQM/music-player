<template>
  <div class="mv-container">
    <!-- MV详情 -->
    <div class="mv-wrap">
      <h3 class="title">mv详情</h3>
      <!-- mv -->
      <div class="video-wrap">
        <video controls :src="url"></video>
      </div>
      <!-- mv信息 -->
      <div class="info-wrap">
        <div class="singer-info">
          <div class="avatar-wrap">
            <img :src="icon" alt="" />
          </div>
          <span class="name">{{ mvInfo.artistName }}</span>
        </div>
        <div class="mv-info">
          <h2 class="title">{{ mvInfo.name }}</h2>
          <span class="date">发布：{{ mvInfo.publishTime }}</span>
          <span class="number">播放：{{ mvInfo.playCount }}次</span>
          <p class="desc">
            {{ mvInfo.desc }}
          </p>
        </div>
      </div>
      <!-- 精彩评论 -->
      <div class="comment-wrap" v-if="hotComments.length != 0">
        <p class="title">精彩评论<span class="number"></span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item, index) in hotComments" v-bind:key="index">
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
        :limit="limit"
      >
      </el-pagination>
    </div>
    <!-- 相关MV -->
    <div class="mv-recommend">
      <h3 class="title">相关推荐</h3>
      <div class="mvs">
        <div class="items">
          <div class="item" v-for="(item, index) in simiMvs" :key="index">
            <!-- 封面区域 -->
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play" @click="playMV(item.id)"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{ item.playCount }}</div>
              </div>
              <span class="time">{{ item.duration }}</span>
            </div>
            <!-- 信息区域 -->
            <div class="info-wrap">
              <div class="name">{{ item.name }}</div>
              <div class="singer">{{ item.artistName }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "mv",
  data() {
    return {
      // 总条数
      total: 20,
      // 页码
      page: 1,
      // 页容量
      limit: 10,
      // mv地址
      url: "",
      // 相关mv
      simiMvs: [],
      // mv的信息
      mvInfo: {},
      // 头像
      icon: "",
      // 热门评论
      hotComments: [],
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
      // 获取评论数据
      axios({
        url: "https://autumnfish.cn/comment/mv",
        method: "get",
        params: {
          id: this.$route.query.q,
          limit: 10,
          offset: (this.page - 1) * 10,
        },
      }).then((res) => {
        // console.log(res);
        // this.hotComments = res.data.hotComments;
        this.comments = res.data.comments;
        // this.total = res.data.total;
      });
    },
    playMV(id) {
      this.$router.push("/mv?q=" + id);
      this.init();
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
    // MV详情页初始化操作
    init() {
      // 获取mv播放地址
      axios({
        url: "https://autumnfish.cn/mv/url",
        method: "get",
        params: {
          // 获取url中的 id
          id: this.$route.query.q,
        },
      }).then((res) => {
        console.log(res.data.data.url)
        console.log(111);
        this.url = res.data.data.url;
      });

      // 获取相关的mv
      axios({
        url: "https://autumnfish.cn/simi/mv",
        method: "get",
        params: {
          mvid: this.$route.query.q,
        },
      }).then((res) => {
        // console.log(res)
        // 保存相关MV
        this.simiMvs = res.data.mvs;
        for (let i = 0; i < this.simiMvs.length; i++) {
          let time = this.simiMvs[i].duration;
          let minutes = parseInt(time / 1000 / 60);
          let seconds = parseInt((time / 1000) % 60);
          if (minutes < 10) {
            minutes = "0" + minutes;
          }
          if (seconds < 10) {
            seconds = "0" + seconds;
          }
          this.simiMvs[i].duration = minutes + ":" + seconds;

          // 播放次数
          if (this.simiMvs[i].playCount > 100000) {
            this.simiMvs[i].playCount =
              parseInt(this.simiMvs[i].playCount / 10000) + "万";
            // console.log(this.simiMvs[i].playCount / 10000);
          }
        }
      });

      // 获取 mv的信息
      axios({
        url: "https://autumnfish.cn/mv/detail",
        method: "get",
        params: {
          mvid: this.$route.query.q,
        },
      }).then((res) => {
        // console.log(res)
        // mv的信息
        this.mvInfo = res.data.data;

        // 获取 歌手信息
        axios({
          url: "https://autumnfish.cn/artists",
          method: "get",
          params: {
            id: this.mvInfo.artists[0].id,
          },
        }).then((res) => {
          // console.log(res);
          // 歌手的封面信息
          this.icon = res.data.artist.picUrl;
        });
      });

      // 获取评论数据
      axios({
        url: "https://autumnfish.cn/comment/mv",
        method: "get",
        params: {
          id: this.$route.query.q,
          limit: 10,
          offset: 0,
        },
      }).then((res) => {
        console.log(res);
        this.hotComments = res.data.hotComments;
        this.comments = res.data.comments;
        this.total = res.data.total;
      });
    },
  },
  created() {
    this.init();
  },
};
</script>

<style></style>
