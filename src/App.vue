<script setup>
import { computed, ref, watch } from 'vue';
const uBikeStops = ref([]);

// 資料來源: https://data.ntpc.gov.tw/openapi/swagger-ui/index.html?configUrl=%2Fapi%2Fv1%2Fopenapi%2Fswagger%2Fconfig&urls.primaryName=%E6%96%B0%E5%8C%97%E5%B8%82%E6%94%BF%E5%BA%9C%E4%BA%A4%E9%80%9A%E5%B1%80(94)#/JSON/get_010e5b15_3823_4b20_b401_b1cf000550c5_json

// 欄位說明: 
// sno：站點代號、 sna：場站名稱(中文)、 total：場站總停車格、
// available_rent_bikes：場站目前可用車輛數量、 
// sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態

// page: 頁碼, size: 每頁筆數, 全部 349 筆.

//每頁顯示
const itemPerPage = ref(10);
// 所有比數
const itemsNum = 349;
// 所有頁數
const pagesTotal = computed(()=>{
  return Math.ceil( itemsNum / itemPerPage.value)
});

// 一開始頁
fetch('https://data.ntpc.gov.tw/api/datasets/010e5b15-3823-4b20-b401-b1cf000550c5/json?page=1&size=10')
  .then(res => res.text())
  .then(data => {
    uBikeStops.value = JSON.parse(data);
});

const nowPage = ref(1);

// 觀察在哪一頁
const WatchNowPage = (i)=>{
  let n = itemPerPage.value;
  nowPage.value = i;
  fetch(`https://data.ntpc.gov.tw/api/datasets/010e5b15-3823-4b20-b401-b1cf000550c5/json?page=${i}&size=${n}`)
    .then(res => res.text())
    .then(data => {
      uBikeStops.value = JSON.parse(data);
    });
  watch(itemPerPage, ()=>{
    fetch(`https://data.ntpc.gov.tw/api/datasets/010e5b15-3823-4b20-b401-b1cf000550c5/json?page=${i}&size=${n}`)
    .then(res => res.text())
    .then(data => {
      uBikeStops.value = JSON.parse(data);
    });
  })  
}


const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6');
};

// --- Search
const searchQuery = ref("");
const filterUBikeStops = ref([]);
let isEmpty = true;
watch(searchQuery, (newSearch)=>{
  filterUBikeStops.value = uBikeStops.value.filter((word)=>{
      // word.sna.match(new RegExp(newSearch, 'i'));
      if(searchQuery.value) {return word.sna.match(newSearch)}
      // return word.sna.includes('智慧')
  })
  isEmpty = !isEmpty;
})

const cartSortUp = ()=>{
  if(filterUBikeStops){
    filterUBikeStops.value.sort((a, b)=>{
      return a.sbi - b.sbi
    })
  }
  uBikeStops.value.sort((a, b)=>{
    return a.sbi - b.sbi
  })
}
const cartSortDown = ()=>{
  if(filterUBikeStops){
    filterUBikeStops.value.sort((a, b)=>{
      return b.sbi - a.sbi
    })
  }
  uBikeStops.value.sort((a, b)=>{
    return b.sbi - a.sbi
  })
}
const parkinglotSortUp = ()=>{
  if(filterUBikeStops){
    filterUBikeStops.value.sort((a, b)=>{
      return a.tot - b.tot
    })
  }
  uBikeStops.value.sort((a, b)=>{
    return a.tot - b.tot
  })
}
const parkinglotSortDown = ()=>{
  if(filterUBikeStops){
    filterUBikeStops.value.sort((a, b)=>{
      return b.tot - a.tot
    })
  }
  uBikeStops.value.sort((a, b)=>{
    return b.tot - a.tot
  })
}



</script>

<template>  
<!--
練習： YouBike 新北市公共自行車即時資訊，實作以下功能
  1. 站點名稱搜尋
  2. 目前可用車輛 / 總停車格 的排序功能
  3. 分頁功能, 已知總筆數為 349 筆

已知問題未解：
  1. 頁碼 active 怎麼加
  2. 每頁顯示筆數切換後，內容沒有watch跟著變
  3. code 整理簡潔
-->

  <div class="">
    <div class="grid grid-cols-2 my-4 px-4 w-full mx-auto">
      <div class="pl-2">
        目前頁面的站點名稱搜尋: <input v-model="searchQuery" type="text" class="border w-60 p-1 ml-2">
      </div>
      <div class="pl-2">
        每頁顯示筆數:
        <select class="border w-20 p-1 ml-2" v-model="itemPerPage">
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
            <i @click="cartSortUp" class="fa fa-sort-asc btn btn-primary mx-1" aria-hidden="true"></i>
            <i @click="cartSortDown" class="fa fa-sort-desc btn btn-primary mx-1" aria-hidden="true"></i>
          </th>
          <th>總停車格
            <i @click="parkinglotSortUp" class="fa fa-sort-asc btn btn-primary mx-1" aria-hidden="true"></i>
            <i @click="parkinglotSortDown" class="fa fa-sort-desc btn btn-primary mx-1" aria-hidden="true"></i>
          </th>
          <th>資料更新時間</th>          
        </tr>
      </thead>
      <tbody>
        <template v-if="isEmpty">
          <tr v-for="(s, idx) in uBikeStops" :key="s.sno">
            <td>{{ idx +1 }}</td>
            <td>{{ s.sna }}</td>
            <td>{{ s.sarea }}</td>
            <td>{{ s.sbi }}</td>
            <td>{{ s.tot }}</td>
            <td>{{ timeFormat(s.mday) }}</td>
          </tr>
        </template>
        <template v-else>
          <tr v-for="(s, idx) in filterUBikeStops" :key="s.sno">
            <td>{{ idx +1 }}</td>
            <td>{{ s.sna }}</td>
            <td>{{ s.sarea }}</td>
            <td>{{ s.sbi }}</td>
            <td>{{ s.tot }}</td>
            <td>{{ timeFormat(s.mday) }}</td>
          </tr>
        </template>
      </tbody>
    </table>
    
    <!-- 頁籤 -->
    <ul class="my-4 flex justify-center pagination">
      <li class="page-item cursor-pointer">
        <span class="page-link">第一頁</span>
      </li>
      <li class="page-item cursor-pointer">
        <span class="page-link">&lt;</span>
      </li>

      <li
      :class="{active: i === nowPage}"
      v-for="i in pagesTotal"
      @click.lazy="WatchNowPage(i)"
      class="page-item cursor-pointer">
        <span class="page-link">{{ i }}</span>
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

<style>
.pagination {
  flex-wrap: wrap;
  max-width: 1000px;
  width: 100%;
  margin: auto;
}
</style>