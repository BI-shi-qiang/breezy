<template>
  <div class="flip-card" @mouseenter="isFlipped = true" @mouseleave="isFlipped = false">
    <div class="flip-inner" :class="{ flipped: isFlipped }">
      <!-- 正面图片 -->
      <div class="flip-front">
        <img :src="imgSrc" alt="" class="front-img" />
      </div>

      <!-- 背面文字 -->
      <div class="flip-back">
        <div class="back-content">
          <h3 class="title">{{ title }}</h3>
          <p class="text">{{ text }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
const isFlipped = ref(false)

defineProps({
  imgSrc: String,
  title: String,
  text: String,
})
</script>

<style scoped>
.flip-card {
  width: 280px;
  height: 380px;
  perspective: 1000px;
  cursor: pointer;
}
.flip-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.7s ease;
  transform-style: preserve-3d;
}
.flipped {
  transform: rotateY(180deg);
}
.flip-front,
.flip-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
}
.flip-front {
  background: #f1f1f1;
}
.front-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.flip-back {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  transform: rotateY(180deg);
  padding: 20px;
  text-align: center;
}
.back-content {
  padding: 0 10px;
}
.title {
  font-size: 22px;
  margin-bottom: 15px;
}
.text {
  font-size: 15px;
  line-height: 1.6;
}
</style>