<script setup>
import { EventBus } from '@/api/eventBus.js';
import { onMounted, onUnmounted, ref } from 'vue';
import { apiReady, loadYouTubeIframeAPI } from "@/api/youtubeSetup";

const props = defineProps({
    videoData: Object
})

const emit = defineEmits(['changeCurSentence'])

const player = ref(null);

const handleSeek = (startTime) => {
    player.value.seekTo(startTime, true);
    EventBus.emit('stop-section-play');
}

const pauseVideo = () => {
    player.value.pauseVideo();
};

let intervalId = null;

const checkTimeAndPause = (stopTime, callback) => {
    if (player.value && player.value.getCurrentTime() >= stopTime) {
        player.value.pauseVideo();
        clearInterval(intervalId);
        if (typeof callback === 'function') {
            callback(); // 구간 재생이 끝나면 콜백 호출
        }
    }
};

const sectionPlay = ({ startTime, stopTime, callback }) => {
    if (player.value) {
        player.value.seekTo(startTime, true);
        player.value.playVideo();

        clearInterval(intervalId);
        intervalId = setInterval(() => checkTimeAndPause(stopTime, callback), 250);
    }
};



onMounted(() => {
    loadYouTubeIframeAPI(); // API 로드 호출

    const createPlayer = () => {
        player.value = new YT.Player('player', {
            height: '360',
            width: '640',
            videoId: props.videoData.videoId,
            events: {
                'onStateChange': onPlayerStateChange
            }
        });
    };

    if (apiReady) { // API가 이미 준비된 경우 바로 플레이어 생성
        createPlayer();
    } else { // API 준비 이벤트를 기다린 후 플레이어 생성
        document.addEventListener('YouTubeAPIReady', createPlayer, { once: true });
    }
    EventBus.on('seek-to', handleSeek);
    EventBus.on('pause-video', pauseVideo);
    EventBus.on('section-play', sectionPlay);
});

onUnmounted (() => {
    EventBus.off('seek-to', handleSeek);
    EventBus.off('pause-video', pauseVideo);
    EventBus.off('section-play', sectionPlay);
    clearInterval(intervalId);
    clearInterval(interval);
    player.value = null;
})

let currentSentenceOrder;
let interval = null;

function onPlayerStateChange(event) {
    if (event.data == YT.PlayerState.PLAYING) {
        const checkTime = () => {
            if (!player.value) {
                clearInterval(interval);
                return;
            }
            const currentTime = player.value.getCurrentTime();

            const nextSentence = props.videoData.sentenceList.find(sentence => {
                return currentTime < sentence.startTime;
            });
            
            let checkedCurrentSentenceOrder;

            if (nextSentence) {
                checkedCurrentSentenceOrder = (nextSentence.order - 1) === 0 ? 1 : nextSentence.order - 1;
            } else {
                checkedCurrentSentenceOrder = props.videoData.sentenceList[props.videoData.sentenceList.length - 1].order;
            }
            if (currentSentenceOrder !== checkedCurrentSentenceOrder) {
                currentSentenceOrder = checkedCurrentSentenceOrder;
                emit('changeCurSentence', checkedCurrentSentenceOrder);
            }
        };
        
        clearInterval(interval);
        interval = setInterval(checkTime, 250);

        event.target.addEventListener('onStateChange', function(e) {
            if (e.data != YT.PlayerState.PLAYING) {
                clearInterval(interval);
            }
        });
    } else if (event.data == YT.PlayerState.PAUSED) {
        EventBus.emit('stop-section-play');
    }
}

</script>

<template>
    <div id=player-container>
        <div id="player"></div>
    </div>
</template>

<style scope>
#player-container {
    position: relative;
    width: 90%;
    padding-top: 50.625%;
    margin-bottom: 15px;
    overflow: hidden;
}
#player {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    border-radius: 10px;
}
</style>