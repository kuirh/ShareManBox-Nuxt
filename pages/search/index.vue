<style lang="scss">
  .search {

    &__info {
      padding: 16px;
      text-align: center;
      color: #999;
      font-size: .95rem;

      span {
        color: $theme-color;
        border-bottom: 1px solid $theme-color;
      }
    }
  }
</style>

<template>
  <LayoutContainer class="search">
    <Blocker height="40px" />
    <Nameplate :sub-title="this.$route.query.keyword || ''" title="搜索" />
    <Blocker height="20px" />
    <div v-if="data.resultList && data.resultList.length > 0" class="search__info">
      共 <span>{{ meta.total }}</span> 条线索数据，页数 <span>{{ meta.current_page }}</span> / {{ meta.last_page }}
    </div>
    <Blocker height="40px" />
    <Timeline
      v-if="data.resultList && data.resultList.length > 0"
      :timeline="data.resultList"
      :meta="meta"
      @on-request-more="requestMoreResult()"
    />
    <Tip v-else>
      暂无搜索结果
    </Tip>
  </LayoutContainer>
</template>

<script>
export default {
  head() {
    return {
      title: `${this.$route.query.keyword || '请输入关键字'} - 搜索`,
      meta: [
        {
          hid: 'index',
          name: 'description',
          content: this.$getSeoInfo('description', '搜索')
        }
      ]
    }
  },
  data() {
    return {
      status: {
        currentPage: 1
      }
    }
  },
  async asyncData({ $axios, query }) {
    if (query.keyword) {
      const { data: resultList, meta } = await $axios.$get('/api/common/search', {
        params: {
          pageSize: 10,
          page: query.page,
          keyword: query.keyword
        }
      })
      return {
        data: {
          resultList
        },
        meta
      }
    } else {
      return {
        data: {
          resultList: null
        },
        meta: {}
      }
    }
  },
  mounted() {
  },
  methods: {
    async requestMoreResult() {
      const requestPage = this.status.currentPage + 1
      const { data: timeline, meta } = await this.$axios.$get('/api/common/search', {
        params: {
          keyword: this.$route.query.keyword,
          page: requestPage
        }
      })
      this.data.resultList = [...this.data.resultList, ...timeline]
      this.meta = meta
      this.status.currentPage++
    }
  },
  watchQuery: ['page', 'keyword']
}
</script>
