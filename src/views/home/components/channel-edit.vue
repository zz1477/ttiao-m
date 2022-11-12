<template>
  <div class="channel-edit">
    <van-cell :border="false">
      <div slot="title" class="title-text">我的频道</div>
      <van-button class="edit-btn" type="danger" plain round size="mini" @click="isEdit = !isEdit"
        >{{ isEdit ? '完成' : '编辑' }}</van-button
      >
    </van-cell>
    <van-grid class="my-grid" :gutter="10">
      <van-grid-item
      @click="onMyChannelClick(channel, index)"
        class="grid-item"
        v-for="(channel, index) in myChannels"
        :key="index"
      >
      <van-icon slot="icon" name="clear" v-show="isEdit && !fiexdChannels.includes(channel.id)"></van-icon>


        <!-- 
          v-bind:class 语法
          一个对象，对象中的 key 表示要作用的 CSS 类名
                    对象中的 value 要计算出布尔值
                      true，则作用该类名
                      false，不作用类名
       -->
        <span class="text" :class="{ active: index === active }" slot="text">{{
          channel.name
        }}</span>
      </van-grid-item>
    </van-grid>

    <!-- 频道推荐 -->
    <van-cell :border="false">
      <div slot="title" class="title-text">频道推荐</div>
    </van-cell>
    <van-grid class="recommend-grid" :gutter="10">
      <van-grid-item
        class="grid-item"
        v-for="(channel, index) in recommendChannels"
        icon="plus"
        :key="index"
        :text="channel.name"
        @click="onAddChannel(channel)"
      />
    </van-grid>
  </div>
</template>

<script>
    import { getAllChannels, addUserChannel, delteUserChannel } from '@/api/channel'
    import { mapState } from 'vuex'
    import { setItem } from '@/utils/storage'

export default {
  name: 'ChannelEdit',
  components: {},
  props: {
    // 我的频道
    myChannels: {
      type: Array,
      required: true
    },
    active: {
      type: Number,
      required: true
    }
  },
  data () {
    return {
        allChannels: [], // 所有频道
        isEdit: false, // 控制编辑状态的显示与隐藏
        fiexdChannels: [0] // 固定频道 不允许删除
    }
  },
  computed: {
    ...mapState(['user']),
    // 计算属性会观测内部依赖数据的变化
    // 如果依赖的数据发生变化，则计算属性会重新执行

    recommendChannels () {
      // 数组的 filter 方法：遍历数组，把符合条件的元素存储到新数组中并返回
      return this.allChannels.filter(channel => {
        // const channels = []

        // 数组的 find 方法：遍历数组，把符合条件的第1个元素返回
        return !this.myChannels.find(mychannel => {
          return mychannel.id === channel.id
        })
        // return channels
      })
    }
    /* recommendChannels () {
       const channels = []
       this.allChannels.forEach(channel => {
        // find 遍历数组 找到满足条件的元素项
        const ret = this.myChannels.find(myChannels => {
          return myChannels.id === channel.id
        })

        // 如果我的频道中不包括该频道项 则该收集到推荐频道中
        if (!ret) {
          channels.push(channel)
        }
       })

       // 把计算结果返回
       return channels
    } */
  },
  watch: {},
  created () {
    this.loadAllChannels()
  },
  mounted () {},
  methods: {
    async loadAllChannels () {
        try {
            const { data } = await getAllChannels()
            this.allChannels = data.data.channels
            console.log(data);
        } catch (err) {
            this.$toast('数据获取失败')
        }
    },

   async onAddChannel (channel) {
      this.myChannels.push(channel)

      // 数据持久化处理
      if (this.user) {
        try {
          // 已登录，把数据请求接口放到线上
          await addUserChannel({
            id: channel.id, // 频道ID
            seq: this.myChannels.length // 序号
          })
        } catch (err) {
          this.$toast('保存失败，请稍后重试')
        }
      } else {
        // 未登录，把数据存储到本地
        setItem('TOUTIAO_CHANNELS', this.myChannels)
      }
          
    },
    // 删除频道
    onMyChannelClick (channel, index) {
      if (this.isEdit) {
        // 如果是固定频道 则不删除
        if (this.fiexdChannels.includes(channel.id)) {
          return
        }
        
        // 删除频道项
        this.myChannels.splice(index, 1)

        // 编辑状态 执行删除频道
        // 要删除的元素的索引
        // 删除的个数 如果不指定 则从头开始一直删除
        if (index <= this.active) {
          //让激活的频道的索引-1
          this.$emit('update-active', this.active - 1, true)
        }

        // 删除持久化
        this.deleteChannel (channel)
      }else {
        // 非编辑状态 执行切换频道
        this.$emit('update-active', index, false)
      }
    },

    async deleteChannel (channel) {
      try {
        if (this.user) {
        // 已登录则将数据更新到线上
       await delteUserChannel(channel.id)
      } else {
        // 未登录则将数据更新到本地
        setItem('TOUTIAO_CHANNELS', this.myChannels)
      }
      } catch (err) {
        this.$toast('操作是啊比，请稍后重试')
      }
    }

  }
}
</script>

<style lang="less" scoped>
.channel-edit {
  padding: 85px 0;

  .title-text {
    font-size: 32px;
    color: #333;
  }

  .edit-btn {
    width: 104px;
    height: 48px;
    font-size: 26px;
    color: #f85959;
    border: 1px solid #f85959;
  }

  /deep/ .grid-item {
    width: 160px;
    height: 86px;
    .van-grid-item__content {
      white-space: nowrap;
      background-color: #f4f5f6;
      .van-grid-item__text,
      .text {
        font-size: 28px;
        color: #222;
        margin-top: 0;
      }
      .active {
        color: red;
      }
      .van-grid-item__icon-wrapper {
        position: unset;
      }
    }
  }

  /deep/ .my-grid {
    .grid-item {
      .van-icon-clear {
        position: absolute;
        right: -10px;
        top: -10px;
        font-size: 30px;
        z-index: 2;
        color: #cacaca;
      }
    }
  }

  /deep/ .recommend-grid {
    .van-grid-item__content {
      flex-direction: row;
      .van-icon-plus {
        font-size: 28px;
        margin-right: 6px;
      }
    }
  }
}
</style>