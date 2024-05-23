<script setup>
import { ref } from "vue";
import { useDebounceFn } from "@vueuse/core";

const uBikeStops = ref([]);
const filterUBikeStops = ref([]);
const searchKey = ref("");

const search = () => {
  if (searchKey.value === "") {
    calculatePage();
    return;
  }
  const filterUBikeStops = uBikeStops.value.filter((item) =>
    item.sna.includes(searchKey.value)
  );
  calculatePage(filterUBikeStops);
};
const searchDebounce = useDebounceFn(() => {
  search();
}, 1000);

const sortData = (order, name) => {
  if (order === "asc") {
    filterUBikeStops.value.sort((a, b) => a[name] - b[name]);
  } else {
    filterUBikeStops.value.sort((a, b) => b[name] - a[name]);
  }
};

const calculatePage = (data = uBikeStops.value) => {
  filterUBikeStops.value = data;
};

// 欄位說明:
// sno：站點代號、 sna：場站名稱(中文)、 total：場站總停車格、
// available_rent_bikes：場站目前可用車輛數量、
// sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態
fetch(
  "https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json"
)
  .then((res) => res.text())
  .then((data) => {
    uBikeStops.value = JSON.parse(data);
    calculatePage();
  });

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, "$1/$2/$3 $4:$5:$6");
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
      <input
        type="text"
        class="border w-60 p-1 ml-2"
        @keyup.enter="search"
        @input="searchDebounce"
        v-model="searchKey"
      />
    </p>

    <table class="table table-striped">
      <thead>
        <tr>
          <th class="w-12">#</th>
          <th>場站名稱</th>
          <th>場站區域</th>
          <th>
            目前可用車輛
            <i
              class="fa fa-sort-asc"
              aria-hidden="true"
              @click="sortData('asc', 'available_rent_bikes')"
            ></i>
            <i
              class="fa fa-sort-desc"
              aria-hidden="true"
              @click="sortData('desc', 'available_rent_bikes')"
            ></i>
          </th>
          <th>
            總停車格
            <i
              class="fa fa-sort-asc"
              aria-hidden="true"
              @click="sortData('asc', 'total')"
            ></i>
            <i
              class="fa fa-sort-desc"
              aria-hidden="true"
              @click="sortData('desc', 'total')"
            ></i>
          </th>
          <th>資料更新時間</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(s, idx) in filterUBikeStops" :key="s.sno">
          <td>{{ idx + 1 }}</td>
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
