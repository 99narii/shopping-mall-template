<script setup>
const props = defineProps({
  data: {
    type: Object,
    required: true
  },
  categories: {
    type: Array,
    default: () => []
  }
})

const SLIDE_THRESHOLD = 10
const sliderRef = ref(null)
const canScrollLeft = ref(false)
const canScrollRight = ref(true)

const showSlider = computed(() => props.categories.length >= SLIDE_THRESHOLD)

const updateScrollState = () => {
  if (!sliderRef.value) return
  const { scrollLeft, scrollWidth, clientWidth } = sliderRef.value

  canScrollLeft.value = scrollLeft > 1
  canScrollRight.value = scrollLeft < scrollWidth - clientWidth - 1
}

const scroll = (direction) => {
  if (!sliderRef.value) return
  const scrollAmount = sliderRef.value.clientWidth * 0.5
  sliderRef.value.scrollBy({
    left: direction === 'prev' ? -scrollAmount : scrollAmount,
    behavior: 'smooth'
  })
}

onMounted(() => {
  if (showSlider.value) {
    nextTick(() => {
      updateScrollState()
      if (sliderRef.value) {
        sliderRef.value.addEventListener('scroll', updateScrollState)
      }
      window.addEventListener('resize', updateScrollState)
    })
  }
})

onUnmounted(() => {
  if (sliderRef.value) {
    sliderRef.value.removeEventListener('scroll', updateScrollState)
  }
  window.removeEventListener('resize', updateScrollState)
})

watch(() => props.categories, () => {
  nextTick(updateScrollState)
}, { deep: true })
</script>

<template>
  <section class="section-categories">
    <div class="section-categories__inner">
      <!-- Grid (10개 미만) -->
      <div v-if="!showSlider" class="section-categories__grid">
        <CategoryCard
          v-for="category in categories"
          :key="category.href"
          :category="category"
        />
      </div>

      <!-- Slider (10개 이상) -->
      <div v-else class="section-categories__slider-wrap">
        <IconSlideButton
          direction="prev"
          class="section-categories__arrow section-categories__arrow--left"
          :class="{ 'section-categories__arrow--hidden': !canScrollLeft }"
          :disabled="!canScrollLeft"
          @click="scroll('prev')"
        />

        <div
          ref="sliderRef"
          class="section-categories__slider"
        >
          <CategoryCard
            v-for="category in categories"
            :key="category.href"
            :category="category"
            class="section-categories__slide-item"
          />
        </div>

        <IconSlideButton
          direction="next"
          class="section-categories__arrow section-categories__arrow--right"
          :class="{ 'section-categories__arrow--hidden': !canScrollRight }"
          :disabled="!canScrollRight"
          @click="scroll('next')"
        />
      </div>
    </div>
  </section>
</template>
