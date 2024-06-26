<script setup>
import { getCityList, getDistrictList, getSearchResult } from "@/api/data";
import BaseButton from "@/components/BaseButton.vue";
import BaseSelect from "@/components/BaseSelect.vue";
import BaseCard from "@/components/BaseCard.vue";
import BasePaginate from "@/components/BasePaginate.vue";
import { useNotification } from "@kyvg/vue3-notification";
import { OK } from "@/constant/status";
import { mainCategory, subCategory } from "@/util/types";
import { ChevronDownIcon, ChevronUpIcon } from "@heroicons/vue/24/solid";
import { ref, watch } from "vue";
import router from "@/router";

const { notify } = useNotification();

const cities = ref([]);
const districts = ref([]);
const keyword = ref("");
const areaCode = ref(0);
const sigunguCode = ref(0);
const isSectionOpen = ref(true);
const isLoading = ref(false);

const currentPage = ref(1);
const searchResults = ref([]);

const handleClickCitySelect = async () => {
  const { data, status } = await getCityList();
  if (status === OK) cities.value = data.response.body.items.item;
};

const handleSelectCity = async code => {
  const { data, status } = await getDistrictList(code);
  if (status === OK) {
    districts.value = data.response.body.items.item;
    areaCode.value = code;
  }
};

const handleClickDistrictSelect = () => {
  if (areaCode.value === 0)
    notify({
      type: "warn",
      text: "시도를 먼저 선택해 주세요!",
    });
};

const handleSelectDistrict = code => {
  sigunguCode.value = code;
};

const handleClickSearch = async () => {
  isLoading.value = true;
  if (keyword.value === "") {
    notify({ type: "error", text: "키워드를 입력해 주세요!" });
    return;
  }
  const { data, status } = await getSearchResult(
    keyword.value,
    areaCode.value,
    sigunguCode.value,
    currentPage.value,
  );

  if (status === OK) {
    if (data.response) isLoading.value = false;
    if (!isLoading.value) searchResults.value = await data.response.body.items.item;
  }
};

watch(isLoading, () => {
  console.log(isLoading.value);
});

const openSection = () => (isSectionOpen.value = !isSectionOpen.value);

const handlerClickPageNum = async page => {
  currentPage.value = page;
  const { data, status } = await getSearchResult(
    areaCode.value,
    sigunguCode.value,
    currentPage.value,
  );
  if (status === OK) {
    searchResults.value = data.response.body.items.item;
  }
};
</script>

<template>
  <main class="page">
    <div class="inner">
      <section class="search">
        <input type="text" placeholder="여행지 키워드 검색 (필수)" v-model="keyword" />
        <p class="gray">여행하고 싶은 지역을 골라주세요.</p>
        <div class="selects">
          <BaseSelect
            defaultSelected="시도 선택"
            :options="cities"
            @handleClickSelect="handleClickCitySelect"
            @handleSelectOption="handleSelectCity"
          />
          <BaseSelect
            defaultSelected="시군구 선택"
            :options="districts"
            @handleClickSelect="handleClickDistrictSelect"
            @handleSelectOption="handleSelectDistrict"
          />
          <BaseButton text="검색" :width="15" :style="'primary'" :on-click="handleClickSearch" />
        </div>
      </section>

      <section class="barrierfree-info">
        <div class="info-title bg-point white" @click="openSection">
          <div>
            <span class="bold" style="font-size: 1.3rem">무장애 정보</span>
            <span style="margin: 0.5rem">다음과 같은 무장애 정보가 제공 됩니다 :)</span>
          </div>
          <ChevronUpIcon v-if="isSectionOpen" class="white" />
          <ChevronDownIcon v-else class="white" />
        </div>
        <div v-if="isSectionOpen" class="info-category">
          <div v-for="category in mainCategory" :key="category" class="category-section">
            <span class="category-title">{{ category }}</span>
            <div class="sub-category">
              <div v-for="sub in subCategory[category]" :key="sub" class="checkbox-container">
                <div class="bg-white round-box">
                  <span class="black">
                    {{ sub }}
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <div v-show="isLoading" style="display: flex; align-items: center; justify-content: center">
        <img src="../assets/img/loading.gif" alt="로딩중" style="margin: 5rem 0; width: 10%" />
      </div>
      <section class="search-result">
        <p v-if="!searchResults">검색 결과가 없습니다😥</p>
        <div v-else>
          <p>
            총 <span class="primary bold">{{ searchResults.length }}</span
            >건
          </p>
          <div
            class="cards"
            :style="searchResults.length < 3 ? '' : 'justify-content: space-between;'"
          >
            <BaseCard
              v-for="result in searchResults"
              :key="result.contentid"
              :id="result.contentid"
              :imageUrl="result.firstimage"
              :title="result.title"
              :addr="result.addr1.split(' ').slice(0, 2).join(' ')"
              @click="() => router.push('/detail/' + result.contentid)"
            />
          </div>
        </div>
      </section>

      <BasePaginate
        v-if="searchResults"
        :total-items="searchResults.length"
        :on-click-handler="handlerClickPageNum"
        :page="currentPage"
      />
    </div>
  </main>
</template>

<style scoped>
.inner {
  margin-bottom: 10%;
}
section {
  display: flex;
  flex-direction: column;
  margin: 1rem 0;
  width: 100%;
}

.search {
  background: #f8f9fa;
  border-radius: 0.5rem;
  box-shadow: 0 0.25rem 0.375rem rgba(0, 0, 0, 0.1);
}

.search input[type="text"] {
  border-radius: 0.25rem;
  margin: 1rem;
  border: 1px solid #7aa2ce;
  padding: 0.8rem 1rem;
}

.search input[type="text"]:focus,
.search input[type="text"]:hover {
  border-color: #7aa2ce;
  box-shadow: 0 0 5px #7aa2ce;
  outline: none;
}

.search p {
  font-size: 1rem;
  margin: 0 1rem;
}

.selects {
  margin: 1rem;
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
}

.barrierfree-info {
  width: 100%;
  background: #ffffff;
  box-shadow: 0 0.25rem 0.375rem rgba(0, 0, 0, 0.1);
  border-radius: 0.5rem;
}

.barrierfree-info svg {
  width: 1.5rem;
  height: 1.5rem;
}

.info-title {
  border-radius: 0.5rem 0.5rem 0 0;
  padding: 1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
}

.info-category {
  padding: 1rem;
  border: 0.0625rem solid #7aa2ce;
}

.category-section {
  display: flex;
  justify-content: space-between;
  margin: 1rem 0;
}

.category-title {
  font-size: 1.1rem;
  font-weight: 700;
  width: 20%;
}

.sub-category {
  display: flex;
  flex-wrap: wrap;
  gap: 0.625rem;
  width: 80%;
}

.checkbox-container {
  display: flex;
  align-items: center;
}

svg {
  width: 1.5rem;
  height: 1.5rem;
}

.round-box {
  display: flex;
  align-items: center;
  justify-content: center;
  border: 0.5px solid #7aa2ce;
  border-radius: 15px;
  transition: background-color 0.3s;
  padding: 0.5rem 1rem;
}

.search-result p {
  font-size: 1rem;
  color: #333;
}

.cards {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}
</style>
