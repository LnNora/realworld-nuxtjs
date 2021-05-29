<template>
  <div class="settings-page">
    <div class="container page">
        <div class="row">

        <div class="col-md-6 offset-md-3 col-xs-12">
            <h1 class="text-xs-center">Your Settings</h1>
            <ul class="error-messages">
            <template v-for="(messages, field) in errors">
              <li v-for="(message, index) in messages" :key="index">
                {{field}} {{message}}
              </li>
            </template>
        </ul>
            <form @submit.prevent="onSubmit">
            <fieldset>
                <fieldset class="form-group">
                    <input v-model="newUser.image" class="form-control" type="text" placeholder="URL of profile picture">
                </fieldset>
                <fieldset class="form-group">
                    <input v-model="newUser.username" class="form-control form-control-lg" type="text" placeholder="Your Name">
                </fieldset>
                <fieldset class="form-group">
                    <textarea v-model="newUser.bio" class="form-control form-control-lg" rows="8" placeholder="Short bio about you"></textarea>
                </fieldset>
                <fieldset class="form-group">
                    <input v-model="newUser.email" class="form-control form-control-lg" type="text" placeholder="Email">
                </fieldset>
                <fieldset class="form-group">
                    <input v-model="newUser.password" class="form-control form-control-lg" type="password" placeholder="Password">
                </fieldset>
                <button class="btn btn-lg btn-primary pull-xs-right">
                    Update Settings
                </button>
            </fieldset>
            </form>
            <hr>
            <button class="btn btn-outline-danger" @click="logout">
                 Or click here to logout.
            </button>
        </div>

        </div>
    </div>
  </div>
</template>

<script>
import { modifyUser } from '@/api/user.js'

// 仅在客户端加载js-cookie包
const Cookie = process.client ? require('js-cookie') : undefined

export default {
  middleware: 'authenticated',
  name: 'SettingIndex',
  data () {
    return {
      newUser: Object.assign({ password: '' }, this.$store.state.user),
      errors: {}
    }
  },
  methods: {
    async onSubmit () {
      try {
        const { email, username, password, image, bio} = this.newUser
        const { data } = await modifyUser({ user: {
          email,
          username,
          password,
          image,
          bio
        }})
        this.$store.commit('setUser', data.user)

        // 为了防止刷新页面丢失，我们需要把数据持久化
        Cookie.set('user', data.user)

        // 跳转到首页
        this.$router.push('/')
      } catch (err) {
        this.errors = err.response.data.errors
      }
      
    },
    logout () {
      this.$store.commit('setUser', null)
      Cookie.remove('user')
      // 跳转到首页
      this.$router.push('/')
    }
  }
}
</script>

<style>

</style>