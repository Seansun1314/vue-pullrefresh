<template>
  <div id="container" class="vue-pull-refresh" @touchstart="touchstart" @touchmove="touchmove" @touchend="touchend" :style="istouchmoveStyle">
    <div class="vue-pull-refresh-msg">
      <template v-if="loading">
        转圈圈~啦啦啦~
      </template>
      <template v-else>
        {{msg}}
      </template>
    </div>
    <slot name="list"></slot>
  </div>
</template>
<script>
  export default {
    props: {
      next: { // 刷新函数
        type: Function,
        required: true
      }
    },
    data() {
      return {
        msg: '',
        flag: 0, // 表示是否达到刷新条件
        loading: false,  // 表示是否正在刷新中
        touchStartPos: 0,  // 手指触摸屏幕的起点
        distance: 0,     // 手指滑动的距离
        scrollTop: Number,
        istouchmoveStyle: 'color: red'  // 手指移动时改变样式
      }
    },
    methods: {
      // 手指触碰
      touchstart(e) {
        //   // 如果loading为true就代表刷新函数正在进行，此时阻止下拉操作，返回
        //   if (this.loading) { 
        //     e.preventDefault()
        //     return
        //   }
        // 取第一个手指的触摸点作为起始点
        this.touchStartPos = e.changedTouches[0].clientY;
        console.log(this.touchStartPos);
      },
      // 手指移动
      touchmove(e) {
        //   // 如果没有触摸起始点，返回
        //   if (!this.touchStart) {
        //     return
        //   }
        //   if (this.loading) {
        //     e.preventDefault()
        //     return
        //   }

        var _this = this;
        const touch = e.changedTouches[0];
        const query = wx.createSelectorQuery();
        query.select('#container').boundingClientRect();
        query.selectViewport().scrollOffset();
        query.exec(function (res) {
          res[0].scrollTop // #the-id节点的上边界坐标
          _this.scrollTop = res[1].scrollTop;
          console.log(_this.scrollTop);
        })
        if (_this.scrollTop === 0) {
          _this.distance = touch.clientY - _this.touchStartPos;
          if (_this.distance > 0 && _this.distance < 80) {
            // e.preventDefault()

            _this.istouchmoveStyle = 'overflow: inherit; transform: translate3D(0px, ' + _this.distance +'px, 0px)';
            if (_this.distance > 55) {
                _this.msg = '释放刷新'
                _this.flag = 1
            } else {
                _this.msg = '下拉刷新'
            }
          }
        }
      },

      touchend() {
        var _this = this;
        if (_this.flag && _this.distance > 0) {
          console.log(_this.loading);
          _this.istouchmoveStyle = 'overflow: visible; transform: translate3D(0px, 50px, 0px)';
          _this.loading = true;
          console.log(_this.loading);
          _this.next().then(() => {
              _this.flag = 0
              _this.loading = false;
              _this.scrollTop = 0
              _this.istouchmoveStyle = 'overflow: auto; transform: translate3D(0px, 0px, 0px)';
          })
          return
        }
        // 重置变量
        _this.flag = 0
        _this.istouchmoveStyle = 'overflow: auto; transform: translate3D(0px, 0px, 0px)';
      }

    },
    mounted() {
    }
  }
</script>
<style>
  .vue-pull-refresh {
    height: 85%;
    overflow-y: auto;
    transition: 330ms;
    -webkit-overflow-scrolling: touch;
  }
  .vue-pull-refresh-msg {
    margin-top: -50px;
    height: 50px;
    text-align: center;
    line-height: 50px;
    color: #666;
    border-bottom: 1px solid #eee;
  }
  .vue-pull-refresh-msg .icon-reverse {
    transform: rotate(180deg);
    transition: all .3s ease;
  }
  .vue-pull-refresh-loading {
    -webkit-animation:loadRotate 1s linear infinite;
    animation:loadRotate 1s linear infinite;
  }
  @-webkit-keyframes loadRotate{
    from{
      -webkit-transform: rotate(0deg);
    }
    to{
      -webkit-transform: rotate(360deg);
    }
  }
  @keyframes loadRotate{
    from{
      transform: rotate(0deg);
    }
    to{
      transform: rotate(360deg);
    }
  }
</style>
