<script setup>
import { computed, ref, watch } from 'vue';
const uBikeStops = ref([]);

const searchText = ref('');
const sort = ref({
  key: 'index', // index、available_rent_bikes、total
  order: 'asc',
});

const currentPage = ref(1);
const pageSize = 10;
const pageGroupSize = 10;

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
  });

// 當搜尋文字改變時，將當前頁數設為 1
watch(searchText, () => {
  currentPage.value = 1;
});

// 設定排序
const setSort = (key) => {
  if (sort.value.key === key) {
    sort.value.order = sort.value.order === 'asc' ? 'desc' : 'asc';
  } else {
    sort.value.key = key;
    sort.value.order = 'asc'; // 預設升冪
  }
};

// 判斷是否顯示排序圖示
const isSorted = (key) => {
  return sort.value.key === key;
};

// 獲取排序圖示的 className
const sortIconClass = computed(() => {
  return sort.value.order === 'asc' ? 'fa fa-sort-asc' : 'fa fa-sort-desc';
});

// 處理後的站點資料 (文字搜尋+排序)
const processedStop = computed(() => {
  const filtered = uBikeStops.value.filter(s => s.sna.includes(searchText.value));

  let sorted = [];
  const sortKey = sort.value.key;
  const sortOrder = sort.value.order;

  switch (sortKey) {
    case 'available_rent_bikes':
    case 'total':
      sorted = filtered.sort((a, b) => sortOrder === 'asc' ? a[sortKey] - b[sortKey] : b[sortKey] - a[sortKey]);
      break;
    case 'index':
      sorted = sortOrder === 'asc' ? filtered : filtered.reverse();
      break
    default:
      sorted = filtered;
      break;
  }

  return sorted;
  
});

// 分頁資料 (將處理後的資料進行分頁處理)
const pageItems = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  const end = start + pageSize;
  return processedStop.value.slice(start, end);
});

// 設定頁碼
const setPage = (page) => {
  if (page && page <= totalPages.value) { // 頁數範圍防呆
    currentPage.value = page;
  }
};

// 總頁數
const totalPages = computed(() => {
  return Math.ceil(processedStop.value.length / pageSize);
});

// 頁碼
const pageNumbers = computed(() => {
  let startPage = Math.max(1, currentPage.value - 5); // 保持當前頁數在中間 (第六個)
  let endPage = Math.min(startPage + pageGroupSize - 1, totalPages.value);

  if (endPage - startPage + 1 < pageGroupSize) { // 處理最後一頁，補齊 10 個頁碼
    startPage = Math.max(1, endPage - pageGroupSize + 1);
  }

  return Array.from({ length: endPage - startPage + 1 }, (_, i) => startPage + i);
});

// 標記關鍵字
const highlightKeyword = (text) => {
  if (!searchText.value) return text;
  return text.replace(new RegExp(searchText.value, 'gi'), '<span style="color: red;">$&</span>');
};

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6');
};
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
      <input v-model="searchText" type="text" class="border w-60 p-1 ml-2">
    </p>
    
    <table class="table table-striped">
      <thead>
        <tr>
          <th 
            class="w-12 cursor-pointer" 
            @click="setSort('index')"
          >
            #
            <i 
              v-if="isSorted('index')" 
              :class="sortIconClass" 
              aria-hidden="true"
            ></i>
          </th>
          <th>場站名稱</th>
          <th>場站區域</th>
          <th 
            class="cursor-pointer"  
            @click="setSort('available_rent_bikes')"
          >
            目前可用車輛
            <i 
              v-if="isSorted('available_rent_bikes')" 
              :class="sortIconClass" 
              aria-hidden="true"
            ></i>
          </th>
          <th
            class="cursor-pointer"
            @click="setSort('total')"
          >
            總停車格
            <i 
              v-if="isSorted('total')" 
              :class="sortIconClass" 
              aria-hidden="true"
            ></i>
          </th>
          <th>資料更新時間</th>          
        </tr>
      </thead>
      <tbody>
        <tr v-for="(s, idx) in pageItems" :key="s.sno">
          <td>{{ idx +1 }}</td>
          <td v-html="highlightKeyword(s.sna)"></td>
          <td>{{ s.sarea }}</td>
          <td>{{ s.available_rent_bikes }}</td>
          <td>{{ s.total }}</td>
          <td>{{ timeFormat(s.mday) }}</td>
        </tr>
      </tbody>
    </table>
    
    <!-- 頁籤 -->
    <ul 
      class="my-4 flex justify-center"
      v-if="totalPages > 1"
    >
      <!-- 第一頁 -->
      <li 
        class="page-item cursor-pointer" 
        @click="setPage(1)"
      >
        <span class="page-link">第一頁</span>
      </li>
      <!-- 上一頁 -->
      <li 
        class="page-item cursor-pointer" 
        @click="setPage(currentPage - 1)"
      >
        <span class="page-link">&lt;</span>
      </li>
      <!-- 頁碼-->
      <li 
        v-for="page in pageNumbers" 
        :key="page" 
        :class="{ active: currentPage === page }" 
        @click="setPage(page)"
        class="page-item cursor-pointer" 
      >
        <span class="page-link">{{ page }}</span>
      </li>
      <!-- 下一頁 -->
      <li 
        class="page-item cursor-pointer" 
        @click="setPage(currentPage + 1)"
      >
        <span class="page-link">&gt;</span>
      </li>
      <!-- 最後一頁 -->
      <li 
        class="page-item cursor-pointer" 
        @click="setPage(totalPages)"
      >
        <span class="page-link">最末頁</span>
      </li>
    </ul>

  </div>
</template>
