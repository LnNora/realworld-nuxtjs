<template>
    <div class="article-meta">
        <nuxt-link :to="{ name: 'profile', params: { username: article.author.username } }">
            <img :src="article.author.image" />
        </nuxt-link>
        <div class="info">
            <nuxt-link :to="{ name: 'profile', params: { username: article.author.username } }" class="author">
              {{ article.author.username }}
            </nuxt-link>
            <span class="date">{{ article.createdAt | date('MMM DD, YYYY')}}</span>
        </div>
        <!--如果已登录，且是自己的文章-->
        <template v-if="user && user.username === article.author.username">
          <nuxt-link :to="{ name: 'editor', params: { slug: article.slug } }" class="btn btn-sm btn-outline-secondary">
              <i class="ion-edit"></i>
              &nbsp;
              Edit Aticle
          </nuxt-link>
          &nbsp;&nbsp;
          <button @click="onDelete" class="btn btn-outline-danger btn-sm">
              <i class="ion-trash-a"></i>
              &nbsp;
               Delete Aticle
          </button>
        </template>
        <!--如果未登录或已登录但是别人的文章-->
        <template v-else>
          <button
            @click="follow"
            :disabled = "followDisabled"
            class="btn btn-sm btn-outline-secondary"
            :class="{
              active: article.author.following
            }"
            >
              <i class="ion-plus-round"></i>
              &nbsp;
              {{ article.author.following ? 'Unfollow' : 'Follow' }} {{article.author.username}} <span class="counter">(10)</span>
          </button>
          &nbsp;&nbsp;
          <button 
            @click="onFavorite()"
            :disabled = article.favoriteDisabled
            class="btn btn-sm btn-outline-primary"
            :class="{
              active: article.favorited
            }">
              <i class="ion-heart"></i>
              &nbsp;
              Favorite Post <span class="counter">(29)</span>
          </button>
        </template>
    </div>

</template>

<script>
import { deleteArticle, deleteFavorite, addFavorite } from '@/api/article.js'
import { follow } from '@/api/user'
import { mapState } from 'vuex'
export default {
  name: 'ArticleMeta',
  props: {
    article: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      followDisabled: false,
      favoriteDisabled: false
    }
  },
  computed: {
    ...mapState(['user'])
  },
  methods: {
    async onDelete () {
      await deleteArticle(this.article.slug)
      this.$router.push({ name: 'home' })
    },
    async follow () {
      if ( !this.user ) {
        this.$router.push({ name: 'login' })
        return
      }
      try {
        this.followDisabled = true
        const { data } = await follow(this.article.author.username)
        console.log(data)
        this.article.author.following = !this.article.author.following
      } catch {

      }
      this.followDisabled = false
    },
    async onFavorite () {
      const article = this.article
      this.favoriteDisabled = true
      if ( !this.user ) {
        this.$router.push({ name: 'login' })
        return
      }
      if (article.favorited) {
        await deleteFavorite(article.slug)
        article.favorited = false
        article.favoritesCount -= 1
      } else {
        await addFavorite(article.slug)
        article.favorited = true
        article.favoritesCount += 1
      }
      this.favoriteDisabled = false
    },
  }
}
</script>

<style>

</style>