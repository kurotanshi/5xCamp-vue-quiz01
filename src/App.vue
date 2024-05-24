<script setup>
import { ref } from 'vue';
const uBikeStops = ref([]);
const searchValue = ref('');
const searchResult = ref([]);
let pageNumber = 10;
let bonus = 0;
let avalableIsSortAsc = 'x'; // x:未排列, +:升冪排列, -:降冪排列
let totalIsSortAsc = 'x'; // x:未排列, +:升冪排列, -:降冪排列

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
    searchResult.value = uBikeStops.value.slice(0, 10);
  });

const search = (isInputSearch) => {
  console.log('search')
  searchResult.value = uBikeStops.value.filter(d => d.sna.includes(searchValue.value));
  pageNumber = Math.ceil(searchResult.value.length/10);
  if (pageNumber > 10) {
    pageNumber = 10;
  }
  console.log(Math.ceil(searchResult.value.length/10))
  if (isInputSearch) {
    searchResult.value = searchResult.value.slice(0, 10);
  }
}

const sortAsc = (label, isPageItem) => {
  console.log('sortAsc')
  search(false);
  searchResult.value.sort((a, b) => {
    if (label === 'available') {
      avalableIsSortAsc = '+';
      totalIsSortAsc = 'x';
      return a.available_rent_bikes - b.available_rent_bikes;
    } else {
      totalIsSortAsc = '+';
      avalableIsSortAsc = 'x';
      return a.total - b.total;
    }
  });
  if (!isPageItem) {
    searchResult.value = searchResult.value.slice(0, 10);
  }
}

const sortDesc = (label, isPageItem) => {
  console.log('sortDesc')
  search(false);
  searchResult.value.sort((a, b) => {
    if (label === 'available') {
      avalableIsSortAsc = '-';
      totalIsSortAsc = 'x';
      return b.available_rent_bikes - a.available_rent_bikes;
    } else {
      totalIsSortAsc = '-';
      avalableIsSortAsc = 'x';
      return b.total - a.total;
    }
  });
  if (!isPageItem) {
    searchResult.value = searchResult.value.slice(0, 10);
  }
}

const pageItem = (page) => {
  console.log('page: ' + page)
  if (avalableIsSortAsc === '+') {
    sortAsc('available' ,true);
  } else if (avalableIsSortAsc === '-') {
    sortDesc('available', true);
  } else if (totalIsSortAsc === '+') {
    sortAsc('total', true);
  } else if (totalIsSortAsc === '-') {
    sortDesc('total', true);
  } else {
    search(false);
  }
  searchResult.value = searchResult.value.slice((page-1)*10, page*10);
}

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
      <input type="text" class="border w-60 p-1 ml-2" v-model="searchValue" @input="search(true)">
    </p>
    
    <table class="table table-striped">
      <thead>
        <tr>
          <th class="w-12">#</th>
          <th>場站名稱</th>
          <th>場站區域</th>
          <th>目前可用車輛
            <i class="fa fa-sort-asc" aria-hidden="true" @click="sortAsc('available', false)"></i>
            <i class="fa fa-sort-desc" aria-hidden="true" @click="sortDesc('available', false)"></i>
          </th>
          <th>總停車格
            <i class="fa fa-sort-asc" aria-hidden="true" @click="sortAsc('total', false)"></i>
            <i class="fa fa-sort-desc" aria-hidden="true" @click="sortDesc('total', false)"></i>
          </th>
          <th>資料更新時間</th>          
        </tr>
      </thead>
      <tbody>
        <tr v-for="(s, idx) in searchResult" :key="s.sno">
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
    <ul class="my-4 flex justify-center">
      <li class="page-item cursor-pointer">
        <span class="page-link" @click="pageItem(1)">第一頁</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link" @click="bonus-=10">&lt;</span>
      </li>

      <li v-for="idx in pageNumber" class="page-item cursor-pointer" :key="idx+bonus">
        <span class="page-link" @click="pageItem(idx+bonus)">{{idx+bonus}}</span>
      </li>

      <li class="page-item cursor-pointer">
        <span class="page-link" href @click="bonus+=10">&gt;</span>
      </li>      
      <li class="page-item cursor-pointer">
        <span class="page-link">最末頁</span>
      </li>
    </ul>

  </div>
</template>
<style>
</style>
