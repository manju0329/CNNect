<template>
  <div id="now-video">
    <div v-if="curVideo.videoId" class="flex">
      <!-- 앞으로 가는 버튼 -->
      <div class="Carousel-btn">
        <svg class="sysmbol-btn" xmlns="http://www.w3.org/2000/svg" @click="changeVideoOrder('fore')" height="24" viewBox="0 -960 960 960" width="24" fill="#CC0000">
          <path d="M400-80 0-480l400-400 71 71-329 329 329 329-71 71Z"/>
        </svg>
      </div>
      <!-- 비디오 컨테이너 -->
      <div @click="goToStudy(curVideo.videoId)" @mouseover="handleMouseOver" @mouseleave="handleMouseLeave" id="now-video-container" class="relative rounded-xl grid grid-cols-3 w-[65vw]" :class="{ 'opacity-80': hovered, 'transition-transform': isSliding }">
        <div id="container-layer" class="rounded-xl"></div>
        <div class="bg-black" id="video-img-container" >
          <img id="now-video-img" :src="`https://img.youtube.com/vi/${curVideo.videoId}/mqdefault.jpg`" alt="Now Video Image" />
        </div>
        <div class="p-10 flex flex-col justify-between col-span-3 sm:col-span-2 mb-3" id="now-video-info">
          <div class="text-2xl font-bold font-[GmarketSansMedium]" id="video-name">
            [ Lv. {{ curVideo.videoLevel }} ] {{ curVideo.videoName }}
          </div>
          <div>
            <div class="flex">
              <div class="text-lg font-bold text-white z-10">문장 수</div>
              <div class="relative text-lg left-14" id="sentence-count">
                {{ curVideo.completedSentenceNum }} / {{ curVideo.totalSentenceNum }}
              </div>
            </div>
            <div class="flex">
              <div class="text-lg whitespace-nowrap font-bold text-white z-10">마지막 문장</div>
              <div id="last-sentence" class="relative text-lg left-5 whitespace-nowrap text-ellipsis overflow-hidden">{{ curVideo.lastSentence }}</div>
            </div>
          </div>
        </div>
      </div>
      <!-- 뒤로 가는 버튼 -->
      <div class="Carousel-btn">
        <svg xmlns="http://www.w3.org/2000/svg" @click="changeVideoOrder('back')" height="24" viewBox="0 -960 960 960" width="24" fill="#CC0000">
          <path d="m321-80-71-71 329-329-329-329 71-71 400 400L321-80Z"/>
        </svg>
      </div>
    </div>
    <!-- 콘텐츠가 없을 때 -->
    <div v-else>
      <div id="no-content" class="rounded-xl bg-gray-200 w-[70vw] h-[35vh] flex items-center justify-center overflow-hidden">
        <div class="flex items-center justify-center">
            <!-- 배경 애니메이션을 위한 요소 -->
        </div>
        <div class="text-center">
          <div class="z-20 text-lg font-bold highlight pl-2 pr-2">학습 진행중인 뉴스가 없습니다</div>
          <div class="z-20">둘러보기 탭에서 CNN 뉴스를 구경해보세요 👀</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, defineProps, onMounted } from "vue";
import { useRouter } from "vue-router";
import { getLearningVideo} from '@/api/history';

const router = useRouter();

onMounted(() => {
    getLearningVideo(
    ({ data }) => {
        if (data.data) {
            learningVideoHistory.value = data.data;
            currentOrder.value = 0;
            totalVideos.value = learningVideoHistory.value.length;
            setCurVideo(0);
            console.log("찍어 보기 : " , learningVideoHistory.value)
            console.log("찍어 보기 : " , learningVideoHistory)

        }
    },
    (error) => {
      console.log(error);
    }
    );

})

const curVideo = ref([])
const hovered = ref(false);
const isSliding = ref(false);

const handleMouseOver = () => {
  hovered.value = true;
};

const handleMouseLeave = () => {
  hovered.value = false;
}

const currentOrder = ref(0);
const totalVideos = ref(0);

const changeVideoOrder = (direction) => {
    isSliding.value = true;
    setTimeout(() => {
      isSliding.value = false;
    }, 3000);
    if (direction === 'fore') {
        if (currentOrder.value > 0) {
            currentOrder.value -= 1;
            setCurVideo(currentOrder.value);
        } else {
            currentOrder.value = totalVideos.value - 1;
            setCurVideo(currentOrder.value);
        }
    } else if (direction === 'back') {
        if (currentOrder.value < totalVideos.value - 1) {
            currentOrder.value += 1;
            setCurVideo(currentOrder.value);
        } else {
            currentOrder.value = 0;
            setCurVideo(currentOrder.value);
        }
    }
}
const learningVideoHistory = ref([])

const setCurVideo = (idx) => {
    curVideo.value = learningVideoHistory.value[idx];
}

const goToStudy = (videoId) => {
  router.push({ name: 'study', params: { videoId: videoId } });
};

</script>

<style scoped>

.highlight {
  display: inline;
  box-shadow: inset 0 -10px 0 #cc000040;
}

#now-video-img {
  width: 70rem;
  height: 100%;
  border-radius: 10px;
  object-fit: contain;
  padding: 10px;
}

#container-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: linear-gradient(-90deg, #000000, #000000);
  pointer-events: none;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.5);
}

#now-video-info {
  height: 100%;
}

#sentence-count,
#last-sentence {
  color: white;
  z-index: 2;
}

#video-img-container {
  position: relative;
  border-radius: 10px;
}

#video-img-container::after {
  content: "";
  position: absolute;
  top: 0;
  left: 30%;
  right: 0;
  bottom: 0;
  background: linear-gradient(to right, rgba(255, 255, 255, 0), rgba(0, 0, 0, 0.708));
  pointer-events: none;
  z-index: 2;
}

#video-name {
  overflow: hidden;
  text-overflow: ellipsis;
  line-height: 1.4em;
  height: 2.9em;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  color: white;
  z-index: 2;
}
</style>
