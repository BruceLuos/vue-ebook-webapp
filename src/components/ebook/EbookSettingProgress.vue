<template>
  <transition name="slide-up">
    <div class="setting-wrapper" v-show="menuVisible && settingVisible === 2">
      <div class="setting-progress">
        <!-- 获取阅读时间 -->
        <div class="read-time-wrapper">
          <span class="read-time-text">{{getReadTimeText()}}</span>
          <span class="icon-forward"></span>
        </div>
        <div class="progress-wrapper">
          <!-- 前一章节 -->
          <div class="progress-icon-wrapper" @click="prevSection()">
            <span class="icon-back"></span>
          </div>
          <!-- 进度条 -->
          <input class="progress" type="range"
                 max="100"
                 min="0"
                 step="1"
                 @change="onProgressChange($event.target.value)"
                 @input="onProgressInput($event.target.value)"
                 :value="progress"
                 :disabled="!bookAvailable"
                 ref="progress">
          <!-- 下一章节 -->
          <div class="progress-icon-wrapper" @click="nextSection()">
            <span class="icon-forward"></span>
          </div>
        </div>
        <div class="text-wrapper">
          <!-- 当前章节名 -->
          <span class="progress-section-text">{{getSectionName}}</span>
          <span>({{bookAvailable ? progress + '%' : '加载中...'}})</span>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
  import { ebookMixin } from '../../utils/mixin'
  export default {
    mixins: [ebookMixin],
    // progress，bookAvilible,section等很多可复用属性都放在mixin中
    computed: {

    },
    methods: {
      // 进度条发生改变时执行的方法
      // 拖动时获取值为progress进度值 （0-100）
      onProgressChange(progress) {
        this.setProgress(progress).then(() => {
          this.displayProgress()
          // 改变进度条的背景色
          this.updateProgressBg()
        })
      },
      // 拖动进度条中执行的方法
      onProgressInput(progress) {
        this.setProgress(progress).then(() => {
          // 改变进度条的背景色
          this.updateProgressBg()
        })
      },
      // 显示进度条
      displayProgress() {
        // 书籍内容的百分比
        const cfi = this.currentBook.locations.cfiFromPercentage(this.progress / 100)
        // 重新定位书籍内容并展示
        this.display(cfi)
        // this.currentBook.rendition.display(cfi).then(() => {
        //   this.refreshLocation()
        // })
      },
      // 改变进度条背景色
      updateProgressBg() {
        this.$refs.progress.style.backgroundSize = `${this.progress}% 100%`
      },
      // 上一章
      prevSection() {
        if (this.section > 0 && this.bookAvailable) {
          this.setSection(this.section - 1).then(() => {
            this.displaySection()
          })
        }
      },
      nextSection() {
        // spine表示章节进度对象
        if (this.section < this.currentBook.spine.length - 1 && this.bookAvailable) {
          this.setSection(this.section + 1).then(() => {
            this.displaySection()
          })
        }
      },
      // 展示章节
      displaySection() {
        // 上一章节信息
            const sectionInfo = this.currentBook.section(this.section)
            // 当存在上一章节信息和地址时，展示在电子书上
            if (sectionInfo && sectionInfo.href) {
              this.display(sectionInfo.href)
              // this.currentBook.rendition.display(sectionInfo.href).then(() => {
              //   //刷新章节进度
              //   this.refreshLocation()
              // })
            }
      }
      //刷新章节进度,这里EbookReader也需要用到所以要把他放在mixin中
      // refreshLocation () {
      //   // 获取电子书的currentLocation对象里面有登记当前章节的开始位置等信息
      //   const currentLocation = this.currentBook.rendition.currentLocation()
      //   console.log(currentLocation)
      //   const startCfi = currentLocation.start.cfi
      //   const progress =  this.currentBook.locations.percentageFromCfi(startCfi)
      //   console.log(progress)
      //   this.setProgress(Math.floor(progress * 100))
      //   // 缓存章节进度
      //   saveLocation(this.fileName, startCfi)
      // }
    },
    updated() {
      // 初始状态下的进度条背景色
      this.updateProgressBg()
    }
  }
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
  @import "../../assets/styles/global";

  .setting-wrapper {
    position: absolute;
    bottom: px2rem(48);
    left: 0;
    z-index: 160;
    width: 100%;
    height: px2rem(90);
    background: white;
    box-shadow: 0 px2rem(-8) px2rem(8) rgba(0, 0, 0, .15);
    .setting-progress {
      position: relative;
      width: 100%;
      height: 100%;
      .read-time-wrapper {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: px2rem(40);
        font-size: px2rem(12);
        @include center;
      }
      .progress-wrapper {
        width: 100%;
        height: 100%;
        @include center;
        padding: 0 px2rem(15);
        box-sizing: border-box;
        .progress-icon-wrapper {
          font-size: px2rem(20);
        }
        .progress {
          width: 100%;
          -webkit-appearance: none;
          height: px2rem(2);
          margin: 0 px2rem(10);
          &:focus {
            outline: none;
          }
          &::-webkit-slider-thumb {
            -webkit-appearance: none;
            height: px2rem(20);
            width: px2rem(20);
            border-radius: 50%;
            background: white;
            box-shadow: 0 4px 4px 0 rgba(0, 0, 0, .35);
            border: px2rem(1) solid #ddd;
          }
        }
      }
      .text-wrapper {
        position: absolute;
        left: 0;
        bottom: px2rem(10);
        width: 100%;
        color: #333;
        font-size: px2rem(12);
        padding: 0 px2rem(15);
        box-sizing: border-box;
        @include center;
        .progress-section-text {
          @include ellipsis;
        }
      }
    }
  }
</style>
