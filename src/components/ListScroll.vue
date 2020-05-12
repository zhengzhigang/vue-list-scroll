<template>
  <div class="list">
    <ul ref="container">
      <li
        :ref="index === 0 ? 'firstItem' : (index === list.length - 1 ? 'lastItem' : '')"
        :id="index === 0 ? 'firstItem' : (index === list.length - 1 ? 'lastItem' : '')"
        v-for="(item, index) in list"
        :key="index">{{item}}</li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data() {
    return {
      firstItemId: 'firstItem',
      lastItemId: 'lastItem',
      firstItem: null,
      lastItem: null,
      itemHeight: 150,
      container: null,
      listSize: 10,
      domDataCache: {
        currentPaddingTop: 0,
        currentPaddingBottom: 0,
        topSentinelPreviousY: 0,
        topSentinelPreviousRatio: 0,
        bottomSentinelPreviousY: 0,
        bottomSentinelPreviousRatio: 0,
        currentIndex: 0
      },
      observer: null,
      list: []
    }
  },

  created() {
    for (let i = 0; i < 10; i++) {
      this.list.push(i)
    }
  },

  mounted() {
    this.container = this.$refs.container
    this.firstItem = this.$refs['firstItem']
    this.lastItem = this.$refs['lastItem']
    this.startObserver()
  },

  methods: {
    startObserver() {
      this.initIntersectionObserver()
    },

    renderFunction(firstIndex) {
      this.renderPage(firstIndex)
    },

    updateDomDataCache(params) {
        Object.assign(this.domDataCache, params)
    },

    // 动态调整容器padding实现滚动
    // eslint-disable-next-line class-methods-use-this
    adjustPaddings(isScrollDown) {
        const { container, itemHeight } = this
        const { currentPaddingTop, currentPaddingBottom } = this.domDataCache

        let newCurrentPaddingTop, newCurrentPaddingBottom

        // TODO 150待抽象
        const remPaddingsVal = itemHeight * (Math.floor(this.listSize / 2))

        if (isScrollDown) {
            newCurrentPaddingTop = currentPaddingTop + remPaddingsVal

            if (currentPaddingBottom === 0) {
                newCurrentPaddingBottom = 0
            } else {
                newCurrentPaddingBottom = currentPaddingBottom - remPaddingsVal
            }
        } else {
            newCurrentPaddingBottom = currentPaddingBottom + remPaddingsVal

            if (currentPaddingTop === 0) {
                newCurrentPaddingTop = 0
            } else {
                newCurrentPaddingTop = currentPaddingTop - remPaddingsVal
            }
        }

        container.style.paddingBottom = `${newCurrentPaddingBottom}px`
        container.style.paddingTop = `${newCurrentPaddingTop}px`

        this.updateDomDataCache({
            currentPaddingTop: newCurrentPaddingTop,
            currentPaddingBottom: newCurrentPaddingBottom
        })
    },

    getWindowFirstIndex(isScrollDown) {
        const {
            currentIndex
        } = this.domDataCache;

        // 以全部容器内所有元素的一半作为增量
        const increment = Math.floor(this.listSize / 2);

        let firstIndex;

        if (isScrollDown) {
            firstIndex = currentIndex + increment;
        } else {
            firstIndex = currentIndex - increment;
        }

        if (firstIndex < 0) {
            firstIndex = 0;
        }

        return firstIndex;
    },

    topItemCb(el) {
      const {
        topSentinelPreviousY,
        topSentinelPreviousRatio
      } = this.domDataCache;
      const currentY = el.boundingClientRect.top
      const currentRatio = el.intersectionRatio // 相交区域和目标元素的比例值 intersectionRect/boundingClientRect 不可见时小于等于0
      const isIntersecting = el.isIntersecting // 元素是否可见
      // 上滑精准判定
      // 保证是在隐藏后再次上拉出现第一个、并且再currentIndex变化(currentIndex !== 0)之后，
      // 否则可能反复（隐藏 -> 出现) 的操作出现多次rendering
      if (
        currentY > topSentinelPreviousY
        && isIntersecting
        && currentRatio >= topSentinelPreviousRatio
      ) {
        console.log('topSentCallback.. go');
        const firstIndex = this.getWindowFirstIndex(false);
        this.renderFunction(firstIndex)
        this.adjustPaddings(false)

        this.updateDomDataCache({
            currentIndex: firstIndex,
            topSentinelPreviousY: currentY,
            topSentinelPreviousRatio: currentRatio
        })
      } else {
        this.updateDomDataCache({
          topSentinelPreviousY: currentY,
          topSentinelPreviousRatio: currentRatio
        })
      }
    },

    bottomitemCb(entry) {
      const {
        bottomSentinelPreviousY,
        bottomSentinelPreviousRatio
      } = this.domDataCache

      // TODO：hasMore => 外层控制
      // if (currentIndex === DBSize - listSize) {
      //     return;
      // }

      const currentY = entry.boundingClientRect.top
      const currentRatio = entry.intersectionRatio
      const isIntersecting = entry.isIntersecting

      // 下滑精准判定 => TODO 更深入明确理解
      if (
        currentY < bottomSentinelPreviousY
        && currentRatio >= bottomSentinelPreviousRatio
        && isIntersecting
      ) {
        console.log('botSentCallback.. go')
        const firstIndex = this.getWindowFirstIndex(true)

        this.renderFunction(firstIndex)
        this.adjustPaddings(true)

        this.updateDomDataCache({
            currentIndex: firstIndex,
            bottomSentinelPreviousY: currentY,
            bottomSentinelPreviousRatio: currentRatio
        })
      } else {
        this.updateDomDataCache({
            bottomSentinelPreviousY: currentY,
            bottomSentinelPreviousRatio: currentRatio
        })
      }
    },

    callback(entries) {
      entries.forEach(el => {
        if (el.target.id === this.firstItemId) {
          this.topItemCb(el)
        } else {
          this.bottomitemCb(el)
        }
      })
    },

    initIntersectionObserver() {
      const options = {
      }
      this.observer = new IntersectionObserver(this.callback, options)

      // 观察列表第一个及i最后一个元素
      this.observer.observe(this.firstItem[0])
      this.observer.observe(this.lastItem[0])
    },

    renderPage(firstIndex) {
      let arr = []
      for (let i = 0 ; i < 10; i++) {
        arr.push(firstIndex + i)
      }
      this.list = arr
    }
  }
}
</script>

<style scoped lang="scss">
  ul {
    list-style: none;
    padding: 0;
    margin: 0;
    li {
      height: 150px;
      border: 1px solid #eee;
      line-height: 150px;
    }
  }
</style>
