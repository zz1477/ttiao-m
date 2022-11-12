<template>
  <div class="search-result">
    <van-list
      v-model="loading"
      :finished="finished"
      finished-text="没有更多了"
      :error.sync="error"
      error-text="加载失败,请点击重试"
      @load="onLoad"
    >
      <van-cell v-for="(article, index) in list" :key="index" :title="article.title" />
    </van-list>
  </div>
</template>

<script>
import { getSearchResult } from '@/api/search'
export default {
  name: 'SearchResult',
  components: {},
  props: {
    searchText: {
      tyep: String,
      required: true
    }
  },
  data () {
    return {
        list: [],
      loading: false,
      finished: false,
      page: 1 ,
      per_page:  15,
      error: false
    }
  },
  computed: {},
  watch: {},
  created () {},
  mounted () {},
  methods: {
   async onLoad() {
      // 异步更新数据
      // setTimeout 仅做示例，真实场景中一般为 ajax 请求
/*       setTimeout(() => {
        for (let i = 0; i < 10; i++) {
          this.list.push(this.list.length + 1);
        }

        // 加载状态结束
        this.loading = false;

        // 数据全部加载完成
        if (this.list.length >= 40) {
          this.finished = true;
        }
      }, 1000); */
    
      try {
        // 请求获取数据
        const { data } = await getSearchResult ({
          page: this.page, //  页码
          per_page: this.per_page, // 每页数据大小
          q: this.searchText  // 查询关键词
        })

        console.log(data);

        // 将数据添加到数组列表中
        const { results } = data.data
        this.list.push(...results)

        // 将本次加载中的loading 关闭
        this.loading = false

        // 判断是否还有数据
        if (results.length) {
          // 如果有 则更新获取下一个数据的页码
          this.page++

        }else {
          // 如果没有 则将加载状态 finished 设置为结束
          this.finished = true
        }


      } catch (err) {
        // 展示加载失败的提示状态
        this.error = true

        // 加载失败 loading 也要关闭
        this.loading = false

        // this.$toast('数据获取失败，请稍后重试')
      }
       
    // 如果没有 则将加载状态 finished 设置为终点
    },
  }
}
</script>

<style lang="less" scoped>
</style>