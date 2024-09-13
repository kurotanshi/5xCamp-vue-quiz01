<script setup>
import { ref, computed, watch } from 'vue';

const uBikeStops = ref([]);
const itemsPerPage = ref(10);
const currentPage = ref(1);
const searchQuery = ref('');
const sortKey = ref('');
const sortOrder = ref(1);

// 資料來源: https://data.ntpc.gov.tw/openapi/swagger-ui/index.html?configUrl=%2Fapi%2Fv1%2Fopenapi%2Fswagger%2Fconfig&urls.primaryName=%E6%96%B0%E5%8C%97%E5%B8%82%E6%94%BF%E5%BA%9C%E4%BA%A4%E9%80%9A%E5%B1%80(94)#/JSON/get_010e5b15_3823_4b20_b401_b1cf000550c5_json

// 欄位說明:
// sno：站點代號、 sna：場站名稱(中文)、 total：場站總停車格、
// available_rent_bikes：場站目前可用車輛數量、
// sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態

// page: 頁碼, size: 每頁筆數, 全部 349 筆.
const fetchUBikeStops = () => {
  fetch(`https://data.ntpc.gov.tw/api/datasets/010e5b15-3823-4b20-b401-b1cf000550c5/json?page=${currentPage.value}&size=${itemsPerPage.value}`)
    .then(res => res.json())
    .then(data => {
      uBikeStops.value = data;
    })
    .catch(error => {
      console.error('Error fetching data:', error);
    });
};

// Fetch data when currentPage or itemsPerPage changes
watch([currentPage, itemsPerPage], fetchUBikeStops, { immediate: true });

const timeFormat = (val) => {
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6');
};

const filteredUBikeStops = computed(() => {
  return uBikeStops.value.filter(stop =>
    stop.sna.includes(searchQuery.value)
  );
});

const sortedUBikeStops = computed(() => {
  return filteredUBikeStops.value.slice().sort((a, b) => {
    if (sortKey.value) {
      return (a[sortKey.value] - b[sortKey.value]) * sortOrder.value;
    }
    return 0;
  });
});

const changeSort = (key) => {
  if (sortKey.value === key) {
    sortOrder.value = -sortOrder.value;
  } else {
    sortKey.value = key;
    sortOrder.value = 1;
  }
};

const totalPages = computed(() => {
  return Math.ceil(349 / itemsPerPage.value); // Assuming total items are 349
});

const pageNumbers = computed(() => {
  const total = totalPages.value;
  const current = currentPage.value;
  const maxPagesToShow = 10;
  let startPage = Math.max(1, current - Math.floor(maxPagesToShow / 2));
  let endPage = startPage + maxPagesToShow - 1;

  if (endPage > total) {
    endPage = total;
    startPage = Math.max(1, endPage - maxPagesToShow + 1);
  }

  const pages = [];
  for (let i = startPage; i <= endPage; i++) {
    pages.push(i);
  }
  return pages;
});
</script>

<template>
  <div class="">
    <div class="grid grid-cols-2 my-4 px-4 w-full mx-auto">
      <div class="pl-2">
        目前頁面的站點名稱搜尋:
        <input v-model="searchQuery" type="text" class="border w-60 p-1 ml-2">
      </div>
      <div class="pl-2">
        每頁顯示筆數:
        <select v-model="itemsPerPage" class="border w-20 p-1 ml-2">
          <option value="10">10</option>
          <option value="20">20</option>
          <option value="30">30</option>
        </select>
      </div>
    </div>

    <table class="table table-striped">
      <thead>
      <tr>
        <th class="w-12">#</th>
        <th>場站名稱</th>
        <th>場站區域</th>
        <th @click="changeSort('sbi')">目前可用車輛
          <i class="fa" :class="sortKey === 'sbi' && sortOrder === 1 ? 'fa-sort-asc' : 'fa-sort-desc'" aria-hidden="true"></i>
        </th>
        <th @click="changeSort('tot')">總停車格
          <i class="fa" :class="sortKey === 'tot' && sortOrder === 1 ? 'fa-sort-asc' : 'fa-sort-desc'" aria-hidden="true"></i>
        </th>
        <th>資料更新時間</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(s, idx) in sortedUBikeStops" :key="s.sno">
        <td>{{ (currentPage - 1) * itemsPerPage + idx + 1 }}</td>
        <td>{{ s.sna }}</td>
        <td>{{ s.sarea }}</td>
        <td>{{ s.sbi }}</td>
        <td>{{ s.tot }}</td>
        <td>{{ timeFormat(s.mday) }}</td>
      </tr>
      </tbody>
    </table>

    <!-- 頁籤 -->
    <ul class="my-4 flex justify-center">
      <li class="page-item cursor-pointer" @click="currentPage = 1">
        <span class="page-link">第一頁</span>
      </li>
      <li class="page-item cursor-pointer" @click="currentPage = Math.max(1, currentPage - 1)" v-if="currentPage > 1">
        <span class="page-link">&lt;</span>
      </li>

      <li v-for="page in pageNumbers" :key="page" class="page-item cursor-pointer" :class="{ active: currentPage === page }" @click="currentPage = page">
        <span class="page-link">{{ page }}</span>
      </li>

      <li class="page-item cursor-pointer" @click="currentPage = Math.min(totalPages, currentPage + 1)" v-if="currentPage < totalPages">
        <span class="page-link">&gt;</span>
      </li>
      <li class="page-item cursor-pointer" @click="currentPage = totalPages">
        <span class="page-link">最末頁</span>
      </li>
    </ul>
  </div>
</template>