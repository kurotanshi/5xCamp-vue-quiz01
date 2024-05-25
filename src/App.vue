<script setup>
import { ref,onMounted, watch, computed } from 'vue';
const uBikeStops = ref([]);
const searchInput = ref('')
const currentPage = ref(1)
const pageSize = 10


// 欄位說明:
// sno：站點代號、 sna：場站名稱(中文)、 total：場站總停車格、
// available_rent_bikes：場站目前可用車輛數量、 
// sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態

const apiURL = 'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
const fetchUBikeData = async () => {
  const response = await fetch(apiURL)
  return await response.json()
}
onMounted(async () => {
  const data = await fetchUBikeData()
  uBikeStops.value = data
})


const filterData = computed(() => {
  if(searchInput.value) {
    return uBikeStops.value.filter(uBikeStop => uBikeStop.sna.includes(searchInput.value))
  } else {
    return uBikeStops.value
  }
})

const resultData = computed(() => {
  const start = (currentPage.value - 1) * pageSize
  const end = currentPage.value * pageSize
  console.log(currentPage.value, (currentPage.value - 1) * pageSize);
  return filterData.value.slice(start, end)
})
const totalPages = computed(() => Math.ceil(filterData.value.length / pageSize))

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6');
};

// 分頁
const pages = computed(() => {
  let startPage = Math.max(1, currentPage.value - 5)
  let endPage = Math.min(startPage + pageSize - 1, totalPages.value)
  if (endPage - startPage + 1 < 10) { // 處理最後一頁，補齊 10 個頁碼
    startPage = Math.max(1, endPage - 10 + 1);
  }

  return Array.from({ length: endPage - startPage + 1 }, (x, i) => startPage + i )
})

const highLightSearchText = (searchText) => {
  const regex = new RegExp(searchInput.value, 'gi')
  return searchText.replace(regex, `<span style="color:red;">${searchInput.value}</span>`)
}

const changePage = (page) => {
  if(page < 1) {
    currentPage.value = 1
    return
  }
  if(page > totalPages.value) {
    currentPage.value = totalPages.value
    return
  }
  currentPage.value = page
}

watch(searchInput, () => changePage(1))


const searchData = (sortIcon, key) => {
  switch(sortIcon) {
    case 'sort-asc':
      uBikeStops.value.sort((a,b) => a[key] - b[key])
    break

    case 'sort-desc':
      uBikeStops.value.sort((a, b) => b[key] - a[key])
    break
  }
}

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
      <input type="text" v-model="searchInput" @change="searchData" class="border w-60 p-1 ml-2">
    </p>
    
    <table class="table table-striped">
      <thead>
        <tr>
          <th class="w-12">#</th>
          <th>場站名稱</th>
          <th>場站區域</th>
          <th>目前可用車輛
            <i class="fa fa-sort-asc pointer" aria-hidden="true" @click="searchData('sort-asc', 'available_rent_bikes')"></i>
            <i class="fa fa-sort-desc pointer" aria-hidden="true" @click="searchData('sort-desc', 'available_rent_bikes')"></i>
          </th>
          <th>總停車格
            <i class="fa fa-sort-asc pointer" aria-hidden="true" @click="searchData('sort-asc', 'total')"></i>
            <i class="fa fa-sort-desc pointer" aria-hidden="true" @click="searchData('sort-desc', 'total')"></i>
          </th>
          <th>資料更新時間</th>          
        </tr>
      </thead>
      <tbody>
        <tr v-for="(s, idx) in resultData" :key="s.sno">
          <td>{{ idx +1 }}</td>
          <td v-html="highLightSearchText(s.sna)"></td>
          <td>{{ s.sarea }}</td>
          <td>{{ s.available_rent_bikes }}</td>
          <td>{{ s.total }}</td>
          <td>{{ timeFormat(s.mday) }}</td>
        </tr>
      </tbody>
    </table>
    
    <!-- 頁籤 -->
    <ul class="my-4 flex justify-center">
      <li
        class="page-item cursor-pointer"  
        @click="changePage(1)"
      >
        <span class="page-link">第一頁</span>
      </li>
      <li   
        class="page-item cursor-pointer"    
        @click="changePage(currentPage - 1)"
      >
        <span class="page-link">&lt;</span>
      </li>
      <li 
        class="page-item cursor-pointer"
        v-for="page in pages" :key="page"
        :class="{ active: currentPage === page }"
        @click="changePage(page)"
      >
        <span class="page-link">{{ page }}</span>
      </li>

      <li 
        class="page-item cursor-pointer"
        @click="changePage(currentPage + 1)"
      >
        <span class="page-link" href>&gt;</span>
      </li>      
      <li 
        class="page-item cursor-pointer"
        @click="changePage(totalPages)"
      >
        <span class="page-link">最末頁</span>
      </li>
    </ul>

  </div>
</template>

<style scoped>
  .pointer {
    cursor: pointer;
  }
</style>