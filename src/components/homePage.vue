<template>
  <el-container class="container">
    <el-header class="header">
        <el-menu mode="horizontal"
                 background-color="#545c64"
                 text-color="#fff"
                 active-text-color="#ffd04b">
          <el-col :span="4" :xs="8" :offset="1" id="head-name">
            <p>在线答题系统</p>
          </el-col>
          <el-col :span="4" :xs="4" style="float: right;line-height: 60px">
            <el-popover trigger="click" placement="bottom" v-model="visible">
              <ul id="toLoad" @click="visible = false"
                  :element-loading-text="listLoadingText"
                  v-loading="listLoading">
                <li v-for="(item, index) in fileList" :key="index"
                    :class="{active: item === curListItem}"
                    :title="item"
                    @click="switchQuestion(item)">{{item}}</li>
              </ul>
              <i class="el-icon-more btn-more" slot="reference"></i>
            </el-popover>
          </el-col>
        </el-menu>
    </el-header>
    <question :questions="questions" :name="curListItem"
              ref="question"
              :element-loading-text="loadingText"
              v-loading="loading">
    </question>
  </el-container>
</template>

<script>
import Question from '../components/Question'
import type from '../store/mutation-types.js'
export default {
  name: 'homePage',
  data () {
    return {
      questions: [],
      // 保存题目列表
      fileList: [],
      curListItem: '',
      loading: true,
      loadingText: '点击右上方按钮选择题库',
      listLoading: true,
      listLoadingText: '正在加载, 请稍等...',
      visible: false
    }
  },
  methods: {
    // 获取题目数据
    switchQuestion (filename) {
      let _this = this
      // 如果没有变化则退出
      if (filename === _this.curListItem) {
        return
      }
      // 如果是首次则直接加载
      if (_this.curListItem === '') {
        _this.loadingText = ''
        return _this.getData(filename)
      }
      _this.$msgbox({
        title: '提示',
        type: 'warning',
        message: _this.$createElement(
          'p',
          {style: 'fontSize: 18px'},
          '确认切换题库？记录将会保存。'
        ),
        showCancelButton: true,
        confirmButtonText: '确认切换',
        cancelButtonText: '继续做题'
      }).then(() => {
        _this.loading = true
        // 子组件触发状态提交，并清除当前状态
        _this.$refs.question.commitState()
        _this.getData(filename)
      }).catch(() => {})
    },
    getData (filename) {
      let _this = this
      _this.$store.dispatch(type.RECEIVE_QUESTION, {
        filename: filename,
        callback: function (data) {
          _this.questions = data
          _this.curListItem = filename
          _this.loading = false
          _this.$message({
            message: '加载成功',
            type: 'success',
            duration: 1200,
            center: true
          })
        }
      })
    }
  },
  components: {
    Question
  },
  mounted () {
    this.$store.dispatch(type.RECEIVE_FILES, fileList => {
      this.fileList = fileList
      this.listLoading = false
    })
  }
}
</script>

<style scoped lang="stylus">
  scrollBar()
    overflow auto
    &::-webkit-scrollbar-button
      display none
    &::-webkit-scrollbar-thumb
      border-radius 5px
      background-color rgba(168,168,168, 0.5)
    &::-webkit-scrollbar-thumb:hover
      background-color rgba(168,168,168, 0.8)
    &::-webkit-scrollbar
      height 7px
      width 7px
      border-radius 5px
      background-color rgba(231,234,237, 0.5)
  .active
    background-color lightblue !important
  ul#toLoad
    font-size 18px
    list-style none
    height 300px
    overflow auto
    background-color white
    & li
      margin 5px 0
      padding 0 5px
      border-radius 5px
      cursor pointer
      height 40px
      line-height 40px
      /* 超出部分省略号代替 */
      text-overflow ellipsis
      white-space nowrap
      overflow hidden
      /* 防止点击时选中文本,影响体验*/
      -ms-user-select none
      -moz-user-select none
      -webkit-user-select none
      &:hover
        background-color rgb(245,245,245)
  .container
    min-width 100%
  .header
    padding 0
  #head-name
    font-size 20px
    color white
    line-height 60px
  .btn-more
    transform scale(2.5)
    color white
</style>
