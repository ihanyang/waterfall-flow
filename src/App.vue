<template>
    <button @click="add">点击新增数据</button>
    <div class="wrapper" ref="wrapper">
        <div class="item" :style="{ transform: 'translate(' + item.x + 'px, ' + item.y + 'px)', width: item.width + 'px', height: item.height + 'px', backgroundColor: item.bg }" v-for="item of list">{{ item.text }}</div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const list = ref([])
const wrapper = ref(null)
const width = ref(0)

let num
const gap = 20
let count = new URLSearchParams(location.search).get('count') ?? 20
const colorArr = ['#409eff', '#67c23a', '#e6a23c', '#f56c6c', '#909399']

onMounted(() => {
    num = getNum(wrapper.value.clientWidth)

    width.value = (wrapper.value.clientWidth - ((num - 1) * gap)) / num

    list.value = Array.from({length: count}).fill(0).map((item, index) => ({
        width: width.value,
        height: Math.ceil(Math.random() * 300 + 100),
        x: 0,
        y: 0,

        index,
        text: index + 1,
        bg: colorArr[index % (colorArr.length - 1)]
    }))

    // 初始化
    setTransform()

    // 监听尺寸更新
    const ro = new ResizeObserver(entries => {
      for (let entry of entries) {
        num = getNum(entry.contentRect.width)

        width.value = (entry.contentRect.width - ((num - 1) * gap)) / num

        list.value.forEach((item) => {
            item.width = width.value
        })

        setTransform()
      }
    })

    ro.observe(wrapper.value)
})

const getNum = (width) => {
    let num

    if (width >= 1600) {
        num = 5
    }

    if (width < 1600 && width >= 1300) {
        num = 4
    }

    if (width < 1300 && width >= 1000) {
        num = 3
    }

    if (width < 1000) {
        num = 2
    }

    return num
}

const getSortedList = (start, end) => list.value.slice(start, end).sort((a, b) => (a.y + a.height) - (b.y + b.height))

const setTransform = (startIndex = 0) => {
    let index = startIndex
    let height = 0

    while(index < list.value.length) {

        const min = index === 0 ? [] : getSortedList(index - num, index)
        const currentList = list.value.slice(index, index + num)

        currentList.forEach((item, i) => {
            if (index === 0) {
                item.x = i * width.value + i * gap
                item.y = 0
            } else {
                item.x = min[i].x,
                item.y = min[i].height + min[i].y + gap
            }
        })

        // 获取最高列
        const item = currentList.sort((a, b) => a.y + a.height - (b.y + b.height)).at(-1)

        height = Math.max(height, item.y + item.height)

        index += num
    }

    wrapper.value.style.height = height + 20 + 'px'
}

const add = () => {
    const arr = Array.from({length: 10}).fill(0).map((item, index) => ({
        width: width.value,
        height: Math.ceil(Math.random() * 300 + 100),
        x: 0,
        y: 0,

        index: index + list.value.length,
        text: index + 1 + list.value.length,
        bg: colorArr[index % (colorArr.length - 1)]
    }))

    const length = list.value.length

    list.value = [... list.value, ... arr]

    // 从新增数据处开始处理
    setTransform(length % num === 0 ? length : length - (length % num))
}

</script>

<style scoped>
button {
    width: 150px;
    margin: 20px auto;
    text-align: center;
    color: #000;
    cursor: pointer;
    border-radius: 20px;
    line-height: 50px;
}
.wrapper {
    position: relative;
    box-sizing: border-box;
    background-color: gray;
}
.item {
    display: flex;
    align-items: center;
    justify-content: center;
    position: absolute;
    top: 0;
    left: 0;
    font-size: 40px;
    border-radius: 15px;
    transition: all .3s;
}
</style>
