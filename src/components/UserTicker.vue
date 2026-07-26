<template>
  <div class="glass-card ticker-card">
    <div class="ticker-wrapper">
      <div v-for="(item, index) in tickerItems" :key="index" class="ticker-item">
        <span>{{ item.text }}</span>
        <span style="color: var(--success-color); font-weight: bold;">[生成完成]</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const tickerItems = ref([
  {
    "text": "用户 138****6611 完成了计算机硕士论文三级大纲与实验架构构建"
  },
  {
    "text": "用户 189****2090 生成了商科本科论文实证分析模型设计方案"
  },
  {
    "text": "用户 133****7712 完成了 SSCI 论文文献综述写作提纲"
  },
  {
    "text": "用户 156****3390 优化了人文社科论文降重与学术表达改写"
  },
  {
    "text": "用户 178****4410 生成了医学毕业论文数据统计与结论章节"
  }
]);
</script>
