<template>
  <div class="wrap" v-if="exhibition">
    <!-- section 1 -->
    <div class="event">
      <EventHeader
        :title="exhibition.title"
        @notification-click="handleNotification"
        @bookmark-click="handleBookmark"
      />

      <!-- 맨 위로 버튼 -->
      <button id="go-top" class="hidden">
        <i class="fas fa-arrow-up"></i> TOP
      </button>

      <!-- section 2 -->
      <main>
        <EventInfo 
          :image-url="`http://localhost:8081/api/v1/uploads/images/exhibition/${exhibition.saveImageName}`"
          :image-alt="`${exhibition.title} 포스터`"
          :venue="exhibition.venue"
          :running-time="exhibition.runningTime"
          :start-date="exhibition.startDate"
          :end-date="exhibition.endDate"
          :age-restriction="exhibition.ageRestriction"
          :ticket-prices="exhibition.ticketPrices"
          :ticket-offices="exhibition.ticketOffices"
          :address="exhibition.address"
        />

        <!-- section 3 -->
        <EventIntro 
          title="전시회"
          :content="exhibition.content"
          :youtube-url="exhibition.youtubeUrl"
          :images="exhibition.exhibitionImages"
        />

        <!-- section 4 -->
        <!-- <EventLocation 
          :address="exhibition.address"
          :venue-name="exhibition.venue"
        /> -->
      </main>
    </div>
  </div>
</template>

<script setup>
import { ref, watchEffect } from 'vue';
import { useRoute } from 'vue-router';
import EventHeader from '~/components/event/EventHeader.vue';
import EventInfo from '~/components/event/EventInfo.vue';
import EventIntro from '~/components/event/EventIntro.vue';
import EventLocation from '~/components/event/EventLocation.vue';

const route = useRoute();
const exhibitionId = route.params.id;

const config = useRuntimeConfig();
const apiBase = config.public.apiBase;

// 상대 관리 위한 ref 선언
const exhibition = ref(null);

// 데이터 가져오기
watchEffect(async () => {
  const { data } = await useFetch(`/admin/exhibition/${exhibitionId}`, {
    baseURL: apiBase,
    key: `exhibition-${exhibitionId}`,
  });

  if(data.value) {
    exhibition.value = data.value;
  }
});

</script>

<style scoped>
@import url('/public/css/components/event.css');
</style>ㄴ