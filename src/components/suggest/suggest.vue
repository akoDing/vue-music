<template>
  <Scroll ref="suggest"
          class="suggest"
          :data="result"
          :pullup="pullup"
          :beforeScroll="beforeScroll"
          @scrollToEnd="searchMore"
          @beforeScroll="listScroll"
  >
    <ul class="suggest-list">
      <li @click="selectItem(item)" class="suggest-item" v-for="item in result" :key="item.index">
        <div class="icon">
          <i :class="getIconCls(item)"></i>
        </div>
        <div class="name">
          <p class="text" v-html="getDisplayName(item)"></p>
        </div>
      </li>
      <Loading v-show="hasMore" title=""></Loading>
    </ul>
    <div v-show="!hasMore && !result.length" class="no-result-wrapper">
      <NoResult title="抱歉，暂无搜索结果"></NoResult>
    </div>
  </Scroll>
</template>

<script>
import Scroll from 'base/scroll/scroll'
import Loading from 'base/loading/loading'
import NoResult from 'base/no-result/no-result'
import {search} from 'api/search'
import {ERR_OK} from 'api/config'
// import {filterSinger} from 'common/js/song'
import {createSong} from 'common/js/song'
// import {getSong} from 'api/song'
import {mapMutations, mapActions} from 'vuex'
import Singer from 'common/js/singer'

const TYPE_SINGER = 'singer'
const perpage = 20

export default {
  props: {
    query: {
      type: String,
      default: ''
    },
    showSinger: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      page: 1,
      result: [],
      result1: [],
      pullup: true,
      beforeScroll: true,
      hasMore: false
    }
  },
  methods: {
    search() {
      // 重置操作
      this.page = 1
      this.hasMore = true
      this.$refs.suggest.scrollTo(0, 0)
      /* new Promise((resolve, reject) => {
        search(this.query, this.page, this.showSinger, perpage).then((res) => {
          if (res.code === ERR_OK) {
            console.log(res.data)
            this.result = this._genResult(res.data)
            this.result = this._normalizeSongs(res.data.song.list)
            this.result = res.data
            console.log(this.result)
            console.log(res.data)
            console.log(3)
            this._checkMore(res.data)
          }
          resolve(res)
        })
      }).then((res) => {
        console.log(res.data)
        this.result = this._genResult(res.data)
      }) */
      search(this.query, this.page, this.showSinger, perpage).then((res) => {
        if (res.code === ERR_OK) {
          // console.log(res.data)
          this.result = this._genResult(res.data)
          // this.result = this._genResult(res.data)
          // console.log(this.result)
          this._checkMore(res.data)
          // this.hasMore = false
        }
      })
    },
    searchMore() {
      if (!this.hasMore) {
        return
      }
      this.page++
      search(this.query, this.page, this.showSinger, perpage).then((res) => {
        if (res.code === ERR_OK) {
          this.result = this.result.concat(this._genResult(res.data))
          // console.log(res.data)
          this._checkMore(res.data)
        }
      })
    },
    refresh() {
      // 传递给search组件的refresh方法
      this.$refs.suggest.refresh()
    },
    getIconCls(item) {
      if (item.type === TYPE_SINGER) {
        return 'icon-mine'
      } else {
        return 'icon-music'
      }
    },
    getDisplayName(item) {
      if (item.type === TYPE_SINGER) {
        return item.singername
      } else {
        return `${item.name}-${item.singer}`
      }
    },
    selectItem(item) {
      if (item.type === TYPE_SINGER) {
        const singer = new Singer({
          id: item.singermid,
          name: item.singername
        })
        this.$router.push({
          path: `/search/${singer.id}`
        })
        this.setSinger(singer)
      } else {
        this.insertSong(item)
      }
      this.$emit('select')
    },
    listScroll() {
      this.$emit('listScroll')
    },
    _checkMore(data) {
      const song = data.song
      // console.log(song.list.length)
      if (!song.list.length || (song.curnum + song.curpage * perpage) >= song.totalnum) {
        this.hasMore = false
      }
    },
    _genResult(data) {
      let ret = []
      if (data.zhida && data.zhida.singerid && data.song.curpage === 1) {
        ret.push({...data.zhida, ...{type: TYPE_SINGER}})
      }
      if (data.song) {
        // alert(1)
        // console.log('**' + data.song.list)
        ret = ret.concat(this._normalizeSongs(data.song.list))
        // console.log(2)
        // console.log(data.song.list)
        // console.log('concat1' + this._normalizeSongs(data.song.list))
      }
      // console.log(ret)
      // console.log(typeof (ret))
      return ret
    },
    _normalizeSongs(list) {
      // console.log('***' + list)
      let ret = []
      list.forEach((musicData) => {
        if (musicData.songid && musicData.albumid) {
          // 这里还需要修改增加vkey
          // console.log(createSong(musicData))
          ret.push(createSong(musicData))
          /* new Promise((resolve, reject) => {
            getSong(musicData.songmid).then((res) => {
            // console.log(5555)
              if (res.code === ERR_OK) {
                const svkey = res.data.items
                const songVkey = svkey[0].vkey
                const newSong = createSong(musicData, songVkey)
                console.log(newSong)
                ret.push(newSong)
                console.log(222)
              }
              resolve(1)
            })
          }).then((res) => {
            console.log(res)
          }) */
          // console.log('****' + musicData.songid)
          /* getSong(musicData.songmid).then((res) => {
            if (res.code === ERR_OK) {
              const svkey = res.data.items
              const songVkey = svkey[0].vkey
              const newSong = createSong(musicData, songVkey)
              console.log(newSong)
              ret.push(newSong)
            }
          }) */
        }
      })
      // console.log('*****' + ret)
      // console.log(ret)
      return ret
    },
    ...mapMutations({
      setSinger: 'SET_SINGER'
    }),
    ...mapActions([
      'insertSong'
    ])
  },
  watch: {
    query() {
      this.search()
    }
  },
  components: {
    Scroll,
    Loading,
    NoResult
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  @import "~common/stylus/mixin"

  .suggest
    height: 100%
    overflow: hidden
    .suggest-list
      padding: 0 30px
      .suggest-item
        display: flex
        align-items: center
        padding-bottom: 20px
      .icon
        flex: 0 0 30px
        width: 30px
        [class^="icon-"]
          font-size: 14px
          color: $color-text-d
      .name
        flex: 1
        font-size: $font-size-medium
        color: $color-text-d
        overflow: hidden
        .text
          no-wrap()
    .no-result-wrapper
      position: absolute
      width: 100%
      top: 50%
      transform: translateY(-50%)
</style>