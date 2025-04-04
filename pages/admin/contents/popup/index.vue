<template>
  <div class="wrap">
    <!-- MAIN -->
    <div class="main-wrap">
      <section class="board">
        <div class="board__header">
          <h1>팝업 Pop-up</h1>
          <p>전체 게시물수 {{ totalCount }}</p>
        </div>
        <div class="board__container">
          <div class="board__list">
            <div class="board__head">
              <div class="board__no">번호</div>
              <div class="board__title">제목</div>
              <div class="board__writer">작성자</div>
              <div class="board__regDate">작성일</div>
              <div class="board__viewCnt">조회수</div>
              <div class="board__management">게시물관리</div>
            </div>
            <div class="board__body">
              <div class="board__content" v-for="popup in popups" :key="popup.id">
                <div class="board__no">{{ popup.id }}</div>
                <div class="board__title">
                  <RouterLink :to="`/admin/contents/popup/${popup.id}`">
                    {{ popup.title }}
                  </RouterLink>
                </div>
                <div class="board__writer">{{ popup.writer }}</div>
                <div class="board__regDate">{{ formatDate(popup.writeDate) }}</div>
                <div class="board__viewCnt">{{ popup.viewCount }}</div>
                <div class="board__management">
                  <button 
                    class="hiddenBtn"
                    :class="{ 'activeBtn' : !popup.isPrivate }"
                    @click="hiddenHandler(popup.id)">
                      {{ popup.isPrivate ? '공개' : '비공개' }}
                  </button>
                  <button class="modifyBtn" @click="goToEditPage(popup.id)">수정</button>
                  <button class="deleteBtn" @click="deleteHandler(popup.id)">삭제</button>
                </div>
              </div>
            </div> <!-- END BOARD BODY -->
          </div> <!-- END BOARD LIST-->
        </div>
        <!-- END BOARD CONTAINER -->
         
        <div class="pagination-registration-container">
          <Pagination 
          :current-page="currentPage"
          :total-pages="totalPages"
          :has-prev-page="hasPrevPage"
          :has-next-page="hasNextPage"
          :visible-pages="pages"
          @page-change="changePage"
          />

          <div class="registration">
            <RouterLink to="/admin/contents/popup/new">
              <button type="button">등록</button>
            </RouterLink>
          </div>
        </div>
      </section>
    </div> <!-- END MAIN WRAP -->

    <!-- SEARCH BAR -->
    <section class="search">
      <div class="search__bar">
        <div class="search__dropdown">
          <div id="drop-text" class="search__text" @click="toggleDropdown">
            <span id="span">{{ selectedCategory }}</span>
            <i id="icon" class='bx bx-chevron-down'
              :style="{ transform: isOpen ? 'rotate(-180deg)' : 'rotate(0deg)' }"></i>
          </div>
          <ul id="drop-list" class="search__list" :class="{ show: isOpen }">
            <li class="search__item" v-for="item in categories" :key="item" @click="selectCategory(item)">
              {{ item }}
            </li>
          </ul>
        </div>
        <div class="search__box">
          <input 
            type="text" 
            id="search-input" 
            :placeholder="inputPlaceholder" 
            v-model="searchQuery" 
            @keyup.enter="performSearch" 
          />
          <i class='bx bx-search' @click="performSearch"></i>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import Pagination from '~/components/admin/Pagination.vue';
import { usePaginationStore } from '~/stores/pagination';

// Pinia 스토어 초기화
const paginationStore = usePaginationStore();

// 상태 관리를 위한 ref 선언
const popups = ref([]);
const totalCount = ref(0);

// 페이지네이션 상태 관리
const totalPages = ref(0);
const currentPage = ref(1);
const hasPrevPage = ref(false);
const hasNextPage = ref(false);
const pages = ref([]);

// 검색 기능
const categories = ['통합검색', '제목 + 내용', '제목', '내용', '작성자']; // 검색 카테고리
const selectedCategory = ref('통합검색');
const inputPlaceholder = ref('검색어를 입력해주세요');
const searchQuery = ref('');
const isOpen = ref(false); // dropdown 상태

const categoryMapping = {
  '통합검색': 'ALL',
  '제목': 'TITLE',
  '내용': 'CONTENT',
  '작성자': 'WRITER'
};

// 라우터 및 라우트 설정
const router = useRouter();
const route = useRoute();
const config = useRuntimeConfig();
const apiBase = config.public.apiBase;

// 팝업 데이터 가져오기
const fetchPopups = async (page = 1) => {
  try {
    const pageNumber = page;
    console.log("fetching page: ", pageNumber);

    const url = `${apiBase || 'http://localhost:8081/api/v1'}/admin/popup`;
    const params = {
      p: pageNumber,
      ...(searchQuery.value ? { k: searchQuery.value } : {}),
      ...(categoryMapping[selectedCategory.value] && categoryMapping[selectedCategory.value] !== 'ALL'
        ? { s: categoryMapping[selectedCategory.value] }
        : {})
    };

    // url 쿼리 문자열 생성
    const queryString = Object.entries(params)
      .map(([key, value]) => `${key}=${encodeURIComponent(value)}`)
      .join('&');

    // api 요청 실행
    const fullUrl = `${url}?${queryString}`;
    console.log("REQUEST SENT TO: ", fullUrl);

    const responseData = await $fetch(fullUrl);
    console.log("SERVER RESPONSE DATA: ", responseData);

    // 응답데이터 처리
    // popups.value = responseData.popupList || [];
    popups.value = [...(responseData.popupList || [])];
    totalCount.value = responseData.totalCount || 0;
    totalPages.value = responseData.totalPages || 0;
    hasPrevPage.value = responseData.hasPrev || false;
    hasNextPage.value = responseData.hasNext || false;

    // 페이지 번호 계산
    const rangeStart = Math.max(1, page - 2);
    const rangeEnd = Math.min(totalPages.value, page + 2);
    pages.value = Array.from({ length: rangeEnd - rangeStart + 1}, (_, i) => rangeStart + i);

    currentPage.value = page;

    console.log("totalPages:", totalPages.value);
    console.log("hasPrevPage:", hasPrevPage.value);
    console.log("hasNextPage:", hasNextPage.value);
    console.log("pages:", pages.value);

  } catch(error) {
    console.error("ERROR FETCHING POPUP LIST: ", error);
  }
};

// 페이지 이동 핸들러
const changePage = async (page) => {
  if (page < 1 || page > totalPages.value) return;

  // URL 쿼리 파라미터 업데이트
  router.push({
    query: {
      page,
      ...(searchQuery.value ? { k: searchQuery.value } : {}),
      ...(categoryMapping[selectedCategory.value] && categoryMapping[selectedCategory.value] !== 'ALL'
        ? { s: categoryMapping[selectedCategory.value] }
        : {})
    }
  });

  await fetchPopups(page);
};

// 수정 버튼 이벤트
const goToEditPage = (id) => {
  router.push(`/admin/contents/popup/${id}/edit`); // 하드코딩
};

// 드롭다운 토글
const toggleDropdown = () => {
  isOpen.value = !isOpen.value;
};

// 카테고리 선택 함수
const selectCategory = (category) => {
  selectedCategory.value = category;
  updatePlaceholder(category);
  isOpen.value = false; // 선택 후 드롭다운 클로즈
};

// placeholder 업데이트
const updatePlaceholder = (category) => {
  if (category === '통합검색') {
    inputPlaceholder.value = '검색어를 입력해주세요';
  } else if (category === '작성자') {
    inputPlaceholder.value = `검색할 ${category}를 입력해주세요`;
  } else {
    inputPlaceholder.value = `검색할 ${category}을 입력해주세요`;
  }
};

// 검색 수행
const performSearch = async () => {
  currentPage.value = 1;

  // url 쿼리 파라미터 업데이트
  router.push({
    query: {
      page: 1,
      ...(searchQuery.value ? { k: searchQuery.value } : {}),
      ...(categoryMapping[selectedCategory.value] && categoryMapping[selectedCategory.value] !== 'ALL'
        ? { s: categoryMapping[selectedCategory.value] }
        : {})
    }
  });

  await fetchPopups();
}

// 클릭 외부 영역 처리
const handleClickOutside = (e) => {
  if (!e.target.closest('.search')) {
    isOpen.value = false;
  }
};

// 날짜 포맷팅 함수 추가
const formatDate = (dateString) => {
  if (!dateString) return '';

  const date = new Date(dateString);
  return date.toLocaleDateString('ko-KR', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit'
  }).replace(/\. /g, '.').replace(/\.$/, '');
};

// URL 쿼리 파라미터 변경 감지
const watchRouteQuery = () => {
  const newPage = parseInt(route.query.page) || 1;
  const newSearchQuery = route.query.k || '';
  const newSearchType = route.query.s || '';

  currentPage.value = newPage;
  searchQuery.value = newSearchQuery;

  if(newSearchType) {
    const category = Object.entries(categoryMapping)
      .find(([key, value]) => value === newSearchType)?.[0];

    if(category) {
      selectedCategory.value = category;
      updatePlaceholder(category);
    }
  }

    fetchPopups(newPage);
};

// 단일 onMounted 훅으로 통합
onMounted(async () => {
  await watchRouteQuery(); // URL 쿼리 파라미터 기반으로 초기 데이터 로드

  window.addEventListener('click', handleClickOutside);

  // url 변경 감지 
  watch(() => route.query, watchRouteQuery, { deep: true });
});

// beforeRouteUpdate 가드 추가
const beforeRouteUpdate = async(to, from, next) => {
  await watchRouteQuery();
  next();
};

// defineExpose를 사용해 beforeRouteUpdate를 외부에 노출
defineExpose({ beforeRouteUpdate });

onUnmounted(() => {
  window.removeEventListener('click', handleClickOutside);
});

// 삭제 처리
const deleteHandler = async (id) => {
  // 사용자 확인 요청
  if(!confirm('해당 게시물을 삭제하시겠습니까?')) {
    return; // 사용자가 취소한 경우
  }

  try {
    // DELETE 요청 보내기
    const url = `${apiBase || 'http://localhost:8081/api/v1'}/admin/popup/${id}`;
    const response = await fetch(url, {
      method: 'DELETE',
      headers: {
        'Content-Type': 'application/json'
      }
    });

    if(!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || `서버 응답 오류: ${response.status}`);
    }

    // 삭제 성공 시 목록 새로고침
    alert('게시물이 삭제되었습니다.');

    // 현재 페이지에 항목이 하나만 있고, 첫 페이지가 아닌 경우 이전 페이지로 이동
    if(popups.value.length === 1 && currentPage.value > 1) {
      await changePage(currentPage.value - 1);
    } else {
      await fetchPopups(currentPage.value);
    }
  }catch(error) {
    console.error('팝업 삭제 중 오류 발생: ', error);
    alert(`팝업 삭제 중 오류 발생: ${error.message}`);
  }
};

// 비공개 처리
const hiddenHandler = async (id) => {
  try {
    if(!confirm('해당 게시물의 공개 상태를 변경하시겠습니까?')) {
      return;
    }

    const url = `${apiBase || 'http://localhost:8081/api/v1'}/admin/popup/${id}/change-status`;

    const response = await fetch(url, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json'
      }
    });

    if(!response.ok) {
      throw new Error(`서버 응답 오류: ${response.status}`);
    }

    // 성공 메시지 표시
    alert('게시물의 공개 상태가 변경되었습니다.');

    // 목록 새로고침
    await fetchPopups(currentPage.value);
    await nextTick();

  } catch(error) {
    console.error('팝업 공개 상태 변경 중 오류 발생: ', error);
    alert(`팝업 공개 상태 변경 중 오류 발생: ${error.message}`);
  }
};


</script>

<style lang="css" scoped>
@import url('/public/css/admin/contents/popup/index.css');
</style>