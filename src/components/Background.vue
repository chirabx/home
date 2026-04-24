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
      @canplay="mediaLoadComplete"
      @error.once="mediaLoadError"
      @animationend="imgAnimationEnd"
    ></video>

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
const bgType = ref("0"); // 新增：用于在本地追踪当前应该渲染视频还是图片
const imgTimeout = ref(null);
const emit = defineEmits(["loadComplete"]);

// 壁纸随机数
const bgRandom = Math.floor(Math.random() * 10 + 1);

// 更换壁纸链接
const changeBg = (type) => {
  if (type == 0) {
    bgType.value = "0";
    bgUrl.value = `/videos/background.mp4`;
  } else {
    bgType.value = "1"; // 只要不是0，就渲染 img 标签
    if (type == 1) {
      bgUrl.value = "https://api.dujin.org/bing/1920.php";
    } else if (type == 2) {
      bgUrl.value = "https://api.aixiaowai.cn/gqapi/gqapi.php";
    } else if (type == 3) {
      bgUrl.value = "https://api.aixiaowai.cn/api/api.php";
    }
  }
};

// 媒体（视频/图片）加载完成
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
  // 失败时，必须将标签状态切换回视频，再赋视频链接
  bgType.value = "0";
  bgUrl.value = `/videos/background.mp4`;
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
/* CSS 样式无需修改，现有的 .bg 样式对 video 和 img 均完美适用 */
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

  .gray {
    opacity: 1;
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-image: radial-gradient(rgba(0, 0, 0, 0) 0, rgba(0, 0, 0, 0.5) 100%),
      radial-gradient(rgba(0, 0, 0, 0) 33%, rgba(0, 0, 0, 0.3) 166%);

    transition: 1.5s;
    &.hidden {
      opacity: 0;
      transition: 1.5s;
    }
  }

  .down {
    font-size: 16px;
    color: white;
    position: absolute;
    bottom: 30px;
    left: 0;
    right: 0;
    margin: 0 auto;
    display: block;
    padding: 20px 26px;
    border-radius: 8px;
    background-color: #00000030;
    width: 120px;
    height: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
    &:hover {
      transform: scale(1.05);
      background-color: #00000060;
    }
    &:active {
      transform: scale(1);
    }
  }
}

@keyframes fade-blur-in {
  to {
    filter: blur(0px) brightness(1);
  }
}
</style>