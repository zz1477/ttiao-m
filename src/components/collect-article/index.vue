<template>
    <van-button :icon="value ? 'star' :'star-o' " :class="value? 'collected' : '' " @click="onCollect" :loading="loading" />
</template>

<script>
import { addCollect, deleteCollect } from '@/api/article'
export default {
    name: 'CollectArticle',
    components:  {},
    props: {
        value: {
            type: Boolean,
            required: true
        },
        articleId: {
            type: [Number, String, Object],
            required: true
        }
    },
    data() {
        return {
            loading: false //加载中
        }
    },
    computed: {},
    watch: {},
    created () {},
    mounted () {},
    methods: {
        async onCollect () {
            this.loading = true
            try {
                if(this.value) {
                    // 已收藏 取消收藏
                    await deleteCollect(this.articleId)
                }else {
                    // 没有收藏 点击收藏
                    await addCollect(this.articleId)
                }

                // 更新视图
                this.$emit('input', !this.value)

                this.$toast.success(!this.value ? '收藏成功' : '取消收藏')
            } catch(err) {
                this.$toast.fail('操作失败， 请重试')
            }
            this.loading = false
        }
    }
}
</script>

<style lang="less" scoped>
.collected {
    .van-icon {
        color:#ffa500
    }
}
</style>