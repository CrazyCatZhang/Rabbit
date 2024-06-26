<script setup lang="ts">
import { getCategoryTopAPI } from '@/services/category'
import { getHomeBannerAPI } from '@/services/home'
import { onLoad } from '@dcloudio/uni-app'
import type { CategoryTopItem } from 'category'
import type { BannerItem } from 'home'
import { computed, ref } from 'vue'
import PageSkeleton from './components/PageSkeleton.vue'

const bannerList = ref<BannerItem[]>([])
const getBannerList = () => {
  getHomeBannerAPI().then((res) => {
    bannerList.value = res.result
  })
}

const categoryList = ref<CategoryTopItem[]>([])
const getCategoryList = () => {
  getCategoryTopAPI().then((res) => {
    categoryList.value = res.result
  })
}

const activeIndex = ref(0)

const subCategoryList = computed(() => {
  return categoryList.value[activeIndex.value]?.children || []
})

const isFinished = ref(false)

onLoad(() => {
  Promise.all([getBannerList(), getCategoryList()]).then(() => {
    isFinished.value = true
  })
})
</script>

<template>
  <view class="viewport" v-if="isFinished">
    <!-- 搜索框 -->
    <view class="search">
      <view class="input">
        <text class="icon-search">女靴</text>
      </view>
    </view>
    <!-- 分类 -->
    <view class="categories">
      <!-- 左侧：一级分类 -->
      <scroll-view class="primary" scroll-y>
        <view
          v-for="(item, index) in categoryList"
          :key="item.id"
          class="item"
          :class="{ active: index === activeIndex }"
          @tap="activeIndex = index"
        >
          <text class="name"> {{ item.name }} </text>
        </view>
      </scroll-view>
      <!-- 右侧：二级分类 -->
      <scroll-view class="secondary" scroll-y>
        <!-- 焦点图 -->
        <XtxSwiper class="banner" :list="bannerList" />
        <!-- 内容区域 -->
        <view class="panel" v-for="item in subCategoryList" :key="item.id">
          <view class="title">
            <text class="name">{{ item.name }}</text>
            <navigator class="more" hover-class="none">全部</navigator>
          </view>
          <view class="section">
            <navigator
              v-for="goods in item.goods"
              :key="goods.id"
              class="goods"
              hover-class="none"
              :url="`/pages/goods/goods?id=${goods.id}`"
            >
              <image class="image" :src="goods.picture"></image>
              <view class="name ellipsis">{{ goods.name }}</view>
              <view class="price">
                <text class="symbol">¥</text>
                <text class="number">{{ goods.price }}</text>
              </view>
            </navigator>
          </view>
        </view>
      </scroll-view>
    </view>
  </view>
  <PageSkeleton v-else />
</template>

<style lang="scss">
@import './styles/category.scss';
</style>
