<script setup>
import {computed, ref, watch} from 'vue';
const uBikeStops = ref([]);
const searchKey = ref('');
const sortedKey = ref('');
const sortedType = ref('asc');
const rawPerPage = ref(10);
const currentPage = ref(1);
const filteredStops = ref([]);

// 資料來源: https://data.ntpc.gov.tw/openapi/swagger-ui/index.html?configUrl=%2Fapi%2Fv1%2Fopenapi%2Fswagger%2Fconfig&urls.primaryName=%E6%96%B0%E5%8C%97%E5%B8%82%E6%94%BF%E5%BA%9C%E4%BA%A4%E9%80%9A%E5%B1%80(94)#/JSON/get_010e5b15_3823_4b20_b401_b1cf000550c5_json

// 欄位說明: 
// sno：站點代號、 sna：場站名稱(中文)、 total：場站總停車格、
// available_rent_bikes：場站目前可用車輛數量、 
// sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態

// page: 頁碼, size: 每頁筆數, 全部 349 筆.
fetch('https://data.ntpc.gov.tw/api/datasets/010e5b15-3823-4b20-b401-b1cf000550c5/json?page=1&size=999')
  .then(res => res.text())
  .then(data => {
    uBikeStops.value = JSON.parse(data);
  });

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6');
};

const setSortedKeyAndType = (key, type) => {
  console.log('key', 'type', key, type);
  sortedKey.value = key;
  sortedType.value = type;
};

const handledStops = computed(() => {
  let outputStops = [];
  filteredStops.value = uBikeStops.value.filter(s => s.sna.includes(searchKey.value));
  outputStops = filteredStops.value;
  
  if (sortedKey.value !== '') {
    outputStops = filteredStops.value.sort((a, b) => {
      if (sortedType.value === 'desc') {
        return b[sortedKey.value] - a[sortedKey.value];
      }
      return a[sortedKey.value] - b[sortedKey.value];
    });
  }
   return outputStops.slice((currentPage.value-1)*(rawPerPage.value), (currentPage.value)*(rawPerPage.value));
});

const resetCurrentPageByChangeRawPerPage = watch(() => rawPerPage.value, () => {
  currentPage.value = 1;
});

const resetCurrentPageByChangeSearchKey = watch(() => searchKey.value, () => {
  currentPage.value = 1;
});

const totalPageCount = computed(() => {
  return Math.ceil(filteredStops.value.length / rawPerPage.value);
});

const setPage = (page) => {
  currentPage.value = page;
};

const lastPage = () => {
  if (currentPage.value > 1) {
    currentPage.value -= 1;
  }
};

const nextPage = () => {
  if (currentPage.value < totalPageCount.value) {
    currentPage.value += 1;
  }
};
</script>

<template>  
<!--
練習： YouBike 新北市公共自行車即時資訊，實作以下功能
  1. 站點名稱搜尋
  2. 目前可用車輛 / 總停車格 的排序功能
  3. 分頁功能, 已知總筆數為 349 筆
-->

  <div class="">
    <div class="grid grid-cols-2 my-4 px-4 w-full mx-auto">
      <div class="pl-2">
        目前頁面的站點名稱搜尋: <input type="text" class="border w-60 p-1 ml-2" v-model="searchKey">
      </div>
      <div class="pl-2">
        每頁顯示筆數: 
        <select class="border w-20 p-1 ml-2" v-model="rawPerPage">
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
          <th>目前可用車輛
            <i class="fa fa-sort-asc cursor-pointer" aria-hidden="true" v-on:click="setSortedKeyAndType('sbi', 'asc')"></i>
            <i class="fa fa-sort-desc cursor-pointer" aria-hidden="true" v-on:click="setSortedKeyAndType('sbi', 'desc')"></i>
          </th>
          <th>總停車格
            <i class="fa fa-sort-asc cursor-pointer" aria-hidden="true" v-on:click="setSortedKeyAndType('tot', 'asc')"></i>
            <i class="fa fa-sort-desc cursor-pointer" aria-hidden="true" v-on:click="setSortedKeyAndType('tot', 'desc')"></i>
          </th>
          <th>資料更新時間</th>          
        </tr>
      </thead>
      <tbody>
        <tr v-for="(s, idx) in handledStops" :key="s.sno">
          <td>{{ idx +1 }}</td>
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
      <li class="page-item cursor-pointer">
        <span class="page-link" v-on:click="setPage(1)">第一頁</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link" v-on:click="lastPage">&lt;</span>
      </li>
      <li class="page-item cursor-pointer" 
          v-for="(s, idx) in totalPageCount" 
          :class="{ active: currentPage === idx + 1 }">
        <span class="page-link" v-on:click="setPage(idx+1)">{{idx+1}}</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link" href v-on:click="nextPage">&gt;</span>
      </li>      
      <li class="page-item cursor-pointer">
        <span class="page-link" v-on:click="setPage(totalPageCount)">最末頁</span>
      </li>
    </ul>

  </div>
</template>
