<script setup>
import { ref, computed } from 'vue';
const uBikeStops = ref([]);
const searchResult = ref('');
const sortKey = ref('');
const sortOrder = ref(1);

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

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6');
};

// 過濾和排序 uBikeStops
const filterUBikeStops = computed(() => {
  let filteredStops = uBikeStops.value;
  if (searchResult.value) {
    filteredStops = filteredStops.filter(station => station.sna.includes(searchResult.value));
  }
  if (sortKey.value) {
    filteredStops.sort((a, b) => {
      const aValue = a[sortKey.value];
      const bValue = b[sortKey.value];
      if (typeof aValue === 'number' && typeof bValue === 'number') {
        return (aValue - bValue) * sortOrder.value;
      } else {
        return aValue.localeCompare(bValue) * sortOrder.value;
      }
    });
  }
  return filteredStops;
});

// 關鍵字 highlight 處理
const highlightKeyword = (text) => {
  if (!searchResult.value) {
    return text;
  }
  const index = text.toLowerCase().indexOf(searchResult.value.toLowerCase());
  if (index === -1) {
    return text;
  }
  const beforeMatch = text.substring(0, index);
  const match = text.substring(index, index + searchResult.value.length);
  const afterMatch = text.substring(index + searchResult.value.length);
  return `${beforeMatch}<span class="highlight">${match}</span>${afterMatch}`;
};

// 處理排序
const handleSort = (key) => {
  if (sortKey.value === key) {
    sortOrder.value = -sortOrder.value;
  } else {
    sortKey.value = key;
    sortOrder.value = 1;
  }
};


</script>

<style>
  .highlight > span {
    background-color: #ffffcc;
    color: #ff0022;
  }
  .sortable {
    cursor: pointer;
  }
  .sortable:hover {
    color: #f2195b;
  }
</style>

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
      <input type="text" class="border w-60 p-1 ml-2" v-model="searchResult">
    </p>
    
    <table class="table table-striped">
      <thead>
        <tr>
          <th class="w-12">#</th>
          <th>場站名稱</th>
          <th>場站區域</th>
          <th class="sortable" @click="handleSort('available_rent_bikes')">
            目前可用車輛
            <i :class="{'fa fa-sort-asc': sortKey === 'available_rent_bikes' && sortOrder === 1, 'fa fa-sort-desc': sortKey === 'available_rent_bikes' && sortOrder === -1}" aria-hidden="true"></i>
          </th>
          <th class="sortable" @click="handleSort('total')">
            總停車格
            <i :class="{'fa fa-sort-asc': sortKey === 'total' && sortOrder === 1, 'fa fa-sort-desc': sortKey === 'total' && sortOrder === -1}" aria-hidden="true"></i>
          </th>
          <th>資料更新時間</th>          
        </tr>
      </thead>
      <tbody>
        <tr v-for="(s, idx) in filterUBikeStops" :key="s.sno">
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
    <ul class="my-4 flex justify-center">
      <li class="page-item cursor-pointer">
        <span class="page-link">第一頁</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link">&lt;</span>
      </li>

      <li class="page-item cursor-pointer active">
        <span class="page-link">1</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link">2</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link">3</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link">4</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link">5</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link">6</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link">7</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link">8</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link">9</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link">10</span>
      </li>

      <li class="page-item cursor-pointer">
        <span class="page-link" href>&gt;</span>
      </li>      
      <li class="page-item cursor-pointer">
        <span class="page-link">最末頁</span>
      </li>
    </ul>

  </div>
</template>
