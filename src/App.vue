<script setup>
import { computed, ref } from 'vue';
const uBikeStops = ref([]);
const keyword = ref('');
const sortKey = ref('sno');
const sortOrder = ref('asc');

const currentPage = ref(1);
const perPage = ref(10); // 每頁顯示筆數
const maxpage = ref(10); // 最大頁碼數
const pageControlCount = ref(10);


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


// 篩選站點 (搜尋) + 排序 computed
const queryStops = computed(() => {
  console.log('queryStops');
  let stops = uBikeStops.value.filter((item) => {
    return item.sna.includes(keyword.value);
  });
  // 排序
  if (sortKey.value) {
    stops = stops.sort((a, b) => {
      if (sortOrder.value === 'asc') {
        return a[sortKey.value] - b[sortKey.value];
      } else {
        return b[sortKey.value] - a[sortKey.value];
      }
    });
  }

  //頁碼數
  maxpage.value = Math.ceil(stops.length / perPage.value);

  return stops.slice(currentPage.value === 1 ? 0 : (currentPage.value - 1) * perPage.value,
    perPage.value * currentPage.value);
});

// 關鍵字高亮
const hightlight = (val) => {
  console.log('hightlight');
  if (!keyword.value) {
    return val;
  }
  return val.replace(keyword.value, `<span style="color:red">${keyword.value}</span>`);
};

// 排序
const sort = (key) => {
  if (sortKey.value === key) {
    sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc';
  } else {
    sortKey.value = key;
    sortOrder.value = 'asc';
  }
};

// 分頁
const changePage = (page) => {
  currentPage.value = page;
  console.log(page);
};

// 第一頁
const firstPage = () => {
  currentPage.value = 1;
};

// 最後一頁
const lastPage = () => {

  currentPage.value = maxpage.value;
  console.log(currentPage.value);
};

const prevPage = () => {
  currentPage.value = currentPage.value > 1 ? currentPage.value - 1 : 1;
};

const nextPage = () => {
  currentPage.value = currentPage.value !== maxpage.value ? currentPage.value + 1 : maxpage.value;
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
      <input type="text" class="border w-60 p-1 ml-2" v-model="keyword">
    </p>

    <table class="table table-striped">
      <thead>
        <tr>
          <th class="w-12 cursor-pointer" @click="sort('sno')">#<i class="fa" v-if="sortKey === 'sno'"
              :class="sortOrder === 'asc' ? 'fa-sort-asc' : 'fa-sort-desc'" aria-hidden="true"></i></th>
          <th>場站名稱</th>
          <th>場站區域</th>
          <th class="cursor-pointer" @click="sort('available_rent_bikes')">目前可用車輛
            <i class="fa" v-if="sortKey === 'available_rent_bikes'"
              :class="sortOrder === 'asc' ? 'fa-sort-asc' : 'fa-sort-desc'" aria-hidden="true"></i>
          </th>
          <th class="cursor-pointer" @click="sort('total')">總停車格
            <i class="fa" v-if="sortKey === 'total'" :class="sortOrder === 'asc' ? 'fa-sort-asc' : 'fa-sort-desc'"
              aria-hidden="true"></i>
          </th>
          <th>資料更新時間</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(s, idx) in queryStops" :key="s.sno">
          <td>{{ idx + 1 }}</td>
          <td v-html="hightlight(s.sna)"></td>
          <td>{{ s.sarea }}</td>
          <td>{{ s.available_rent_bikes }}</td>
          <td>{{ s.total }}</td>
          <td>{{ timeFormat(s.mday) }}</td>
        </tr>
      </tbody>
    </table>

    <!-- 頁籤 -->
    <ul class="my-4 flex justify-center flex-wrap">
      <li class="page-item " @click="firstPage"
        :class="{ disabled: currentPage === 1, 'cursor-pointer': currentPage !== 1 }">
        <span class="page-link">第一頁</span>
      </li>
      <li class="page-item " @click="prevPage"
        :class="{ disabled: currentPage === 1, 'cursor-pointer': currentPage !== 1 }">
        <span class="page-link">&lt;</span>
      </li>

      <li class="page-item cursor-pointer" v-for="i in maxpage" :key="i" :class="currentPage === i ? 'active' : ''"
        @click="changePage(i)">
        <span class="page-link">{{ i }}</span>
      </li>

      <li class="page-item cursor-pointer" @click="nextPage"
        :class="{ disabled: currentPage === maxpage, 'cursor-pointer': currentPage !== maxpage }">
        <span class="page-link" href>&gt;</span>
      </li>
      <li class="page-item cursor-pointer" @click="lastPage"
        :class="{ disabled: currentPage === maxpage, 'cursor-pointer': currentPage !== maxpage }">
        <span class="page-link">最末頁</span>
      </li>
    </ul>

  </div>
</template>
