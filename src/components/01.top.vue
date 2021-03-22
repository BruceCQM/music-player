<template>
  <div class="top-container">
    <div class="left-box">
      <div class="icon-wrapper" style="margin-right:30px">
        <img src="../../public/music.png" alt="音乐图标" style="width:30px;vertical-align:middle">
        <span style="display:inline-block;margin-left:15px;margin-top:17px">天籁音乐</span>
        <!-- <span class="iconfont icon-home"></span>
        <span class="iconfont icon-sami-select" ></span>
        <span class="iconfont icon-full-screen" ></span> -->
      </div>
      <div class="history-wrapper">
        <span class="iconfont icon-arrow-lift" @click="back"></span>
        <span class="iconfont icon-arrow-right" @click="forward"></span>
      </div>
    </div>
    <div class="right-box">
      <div class="el-input el-input--small el-input--prefix">
        <input
          type="text"
          autocomplete="off"
          placeholder="搜索"
          class="el-input__inner"
          v-model="inputValue"
          @keyup.enter="toResult"
        />
        <span class="el-input__prefix">
          <i class="el-input__icon el-icon-search"></i>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'top',
    data() {
      return {
        query: '',
        inputValue: ''
      }
    },
    methods:{
      toResult(){
        if(this.inputValue == ''){
          this.$message({
          message: '请输入内容',
          type: 'warning'
        });
        }else{
          this.$router.push('/result?q=' + this.inputValue)
        }
      },
      forward(){
        history.forward();
      },
      back(){
        history.back();
      }
    }
  }
</script>

<style scoped></style>
