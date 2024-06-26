<script setup>
import BaseButton from "@/components/BaseButton.vue";
import { onMounted, ref } from "vue";
import router from "@/router";
import { getBoardList } from "@/api/board";
import { OK } from "@/constant/status";
import BasePaginate from "@/components/BasePaginate.vue";

const boards = ref([]);
const currentPage = ref(1);

onMounted(async () => {
  const { status, data } = await getBoardList();
  if (status === OK) boards.value = data;
});

const handleClickPageNum = pageNum => {
  currentPage.value = pageNum;
};
</script>

<template>
  <main class="page">
    <div class="inner">
      <div class="search">
        <input type="text" placeholder="검색하고 싶은 제목 또는 내용을 입력하세요." />
        <BaseButton text="검색" :width="15" :style="'primary'" />
      </div>
      <div
        style="display: flex; justify-content: space-between; margin-bottom: 1rem; align-items: end"
      >
        <p style="margin: 0">
          총 <span class="primary bold">{{ boards.length }}</span
          >건
        </p>
        <RouterLink to="/board/regist" style="width: 15%; height: 100%">
          <BaseButton :style="'white'" :width="100" text="글쓰기" />
        </RouterLink>
      </div>
      <table>
        <thead>
          <th>ID</th>
          <th>제목</th>
          <th>작성자</th>
          <th>작성 날짜</th>
        </thead>
        <tbody>
          <tr
            v-for="board in boards"
            :key="board.boardId"
            @click="
              () => {
                router.push('/board/' + board.boardId);
              }
            "
          >
            <td>{{ board.boardId }}</td>
            <td>{{ board.title }}</td>
            <td>{{ board.writerId }}</td>
            <td>{{ board.writingTime }}</td>
          </tr>
        </tbody>
      </table>
      <div class="paging">
        <BasePaginate
          v-if="boards.length > 0"
          :total-items="boards.length"
          :on-click-handler="handleClickPageNum"
          :page="currentPage"
        />
      </div>
    </div>
  </main>
</template>

<style scoped>
.search {
  display: flex;
  justify-content: space-between;
  margin-top: 1.5rem;
  margin-bottom: 1.5rem;
}

.search input {
  width: 75%;
}

table {
  width: 100%;
  text-align: center;
  border-collapse: collapse;
  font-size: 1.2rem;
}

thead {
  border-top: 0.5px solid #aaa;
  border-bottom: 0.5px solid #aaa;
}

th:first-child {
  border-left: none;
}

th:last-child {
  border-right: none;
}

th {
  border-left: 0.3px solid #aaa;
  border-right: 0.3px solid #aaa;
  padding: 1rem;
}

tr {
  border-top: 0.5px solid #aaa;
  border-bottom: 0.5px solid #aaa;
  cursor: pointer;
}

tr:hover {
  background-color: #efefef;
}

td {
  padding: 1rem;
}

.paging {
  text-align: center;
  margin-top: 0.5rem;
  margin-bottom: 0.5rem;
}

.paging span {
  font-size: 1.2rem;
  margin-left: 0.5rem;
  margin-right: 0.5rem;
}
</style>
