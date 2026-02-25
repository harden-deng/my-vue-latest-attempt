<template>
  <!-- 虚拟列表外层容器：核心是固定高度+overflow:auto+相对定位 -->
  <div class="virtual-list" ref="listRef" @scroll="handleScroll">
    <!-- 占位层：用padding-top模拟十万条数据的总高度，生成正常滚动条 -->
    <div class="virtual-list-placeholder" :style="{ paddingTop: totalHeight + 'px' }"></div>
    <!-- 可视区渲染层：绝对定位，只渲染截取后的少量数据 -->
    <div class="virtual-list-view" :style="{ top: viewTop + 'px' }">
      <div class="virtual-list-item" v-for="item in showData" :key="item.id"
        :style="{ height: itemHeight + 'px', lineHeight: itemHeight + 'px' }">
        {{ item.content }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'

// -------------------------- 基础配置（可根据业务自定义） --------------------------
const itemHeight = 50 // 每条列表项的固定高度（px），核心配置
const bufferCount = 5 // 可视区上下缓冲条数（3-5条即可，防止滚动时出现空白）
const listRef = ref(null) // 列表容器Ref，用于获取滚动/容器信息

// -------------------------- 模拟后端一次性返回的十万条数据 --------------------------
const bigData = ref([])
// 初始化10万条测试数据（实际开发中替换为接口返回的全量数据）
const initBigData = () => {
  const data = []
  for (let i = 0; i < 100000; i++) {
    data.push({
      id: i + 1, // 唯一key，避免Vue列表渲染警告
      content: `虚拟列表数据 - 第${i + 1}条`
    })
  }
  bigData.value = data
}

// -------------------------- 滚动核心状态（响应式） --------------------------
const scrollState = reactive({
  scrollTop: 0,    // 容器滚动距离（top）
  containerHeight: 0, // 列表容器的可视高度
  totalCount: 0,   // 全量数据总条数
  startIndex: 0,   // 可视区起始索引（要渲染的第一条数据下标）
  endIndex: 0      // 可视区结束索引（要渲染的最后一条数据下标+1，适配slice）
})

// -------------------------- 计算属性：推导核心渲染参数 --------------------------
// 列表总高度：用于占位层的padding-top，生成正常滚动条
const totalHeight = computed(() => {
  scrollState.totalCount = bigData.value.length
  return scrollState.totalCount * itemHeight
})

// 可视区要渲染的切片数据（核心：只截取少量数据，减少DOM）
const showData = computed(() => {
  return bigData.value.slice(scrollState.startIndex, scrollState.endIndex)
})

// 渲染层的top偏移值：让渲染的DOM始终定位在可视区
const viewTop = computed(() => {
  return scrollState.startIndex * itemHeight
})

// -------------------------- 核心方法：计算可视区索引范围 --------------------------
const calculateViewRange = () => {
  if (!listRef.value) return
  // 更新容器可视高度（兼容容器大小变化）
  scrollState.containerHeight = listRef.value.clientHeight
  // 计算可视区基础起始索引：滚动距离 / 每条高度，向下取整
  const baseStartIndex = Math.floor(scrollState.scrollTop / itemHeight)  //向下取整
  // 最终起始索引：减去缓冲条数，且不能小于0（防止越界）
  scrollState.startIndex = Math.max(0, baseStartIndex - bufferCount) //最大值为0，最小值为baseStartIndex - bufferCount
  // 计算可视区可显示的基础条数：容器高度 / 每条高度，向上取整
  const baseShowCount = Math.ceil(scrollState.containerHeight / itemHeight)  //向上取整
  // 最终结束索引：基础条数 + 2倍缓冲条数（上下各缓冲），且不超过总条数
  scrollState.endIndex = Math.min(
    scrollState.totalCount,   //100000
    baseStartIndex + baseShowCount + bufferCount  //'' + '' + 5 = 200005
  ) //最小值为scrollState.totalCount，最大值为baseStartIndex + baseShowCount + bufferCount
}

// -------------------------- 滚动事件处理 --------------------------
const handleScroll = (e) => {
  console.log('滚动事件：', e)
  console.log('滚动距离：', e.target.scrollTop)
  // 更新滚动距离，触发计算属性重新计算
  scrollState.scrollTop = e.target.scrollTop
  // 计算最新的可视区索引范围
  calculateViewRange()
}

// -------------------------- 生命周期：初始化/销毁 --------------------------
onMounted(() => {
  // 初始化十万条数据
  initBigData()
  // 初始化计算可视区范围
  calculateViewRange()
})

onUnmounted(() => {
  // 销毁时清空大数据，防止内存泄漏
  bigData.value = []
})
</script>

<style scoped>
/* 虚拟列表容器：核心是固定高度+overflow:auto */
.virtual-list {
  width: 800px;
  height: 600px;
  margin: 20px auto;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: auto;
  position: relative;
}

/* 占位层：用于模拟总高度，无实际内容 */
.virtual-list-placeholder {
  width: 100%;
}

/* 可视区渲染层：绝对定位，宽度100% */
.virtual-list-view {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
}

/* 列表项样式：可根据业务自定义 */
.virtual-list-item {
  width: 100%;
  border-bottom: 1px solid #f5f5f5;
  box-sizing: border-box;
  padding: 0 20px;
  color: #333;
}
</style>