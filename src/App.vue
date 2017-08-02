<template>
  <div id="app">
    <view-box>
      <x-header slot="header" class="header" :left-options="{showBack:false}">
        <div slot="left">直播 </div>
        <div>网易新闻</div>
        <div slot="right"> 搜索</div>
      </x-header>
      <sc :lock-y="true">
        <div class="tab">
          <tab>
            <tab-item selected>推荐</tab-item>
            <tab-item>要文</tab-item>
            <tab-item>游戏</tab-item>
            <tab-item>科技</tab-item>
            <tab-item>娱乐</tab-item>
            <tab-item>体育</tab-item>
          </tab>
        </div>
      </sc>

      <scroller class="my-scroll"
                :on-refresh="refresh"
                :on-infinite="infinite"
                ref="scro">

        <swiper :list="swipeList" :auto="true" :show-dots="true" :loop="true" v-model="swiperIndex">

        </swiper>
        <div class="news-marquee">
          <marquee>
            <marquee-item class="roll_item" v-for="item in rollListData"><a :href="item.link">{{item.title}}</a>
            </marquee-item>
          </marquee>
        </div>
        <panel :list="datalist">

        </panel>
        <panel :list="moreDataList">

        </panel>

      </scroller>

      <tabbar slot="bottom">
        <tabbar-item>
          <img src="./assets/img/home.png" slot="icon" alt="">
          <span slot="label">首页</span>
        </tabbar-item>
        <tabbar-item>
          <img src="./assets/img/good.png" slot="icon" alt="">
          <span slot="label">推荐</span>
        </tabbar-item>
        <tabbar-item>
          <img src="./assets/img/mine.png" slot="icon" alt="">
          <span slot="label">我的</span>
        </tabbar-item>

      </tabbar>
    </view-box>
  </div>
</template>

<script>
  import {
    ViewBox,
    XHeader,
    Tabbar,
    TabbarItem,
    Tab,
    TabItem,
    Scroller as sc,
    Swiper,
    Marquee,
    MarqueeItem,
    Panel
  } from 'vux'


  var refreshKey = ['A', 'B01', 'B02', 'B03', 'B04', 'B05', 'B06', 'B07', 'B08', 'B09', 'B010'];
  var key = 0
  var start = 0
  var end = start + 9
  var keyValue = 'A'
  var isFinished = false

  function getKey() {
    key++
    if (key === refreshKey.length) {
      key = 0
    }
    keyValue = refreshKey[key]

  }

  export default {
    name: 'app',
    components: {
      ViewBox,
      XHeader,
      Tabbar,
      TabbarItem,
      Tab,
      TabItem,
      sc,
      Swiper,
      Marquee,
      MarqueeItem,
      Panel
    }, created() {
//      console.log(this.$refs.scro)
      this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html').then(data => {
//        console.log(data)
        /*首屏新闻列表*/
        this.swipeList = data.focus.filter(item => {
          return item.addata === null && item.picInfo[0];
        }).map(item => {
          return {
            url: item.link,
            img: item.picInfo[0].url,
            title: item.title
          }
        })
        /*列表新闻*/
        this.datalist = data.list.filter(item => {
          return item.addata === null && item.picInfo[0]
        }).map(item => {
          return {
            src: item.picInfo[0].url,
            title: item.title,
            desc: item.digest,
            url: item.link
          }
        })
        /*滚动新闻*/
        this.rollListData = data.live.filter(item => {
          return item.addata === null && item.picInfo[0]
        }).map(item => {
          return {
            title: item.title,
            link: item.link
          }
        })
        isFinished = true
      })

    },
    data() {

      return {
        swipeList: [],
        swiperIndex: 0,
        datalist: [],
        rollListData: [],
        moreDataList: []
      }
    },
    methods: {
      refresh() {
        /*获取不同类型*/
        getKey()
//        console.log('shang')
        /*下拉数据加载*/
        this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html', {
          miss: '00',
          refresh: keyValue
        }).then(data => {
//          console.log(data)
          setTimeout(() => {
            this.datalist = data.list.filter(item => {
              return item.addata === null && item.picInfo[0]
            }).map(item => {
              return {
                src: item.picInfo[0].url,
                title: item.title,
                desc: item.digest,
                url: item.link
              }
            })
          }, 1000)
          this.$refs.scro.finishPullToRefresh()
          this.$vux.toast.text(`当前一下拉共刷新了${this.datalist.length}`, "top")
        })

      },
      infinite() {
        if (!isFinished) {
          this.$refs.scro.finishInfinite()
          return;
        }
        console.log('shang')

        this.$jsonp(`http://3g.163.com/touch/jsonp/sy/recommend/${start}-${end}.html`, {
          miss: '00',
          refresh: keyValue
        }).then(data => {

            setTimeout(() => {
              this.moreDataList = data.list.filter(item => {
                return item.addata === null && item.picInfo[0]
              }).map(item => {
                return {
                  src: item.picInfo[0].url,
                  title: item.title,
                  desc: item.digest,
                  url: item.link
                }
              })
              start += 10;
              end = start + 9
              console.log(data)
              this.$refs.scro.finishInfinite()
            }, 1000)
          }
        )
      }
    }
  }
</script>

<style lang="less">
  @import "~vux/src/styles/reset.less";

  html, body {
    margin: 0;
    width: 100%;
    height: 100%;
    overflow-x: hidden;
  }

  #app {
    height: 100%;
    .tab {
      margin-top: 46px;
      width: 1200px;

    }
    .header {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      z-index: 9;
    }
    .news-marquee {
      margin: 10px;
    }
    .roll_item a {
      color: #000;
    }
    .weui-media-box__hd, .weui-media-box__hd img {

      width: 98px;
      height: 60px;
    }
    .my-scroll {
      margin-top: 90px;
    }
    .loading-layer {
      padding-bottom: 90px;
    }
  }
</style>
