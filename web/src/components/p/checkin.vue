<template>
  <div v-loading="loading" class="checkin-main">
    <el-input :class="{ 'rad-border': (checked&&(!$store.state.name||type.name))}" placeholder="一定是真名呦~" v-model="$store.state.name" >
      <template slot="prepend"><div>姓名:</div></template>
    </el-input>
    <el-input :class="{ 'rad-border': (checked&&(!$store.state.number||type.number))}" type="number" placeholder="注意别填错啦~" v-model="$store.state.number" >
      <template slot="prepend">学号:</template>
    </el-input>
    <el-input :class="{ 'rad-border': (checked&&(!$store.state.school||type.school))}" placeholder="一定来自出色的学院" v-model="$store.state.school" >
      <template slot="prepend">学院:</template>
    </el-input>
    <el-input :class="{ 'rad-border': (checked&&(!$store.state.class||type.class))}" type="number" placeholder="不知道说什么了..." v-model="$store.state.class" >
      <template slot="prepend">班级:</template>
    </el-input>
    <el-input :class="{ 'rad-border': (checked&&(!$store.state.phone||type.phone))}" type="number" placeholder="联系不上是没有奖励的（任性" v-model="$store.state.phone" >
      <template slot="prepend">手机:</template>
    </el-input>
    <br>
    <el-input 
      v-for="(other,index) in others"
      :key="index"
      :class="{ 'rad-border': (checked&&(!$store.state.other[other.name]||type.other[other.name]))}" 
      :type="other.type" 
      :placeholder="other.placeholder" 
      v-model="$store.state.other[other.name]">
      <template slot="prepend">{{other.text}}:</template>
    </el-input>
    <div class="section">
      <h5 style="font-size: 0.4rem;">本次答题限时 {{$store.state.timeLimit / 60}} 分钟。</h5>
    </div>
    <el-button type="primary" :disabled="canUsePaper" @click="checkTable" round>让我们开始吧</el-button>
  </div>
</template>
<script>
export default {
  name: "checkin",
  data() {
    // console.log(BASE_URL);
    this.$store.state.routerPush = (string)=>{
      return this.$router.push(string);
    }
    return {
      checked: false,
      canUsePaper: true,
      type:{
        other:{}
      },
      others:[
      ],
      loading: true,
    }
  },
  mounted(){
    this.$http.post(
      '/api/getTitle', 
      JSON.stringify({id: Number(this.$route.params.id)})
    ).then(function(res) {
        if(res.data.code == 200){
          this.$store.state.title = res.data.data.title;
          this.$store.state.realtitle = res.data.data.title;
          document.title = res.data.data.title + ' | 学生会答题系统';
          this.$store.state.DocTitle = res.data.data.title + ' | 学生会答题系统';
          this.canUsePaper = false;
          this.others = res.data.data.other;
          this.$store.state.timeLimit = res.data.data.timeLimit;
          this.loading = false;
        }else{
          var errMsg = {
            400: "您已经挑战过一次了，请关注以后的活动",
            403: "活动尚未开始或已过期，别灰心,学生会好玩的活动有的是",
            404: "你可能到了外星球，这个活动我们没有",
            500: "%^&$%^%&^^$%^%$%"
          }
          this.$alert(errMsg[res.data.code]||res.data.message,"提示:",{
            confirmButtonText: '确定',
            callback: action => {
              this.$router.back();
            }
          })
        }
      }, function(err) {
        alert("服务器错误，请联系系统管理员！")
        console.log("err" + err);  
    })
  },
  methods:{
    checkTable: function(){
      let state = this.$store.state;
      this.checked = true;
      if(!(state.name && state.number && state.school && state.class && state.phone)){
        this.$notify.error({
          title: '好像哪里不太对',
          message: '亲的信息还没有填全面(●ˇ∀ˇ●)'
        });
        return false;
      }
      for(let i = 0; i < this.others.length; i++){
        if(!(state.other[this.others[i].name])){
          this.$notify.error({
            title: '好像哪里不太对',
            message: '亲的信息还没有填全面(●ˇ∀ˇ●)'
          });
          return false;
        }
      }
      let cont = 0;
      let re = /^1\d{10}$/;
      if (!re.test(state.phone)) {
        this.$notify.error({
          title: '好像哪里不太对',
          message: '手机号好像不太对劲的说(●ˇ∀ˇ●)'
        });
        this.type.phone=true;cont++;
      }
      re = /^\d{10}$/
      if (!re.test(this.$store.state.number)) {
        this.$notify.error({
          title: '好像哪里不太对',
          message: '学号好像不太对劲的说(●ˇ∀ˇ●)'
        });
        this.type.number=true;cont++;
      }else if(this.$store.state.number > 2019000000){
        this.$notify.error({
          title: '好像哪里不太对',
          message: '你是穿越过来的么？'
        });
        this.type.number=true;cont++;
      }
      if (!re.test(this.$store.state.class)) {
        this.$notify.error({
          title: '好像哪里不太对',
          message: '班号好像不太对劲的说(●ˇ∀ˇ●)'
        });
        this.type.class=true;cont++;
      }
      for(let i = 0; i < this.others.length;i++){
        if(this.others[i].re && !RegExp(re).test(this.$store.state.other[this.others[i].name])){
          this.$notify.error({
            title: '好像哪里不太对',
            message: this.others[i].text+'好像不太对劲的说(●ˇ∀ˇ●)'
          });
          this.type.other[this.others[i].name]=true;cont++;
        }
      }
      if(cont == 0){
        this.$store.state.power = 1;
        this.$router.push({name: 'survey'});
      }
    }
  }
}
</script>

<style>
.checkin-main {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.rad-border > * {
  /* color: red; */
  border: 1px solid #ce2121;
  transition: border-color 0.2s;
}
.el-input > * {
  transition: border-color 0.2s;
}
</style>

