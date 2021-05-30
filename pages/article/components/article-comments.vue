<template>
  <div>
    <form class="card comment-form" @submit.prevent="onSubmit">
      <div class="card-block">
        <textarea v-model="comment.body" class="form-control" placeholder="Write a comment..." rows="3" required></textarea>
      </div>
      <div class="card-footer">
        <img :src="user.image" class="comment-author-img" />
        <button class="btn btn-sm btn-primary">
        Post Comment
        </button>
      </div>
    </form>
            
    <div 
      class="card" 
      v-for="comment in comments"
      :key="comment.id">
      <div class="card-block">
        <p class="card-text">{{comment.body}}</p>
      </div>
      <div class="card-footer">
        <nuxt-link 
         :to="{
           name: 'profile',
           params: {
             username: comment.author.username
           }
         }"
         class="comment-author">
        <img :src="comment.author.image" class="comment-author-img" />
        </nuxt-link>
        &nbsp;
        <nuxt-link 
         :to="{
           name: 'profile',
           params: {
             username: comment.author.username
           }
         }"
         class="comment-author">
        {{comment.author.username}}
        </nuxt-link>
        <span class="date-posted">{{ comment.createdAt | date('MMM DD, YYYY') }}</span>
      </div>
    </div>
  </div>
</template>

<script>
import { getComments, createComment } from '@/api/article'
import { mapState } from 'vuex'

export default {
  name: 'ArticleComments',
  props: {
    article: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      comments: [], // 文章列表
      comment: {
        body: ''
      }
    }
  },
  computed: {
    ...mapState(['user'])
  },
  async mounted () {
    const { data } = await getComments(this.article.slug)
    this.comments = data.comments
  },
  methods: {
    async onSubmit () {
      const { data } = await createComment(this.article.slug, { comment: this.comment })
      this.comments.unshift(data.comment)
      this.comment.body = ''
    }
  }
}
</script>

<style>

</style>