<template>
  <div class="user-profile">
    <!-- 导航栏 -->
    <van-nav-bar
      class="page-nav-bar"
      title="个人信息"
      left-arrow
      @click-left="$router.back()"
    />

    <input type="file" hidden ref="file" @change="onFileChange" />

    <!-- 个人信息 -->
    <van-cell
      @click="$refs.file.click()"
      class="photo-cell"
      title="头像"
      is-link
      center
    >
      <van-image class="avatar" fit="cover" round :src="user.photo" />
    </van-cell>
    <van-cell
      @click="isUpdateNameShow = true"
      title="昵称"
      :value="user.name"
      is-link
    />
    <van-cell
      @click="isUpdateGenderShow = true"
      title="性别"
      :value="user.gender === 0 ? '男' : '女'"
      is-link
    />
    <van-cell
      @click="isUpdateBirthdayShow = true"
      title="生日"
      :value="user.birthday"
      is-link
    />

    <!-- 编辑昵称 弹出层 -->
    <van-popup
      v-model="isUpdateNameShow"
      style="height: 100%"
      position="bottom"
    >
      <update-name
        v-if="isUpdateNameShow"
        @close="isUpdateNameShow = false"
        v-model="user.name"
      />
    </van-popup>

    <!-- 修改性别 弹出层 -->
    <van-popup v-model="isUpdateGenderShow" position="bottom">
      <update-gender
        @close="isUpdateGenderShow = false"
        v-model="user.gender"
        v-if="isUpdateGenderShow"
      />
    </van-popup>

    <!-- 编辑生日 弹出层 -->
    <van-popup v-model="isUpdateBirthdayShow" position="bottom">
      <update-birthday
        @close="isUpdateBirthdayShow = false"
        v-model="user.birthday"
        v-if="isUpdateBirthdayShow"
      />
    </van-popup>

    <!-- 修改头像 弹出层 -->
    <van-popup
      v-model="isUpdatePhotoShow"
      position="bottom"
      style="height: 100%"
    >
      <update-photo 
      v-if="isUpdatePhotoShow"
      :img="img" 
      @close="isUpdatePhotoShow = false"
      @update-photo="user.photo = $event" />
    </van-popup>
  </div>
</template>

<script>
import { getUserProfile } from '@/api/user'
import UpdateName from './components/update-name'
import UpdateGender from './components/update-gender'
import updateBirthday from './components/update-birthday'
import updatePhoto from './components/update-photo'

export default {
  name: 'UserProfile',
  components: {
    UpdateName,
    UpdateGender,
    updateBirthday,
    updatePhoto
  },
  props: {},
  data () {
    return {
      user: {}, // 用户的个人信息
      isUpdateNameShow: false, // 控制弹出层显示与隐藏
      isUpdateGenderShow: false,
      isUpdateBirthdayShow: false,
      isUpdatePhotoShow: false,
      img: null // 预览的图片
    }
  },
  computed: {},
  watch: {},
  created () {
    this.loadUserProfile()
  },
  mounted () {},
  methods: {
    async loadUserProfile() {
      try {
        const { data } = await getUserProfile()
        this.user = data.data
      } catch (err) {
        this.$toast('数据获取失败')
      }
    },

    onFileChange() {
      // 获取文件对象
      const file = this.$refs.file.files[0]
      // 基于文章对象获取 blob 数据
      this.img = window.URL.createObjectURL(file)
      // this.img = data

      // 展示预览图片弹出层
      this.isUpdatePhotoShow = true

      // file-input 如果选了同一个文件不会触发 change 事件
      // 解决办法就是每次使用完毕，把它的value 清空
      this.$refs.file.value = ''
    }
  }
}
</script>

<style lang="less" scoped>
.user-profile {
  .avatar {
    width: 60px;
    height: 60px;
  }

  .van-popup {
    background-color: #f5f7f9;
  }
}
</style>