<script setup>
import { ref, computed, watch } from 'vue';
const searchKey = ref('')
const activePage = ref(1)
const uBikeStops = ref([])
const perPage = 10

// 欄位說明:
// sno：站點代號、 sna：場站名稱(中文)、 total：場站總停車格、
// available_rent_bikes：場站目前可用車輛數量、 
// sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態
fetch('https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json')
  .then(res => res.text())
  .then(data => {
    uBikeStops.value = JSON.parse(data);
  })

const filterBikeStops = computed(() => {
  return uBikeStops.value.filter((stop) => stop.sna.includes(searchKey.value))
})
const pageFilterBikeStops = computed(() => {
  const start = (activePage.value - 1) * perPage
  return filterBikeStops.value.slice(start, start + 10)
})

const totalPage = computed(() => Math.ceil(filterBikeStops.value.length / perPage))
const pagination = computed(() => {
  const pagesToShow = 10
  let start = Math.max(activePage.value - Math.floor(pagesToShow / 2), 1)
  let end = Math.min(start + pagesToShow - 1, totalPage.value)

  start = Math.max(end - pagesToShow + 1, 1)

  const pages = []
  for (let i = start; i <= end; i++) {
    pages.push(i)
  }
  return pages
})

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6')
}

const changePage = (type) => {
  if (type === 'forward' && activePage.value !== 1) {
    activePage.value --
  } 
  if (type === 'backward' && activePage.value !== totalPage.value) {
    activePage.value ++
  }
}

watch(searchKey, () => {
  activePage.value = 1
})
</script>

<template>  
<!--
練習： YouBike 臺北市公共自行車即時資訊，實作以下功能
1. 站點名稱搜尋
2. 目前可用車輛 / 總停車格 的排序功能
3. 分頁功能 (Optional)
-->
  <div class="my-4">
    <p class="my-4 pl-2">
      站點名稱搜尋: 
      <input v-model="searchKey" type="text" class="border w-60 p-1 ml-2">
    </p>
    
    <table class="table table-striped">
      <thead>
        <tr>
          <th class="w-12">#</th>
          <th>場站名稱</th>
          <th>場站區域</th>
          <th>目前可用車輛
            <i class="fa fa-sort-asc" aria-hidden="true"></i>
            <i class="fa fa-sort-desc" aria-hidden="true"></i>
          </th>
          <th>總停車格
            <i class="fa fa-sort-asc" aria-hidden="true"></i>
            <i class="fa fa-sort-desc" aria-hidden="true"></i>
          </th>
          <th>資料更新時間</th>          
        </tr>
      </thead>
      <tbody>
        <tr v-for="(s, idx) in pageFilterBikeStops" :key="s.sno">
          <td>{{ idx +1 }}</td>
          <td>{{ s.sna }}</td>
          <td>{{ s.sarea }}</td>
          <td>{{ s.available_rent_bikes }}</td>
          <td>{{ s.total }}</td>
          <td>{{ timeFormat(s.mday) }}</td>
        </tr>
      </tbody>
    </table>
    
    <!-- 頁籤 -->
    <ul v-if="totalPage > 1" class="my-4 flex justify-center">
      <li class="page-item cursor-pointer" @click="activePage = 1">
        <span class="page-link">第一頁</span>
      </li>
      <li class="page-item cursor-pointer" @click="changePage('forward')">
        <span class="page-link">&lt;</span>
      </li>

      <li 
        v-for="number in pagination" 
        :key="number"
        class="page-item cursor-pointer" 
        :class="{ 'active': activePage === number }"
        @click="activePage = number"
      >
        <span class="page-link">{{ number }}</span>
      </li>
      
      <li class="page-item cursor-pointer" @click="changePage('backward')">
        <span class="page-link" href>&gt;</span>
      </li>      
      <li class="page-item cursor-pointer" @click="activePage = totalPage">
        <span class="page-link">最末頁</span>
      </li>
    </ul>

  </div>
</template>
