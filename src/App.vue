<script setup>
import { computed, ref, watch } from "vue";
import { useDebounceFn } from "@vueuse/core";

const uBikeStops = ref([]);
const filterUBikeStops = ref([]);
const searchKey = ref("");
const currentPage = ref(1);
const totalPages = ref(0);
const limit = ref(10);
const visiblePages = computed(() => {
  const start = Math.max(1, currentPage.value - 5);
  const end = Math.min(totalPages.value, currentPage.value + 4);
  return Array.from({ length: end - start + 1 }, (_, i) => start + i);
});
const isFirstPage = computed(() => currentPage.value === 1);
const isLastPage = computed(() => currentPage.value >= totalPages.value);

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

const showMorePage = (page) => {
  return visiblePages.value.includes(page);
};
const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++;
    calculatePage();
  }
};
const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--;
    calculatePage();
  }
};
const goToFirstPage = () => {
  currentPage.value = 1;
  calculatePage();
};
const goToLastPage = () => {
  currentPage.value = totalPages.value;
  calculatePage();
};
const goToPage = (page) => {
  currentPage.value = page;
  calculatePage();
};
const calculatePage = (data = uBikeStops.value) => {
  const start = (currentPage.value - 1) * limit.value;
  const end = start + limit.value;
  filterUBikeStops.value = data.slice(start, end);
  totalPages.value = Math.ceil(data.length / limit.value);
};

watch(limit, () => {
  // 問題：這裡有一個 BUG 不確定怎麼解比較好，當前頁面在最末頁時，改變每頁筆數，會計算錯誤
  calculatePage();
});

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
    <div class="flex items-center justify-center space-x-4">
      <ul class="my-4 flex">
        <li class="page-item cursor-pointer">
          <span
            class="page-link"
            :class="{
              'text-gray-400 bg-gray-100 cursor-not-allowed hover:text-gray-400':
                isFirstPage,
            }"
            :disabled="isFirstPage"
            @click="goToFirstPage"
            >第一頁</span
          >
        </li>
        <li class="page-item cursor-pointer">
          <span
            class="page-link"
            :class="{
              'text-gray-400 bg-gray-100 cursor-not-allowed hover:text-gray-400':
                isFirstPage,
            }"
            :disabled="isFirstPage"
            @click="prevPage"
            >&lt;</span
          >
        </li>
        <li class="page-item" v-if="!showMorePage(1)">
          <span class="page-link cursor-not-allowed text-gray-400 border-y-0"
            >...</span
          >
        </li>
        <li
          class="page-item cursor-pointer"
          :class="{ active: currentPage === page }"
          v-for="page in visiblePages"
          :key="page"
        >
          <span class="page-link" @click="goToPage(page)">{{ page }}</span>
        </li>
        <li class="page-item" v-if="!showMorePage(totalPages)">
          <span class="page-link cursor-not-allowed text-gray-400 border-y-0"
            >...</span
          >
        </li>
        <li class="page-item cursor-pointer">
          <span
            class="page-link"
            :class="{
              'text-gray-400 bg-gray-100 cursor-not-allowed hover:text-gray-400':
                isLastPage,
            }"
            :disabled="isLastPage"
            @click="nextPage"
            >&gt;</span
          >
        </li>
        <li class="page-item cursor-pointer">
          <span
            class="page-link"
            :class="{
              'text-gray-400 bg-gray-100 cursor-not-allowed hover:text-gray-400':
                isLastPage,
            }"
            :disabled="isLastPage"
            @click="goToLastPage"
            >最末頁</span
          >
        </li>
      </ul>
      <div>
        每頁<input
          class="border w-16 mx-2"
          type="number"
          v-model.lazy="limit"
        />筆
      </div>
    </div>
  </div>
</template>
