<template>
  <div :class="store.backgroundShow ? 'cover show' : 'cover'">
    <video
      v-if="bgType === '0'"
      v-show="store.imgLoadStatus"
      :src="bgUrl"
      class="bg"
      autoplay
      loop
      muted
      playsinline
      webkit-playsinline="true"
      x5-playsinline="true"
      x5-video-player-type="h5"
      x5-video-player-fullscreen="true"
      @playing="videoPlaying = true"
      @error.once="mediaLoadError"
    ></video>

    <img
      v-if="bgType === '0'"
      v-show="store.imgLoadStatus"
      :src="posterUrl"
      class="bg"
      :style="{ opacity: videoPlaying ? 0 : 1, transition: 'opacity 0.8s ease' }"
      @load="mediaLoadComplete"
      @error.once="mediaLoadError"
      @animationend="imgAnimationEnd"
    />

    <img
      v-else
      v-show="store.imgLoadStatus"
      :src="bgUrl"
      class="bg"
      @load="mediaLoadComplete"
      @error.once="mediaLoadError"
      @animationend="imgAnimationEnd"
    />

    <div :class="store.backgroundShow ? 'gray hidden' : 'gray'" />

    <Transition name="fade" mode="out-in">
      <a
        v-if="store.backgroundShow && store.coverType != '3'"
        class="down"
        :href="bgUrl"
        target="_blank"
      >
        下载壁纸
      </a>
    </Transition>
  </div>
</template>

<script setup>
import { mainStore } from "@/store";
import { Error } from "@icon-park/vue-next";
import { ref, watch, onMounted, onBeforeUnmount, h } from "vue";
import { ElMessage } from "element-plus";

const store = mainStore();
const bgUrl = ref(null);
const posterUrl = ref(null); // 新增：视频预览图链接
const videoPlaying = ref(false); // 新增：视频是否已经开始播放

const bgType = ref("0");

const imgTimeout = ref(null);
const emit = defineEmits(["loadComplete"]);

// 更换壁纸链接
const changeBg = (type) => {
  if (type == 0) {
    bgType.value = "0";
    bgUrl.value = `/videos/background.mp4`;
    // 【注意】你需要准备一张视频第一帧的图片，放到 public/videos/ 目录下
    posterUrl.value = `/videos/background-poster.jpg`;
    videoPlaying.value = false; // 切换时重置播放状态
  } else {
    bgType.value = "1";
    if (type == 1) {
      bgUrl.value = "https://api.dujin.org/bing/1920.php";
    } else if (type == 2) {
      bgUrl.value = "https://api.aixiaowai.cn/gqapi/gqapi.php";
    } else if (type == 3) {
      bgUrl.value = "https://api.aixiaowai.cn/api/api.php";
    }
  }
};

// 媒体（图片/预览图）加载完成
const mediaLoadComplete = () => {
  imgTimeout.value = setTimeout(() => {
    store.setImgLoadStatus(true);
  }, Math.floor(Math.random() * (600 - 300 + 1)) + 300);
};

// 动画完成
const imgAnimationEnd = () => {
  console.log("背景加载且动画完成");
  emit("loadComplete");
};

// 加载失败
const mediaLoadError = () => {
  console.error("背景加载失败：", bgUrl.value);
  ElMessage({
    message: "背景加载失败，已临时切换回默认",
    icon: h(Error, {
      theme: "filled",
      fill: "#efefef",
    }),
  });
  bgType.value = "0";
  bgUrl.value = `/videos/background.mp4`;
  posterUrl.value = `/images/background-poster.png`;
  videoPlaying.value = false;
};

// 监听切换
watch(
  () => store.coverType,
  (value) => {
    changeBg(value);
  },
);

onMounted(() => {
  changeBg(store.coverType);
});

onBeforeUnmount(() => {
  clearTimeout(imgTimeout.value);
});
</script>

<style lang="scss" scoped>
/* 保持原样即可 */
.cover {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  transition: 0.25s;
  z-index: -1;

  &.show {
    z-index: 1;
  }

  .bg {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    backface-visibility: hidden;
    filter: blur(20px) brightness(0.3);
    transition: filter 0.3s, transform 0.3s;
    animation: fade-blur-in 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94) forwards;
    animation-delay: 0.45s;
  }
  /* 下方省略其他原有样式 ... */
}
</style>