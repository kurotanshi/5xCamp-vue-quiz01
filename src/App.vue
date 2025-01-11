<script setup>
import { ref, computed, watch } from 'vue';
const uBikeStops = ref([]);

// 資料來源: https://data.ntpc.gov.tw/openapi/swagger-ui/index.html?configUrl=%2Fapi%2Fv1%2Fopenapi%2Fswagger%2Fconfig&urls.primaryName=%E6%96%B0%E5%8C%97%E5%B8%82%E6%94%BF%E5%BA%9C%E4%BA%A4%E9%80%9A%E5%B1%80(94)#/JSON/get_010e5b15_3823_4b20_b401_b1cf000550c5_json

// 欄位說明: 
// sno：站點代號、 sna：場站名稱(中文)、 total：場站總停車格、
// available_rent_bikes：場站目前可用車輛數量、 
// sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態

// page: 頁碼, size: 每頁筆數, 全部 349 筆.
fetch('/api/datasets/010e5b15-3823-4b20-b401-b1cf000550c5/json?page=1&size=999')
  .then(res => res.text())
  .then(data => {
    uBikeStops.value = JSON.parse(data);
  });

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6');
};

const dataShow = computed(()=>{
  let filterColumn = [];
  if(uBikeStops.value.length > 0) {
    // get keys
    filterColumn = Object.keys(uBikeStops.value[0]);
  }
  // filter keyword
  const filterData = uBikeStops.value.filter(data => {
    let includs = false;
    for (let i = 0; i < filterColumn.length; i++) {
      if (data[filterColumn[i]]) {
        includs = includs || `${data[filterColumn[i]]}`.toLowerCase().includes(keyword.value);
      }
    }
    console.log("keyword:", keyword.value, "includs:", includs);
    if (keyword.value.length > 0)
      return includs;
    else
      return true;
  })
  
  // sort column
  if (sortColumn1.value) {
    filterData.sort((a, b) => {
      let dateA = a["sbi"];
      let dateB = b["sbi"];
      if (Number.isInteger(dateA) && Number.isInteger(dateB)) {
        return sortColumn1.value === "asc" ? dateA - dateB : dateB - dateA;
      } else {
        dateA += "";
        dateB += "";
      }

      return sortColumn1.value === "asc" ? dateA.localeCompare(dateB) : dateB.localeCompare(dateA);
    });
  } else if(sortColumn2.value) {
    filterData.sort((a, b) => {
      let dateA = a["tot"];
      let dateB = b["tot"];
      console.log("isInteger(dateA)", isInteger(dateA), dateA);
      console.log("isInteger(dateB)", isInteger(dateB), dateB);
      if (isInteger(dateA) && isInteger(dateB)) {
        return sortColumn2.value === "asc" ? dateA - dateB : dateB - dateA;
      } else {
        dateA += "";
        dateB += "";
      }

      return sortColumn2.value === "asc" ? dateA.localeCompare(dateB) : dateB.localeCompare(dateA);
    });
  }

  console.log("filterData:", filterData);
  return filterData;
})
const keyword = ref("");
const count = ref(10);
const pageNo = ref(1);
const sortColumn1 = ref("asc");
const sortColumn2 = ref("");

function changeSort(columnName){
  
  if(!this[columnName]) {
    this[columnName] = "asc";
    
    if(columnName === "sortColumn1") {
      sortColumn2.value = "";
    } else {
      sortColumn1.value = "";
    }
  } else if(this[columnName] === "asc") {
    this[columnName] = "desc";
  } else {
    this[columnName] = "asc";
  }
}

function isInteger(x) {
    return x == parseInt(x, 10);
}

const _page = ref( 1 );

function fn_beforePage(){
  const __page = _page.value-1;
  if(__page < 1) {
    _page.value = 1;
  } else {
    _page.value = __page;
  }
}

function fn_nextPage(){
  const __page = _page.value+1;
  if(__page > Math.ceil(dataShow.value.length/count.value)) {
    fn_lastPage();
  } else {
    _page.value = __page
  }
}

function fn_lastPage(){
  console.log(dataShow.length);
  _page.value = Math.ceil(dataShow.value.length/count.value);
}

watch(() => count.value,
  (newValue, oldValue) => {
    if(newValue > oldValue) {
      if(_page.value > Math.ceil(dataShow.value.length/count.value)) {
        fn_lastPage();
      }
    }
  }
)
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
        目前頁面的站點名稱搜尋: <input type="text" v-model="keyword" class="border w-60 p-1 ml-2">
      </div>
      <div class="pl-2">
        每頁顯示筆數:
        <select v-model="count" class="border w-20 p-1 ml-2">
          <option :value="10">10</option>
          <option :value="20">20</option>
          <option :value="30">30</option>
        </select>
      </div>
    </div>

    
    <table class="table table-striped">
      <thead>
        <tr>
          <th class="w-12">#</th>
          <th>場站名稱</th>
          <th>場站區域</th>
          <th @click="changeSort('sortColumn1')">目前可用車輛
            <i v-show="sortColumn1==='asc'" class="fa fa-sort-asc" aria-hidden="true"></i>
            <i v-show="sortColumn1==='desc'" class="fa fa-sort-desc" aria-hidden="true"></i>
          </th>
          <th @click="changeSort('sortColumn2')">總停車格
            <i v-show="sortColumn2==='asc'" class="fa fa-sort-asc" aria-hidden="true"></i>
            <i v-show="sortColumn2==='desc'" class="fa fa-sort-desc" aria-hidden="true"></i>
          </th>
          <th>資料更新時間</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(s, idx) in dataShow"  :key="s.sno">
          <template v-if="idx>=count*(_page-1) && idx<count*(_page)">
              <td>{{ idx +1 }}</td>
              <td>{{ s.sna }}</td>
              <td>{{ s.sarea }}</td>
              <td>{{ s.sbi }}</td>
              <td>{{ s.tot }}</td>
              <td>{{ timeFormat(s.mday) }}</td>
          </template>
        </tr>
      </tbody>
    </table>
    
    <!-- 頁籤 -->
    <ul class="my-4 flex justify-center">
      <li class="page-item cursor-pointer" @click="_page=1">
        <span class="page-link">第一頁</span>
      </li>
      <li class="page-item cursor-pointer" @click="fn_beforePage()">
        <span class="page-link">&lt;</span>
      </li>
      <template v-for="n in 10">
        <li v-if="n <= Math.ceil(dataShow.length/count)" class="page-item cursor-pointer" :class="{active:(n == _page)}" @click="_page = n">
          <span class="page-link"> {{ n }}</span>
        </li>
      </template>
      <li class="page-item cursor-pointer" @click="fn_nextPage()">
        <span class="page-link" href>&gt;</span>
      </li>
      <li class="page-item cursor-pointer" @click="fn_lastPage()">
        <span class="page-link">最末頁</span>
      </li>
    </ul>

  </div>
</template>
