<template>
  <div class="editor-page">
    <div class="container page">
        <div class="row">

        <div class="col-md-10 offset-md-1 col-xs-12">
            <ul class="error-messages">
                <template v-for="(messages, field) in errors">
                <li v-for="(message, index) in messages" :key="index">
                    {{field}} {{message}}
                </li>
                </template>
            </ul>
            <form @submit.prevent="onSubmit">
                <fieldset class="form-group">
                    <input v-model="article.title" type="text" class="form-control form-control-lg" placeholder="Article Title" required>
                </fieldset>
                <fieldset class="form-group">
                    <input v-model="article.description" type="text" class="form-control" placeholder="What's this article about?" required>
                </fieldset>
                <fieldset class="form-group">
                    <textarea v-model="article.body" class="form-control" rows="8" placeholder="Write your article (in markdown)" required></textarea>
                </fieldset>
                <fieldset class="form-group">
                    <input v-model="article.tagList" type="text" class="form-control" placeholder="Enter tags"><div class="tag-list" ></div>
                </fieldset>
                <button :disabled="submitDisabled" class="btn btn-lg pull-xs-right btn-primary">
                    Publish Article
                </button>
            </form>
        </div>

        </div>
    </div>
  </div>
</template>

<script>
import { getArticle, createArticle, updateArticle } from '@/api/article.js'

export default {
  // 在路由匹配组件渲染之前会先执行中间件处理
  middleware: 'authenticated',
  name: 'EditorIndex',
  data () {
    return {
      errors: {},
      submitDisabled: false
    }
  },
  async asyncData ({ params }) {
    let title, description, body, tagList
    if ( params.slug ) { 
      const { data } = await  getArticle(params.slug)
      const { article } = data
      title = article.title
      description = article.description
      body = article.body
      tagList = article.tagList.join(',')
     }

    return {
          article: {
            title, 
            description, 
            body, 
            tagList
          },
          slug: params.sulg || null
      }
  },
  methods: {
    async onSubmit () {
      try {
        this.submitDisabled = true
        const article = { ...this.article }
        article.tagList = article.tagList.split(',')
        const { data } = this.slug
        ? await updateArticle( this.slug, { article: article } )
        : await createArticle({ article: article })
        this.$router.push({ name: 'article', params: { slug: data.article.slug } })
      } catch (err) {
        this.errors = err.response.data.errors
      }
      this.submitDisabled = false

    }
  }
}
</script>

<style>

</style>