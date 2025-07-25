<script setup>
import PageNavigation from '@/components/common/PageNavigation.vue';
import axios from 'axios';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import NoticeModal from '../NoticeModal/NoticeModal.vue';
import { useModalState } from '@/stores/modalState';

const route = useRoute();
const noticeList = ref([]);
const noticeCount = ref(0);
const modalState = useModalState();
const detailId = ref(0);

const noticeSearch = (cPage = 1) => {
  const param = new URLSearchParams(route.query);
  param.append('currentPage', cPage);
  param.append('pageSize', 5);

  axios.post('/api/support/noticeListBody.do', param).then((res) => {
    noticeList.value = res.data.list;
    noticeCount.value = res.data.count;
  });
};

const noticeDetail = (id) => {
  modalState.$patch({ isOpen: true, type: 'notice' });
  detailId.value = id;
};

watch(
  () => route.query,
  () => {
    noticeSearch();
  },
);

onMounted(() => {
  noticeSearch();
});
</script>

<template>
  <div class="notice-main-container">
    <table class="notice-table">
      <thead class="notice-table-header">
        <tr>
          <th>공지번호</th>
          <th>공지 제목</th>
          <th>공지 날짜</th>
          <th>작성자</th>
        </tr>
      </thead>
      <tbody>
        <template v-if="noticeCount > 0">
          <tr v-for="notice in noticeList" :key="notice.noticeId" class="notice-table-row">
            <td class="notice-cell">{{ notice.noticeId }}</td>
            <td
              class="notice-cell cursor-pointer hover:underline"
              @click="noticeDetail(notice.noticeId)"
            >
              {{ notice.noticeTitle }}
            </td>
            <td class="notice-cell">{{ notice.regDate.substr(0, 10) }}</td>
            <td class="notice-cell">{{ notice.loginId }}</td>
          </tr>
        </template>
        <template v-else>
          <tr>
            <td colspan="4" class="notice-empty-row">일치하는 검색 결과가 없습니다</td>
          </tr>
        </template>
      </tbody>
    </table>
    <PageNavigation :total-items="noticeCount" :items-per-page="5" :on-page-change="noticeSearch" />
  </div>
  <NoticeModal
    v-if="modalState.isOpen && modalState.type === 'notice'"
    :detail-id
    @post-success="noticeSearch()"
    @un-mounted-modal="detailId = $event"
  />
</template>

<style>
@import './styled.css';
</style>
