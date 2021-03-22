<template>
  <div class="songs-container">
    <div class="tab-bar">
      <span class="item" :class="{active:tag==0}" @click="tag=0">全部</span>
      <span class="item" :class="{active:tag==7}" @click="tag=7">华语</span>
      <span class="item" :class="{active:tag==96}" @click="tag=96">欧美</span>
      <span class="item" :class="{active:tag==8}" @click="tag=8">日本</span>
      <span class="item" :class="{active:tag==16}" @click="tag=16">韩国</span>
    </div>
    <!-- 底部的table -->
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
        <tr class="el-table__row" v-for="(item, index) in lists" :key="index">
          <td>{{ index + 1 }}</td>
          <td>
            <div class="img-wrap">
              <img :src="item.album.picUrl" alt="" />
              <span class="iconfont icon-play" @click="playMusic(item.id)"></span>
            </div>
          </td>
          <td>
            <div class="song-wrap">
              <div class="name-wrap">
                <span>{{ item.name }}</span>
                <span v-if="item.mvid != 0" class="iconfont icon-mv" @click="toMv(item.mvid)"></span>
              </div>
              <span>{{ item.album.company }}</span>
            </div>
          </td>
          <td>{{ item.album.artists[0].name }}</td>
          <td>{{ item.album.name }}</td>
          <td>{{ item.duration }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'songs',
  watch:{
    tag:function(){
      this.getList();
    }
  },
  data() {
    return {
        lists:[],
        tag:0
    };
  },
  created:function(){
    this.getList()
  },
  methods:{
    getList(){
      axios({
      url:" https://autumnfish.cn/top/song",
      method:"get",
      params:{
        type: this.tag
      }
    }).then(res=>{
      // console.log(res);
      this.lists = res.data.data;
      for(let i = 0;i < this.lists.length;i++){
        let duration = this.lists[i].duration;
        let minutes = parseInt(duration / 1000 / 60);
        if(minutes < 10){
          minutes = '0' + minutes;
        }
        let seconds = parseInt(duration / 1000 % 60);
        if(seconds < 10){
          seconds = '0' + seconds;
        }
        this.lists[i].duration = `${minutes}:${seconds}`;
        // console.log(`${minutes}:${seconds}`);
      }
    })
    },
    playMusic(id){
      axios({
        url:"https://autumnfish.cn/song/url",
        method:"get",
        params:{
          id
        }
      }).then(res=>{
        // console.log(res.data.data[0].url);
        let url = res.data.data[0].url;
        this.$parent.musicUrl = url;
      })
    },
    toMv(mvid){
      this.$router.push('/mv?q=' + mvid);
    }
  }
};
</script>

<style >

</style>
