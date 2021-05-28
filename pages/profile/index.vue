<template>
  <div class="profile-page">

    <div class="user-info">
    <div class="container">
        <div class="row">

        <div class="col-xs-12 col-md-10 offset-md-1">
            <img :src="profile.image" class="user-img" />
            <h4>{{ profile.username }}</h4>
            <p>{{ profile.bio }}</p>
            <nuxt-link v-if="user && user.username === username"  :to="{ name: 'settings' }" class="btn btn-sm btn-outline-secondary action-btn">
              <i class="ion-gear-a"></i> Edit Profile Settings
            </nuxt-link>
            <button v-else @click="follow" :disabled = followDisabled class="btn btn-sm btn-outline-secondary action-btn">
            <i class="ion-plus-round"></i>
            &nbsp;
            {{ !profile.following ? 'Follow' : 'Unfollow' }} {{ profile.username }} 
            </button>
        </div>

        </div>
    </div>
    </div>

    <div class="container">
    <div class="row">

        <div class="col-xs-12 col-md-10 offset-md-1">
        <!--tab栏-->
        <div class="articles-toggle">
            <ul class="nav nav-pills outline-active">
                <li class="nav-item">
                    <nuxt-link class="nav-link" :to="{ name: 'profile', query: { tab: 'my' } }" :class="{ active: tab === 'my' }" exact>My Articles</nuxt-link>
                </li>
                <li class="nav-item">
                    <nuxt-link class="nav-link" :to="{ name: 'profile', query: { tab: 'favorites' } }" :class="{ active: tab === 'favorites' }"  exact>Favorited Articles</nuxt-link>
                </li>
            </ul>
        </div>

        <!--文章列表-->
        <div 
          class="article-preview"
          v-for="article in articles"
          :key="article.slug">
          <div class="article-meta">
            <nuxt-link :to="{name: 'profile', params: {username: article.author.username}}"><img :src="article.author.image" /></nuxt-link>
            <div class="info">
             <nuxt-link class="author" :to="{name: 'profile', params: {username: article.author.username}}">{{article.author.username}}</nuxt-link>
              <span class="date">{{ article.createdAt | date('MMM DD, YYYY') }}</span>
            </div>
            <button 
              class="btn btn-outline-primary btn-sm pull-xs-right" 
              :class="{ active: article.favorited }"
              @click="onFavorite(article)"
              :disabled = article.favoriteDisabled
              >
              <i class="ion-heart"></i> {{ article.favoritesCount }}
            </button>
          </div>
          <nuxt-link :to="{ name: 'article', params: { slug: article.slug } }" class="preview-link">
            <h1>{{ article.title }}</h1>
            <p>{{ article.description }}</p>
            <span>Read more...</span>
          </nuxt-link>
        </div>

        <!--分页列表-->

       <ul class="pagination">
          <li 
            class="page-item"
            :class="{
              active: item === page
            }"
            v-for="item in totalPage" 
            :key="item"
            @click="changePage(item)"
            >
            <a class="page-link" href="#">{{ item }}</a>
            <!-- <nuxt-link class="page-link" :to="{ name: 'profile', query: { page: item, tab: tab } }" >{{ item }}</nuxt-link> -->
          </li>
       </ul>

        </div>

    </div>
    </div>

  </div>
</template>

<script>
import { getProfile, follow } from '@/api/user'
import { getArticles, addFavorite, deleteFavorite } from '@/api/article'
import { mapState } from 'vuex'


export default {
  watchQuery: ['page', 'tab'],
  name: 'UserProfile',
  async asyncData ({ params, query, store }) {
    const { tab = 'my' } = query
    const { username } = params
    const page = Number.parseInt(query.page || 1)
    const limit = 5
    const apiParams = {  
        limit,
        offset: (page - 1) * limit
    }
    if ( tab === 'my' ) {
        apiParams.author = username
    } else {
        apiParams.favorited = username
    }

    const [ profileRes,  articleRes] = await Promise.all([ getProfile(username), getArticles(apiParams)])

    const { profile } = profileRes.data
    const { articles, articlesCount } = articleRes.data

    articles.forEach(article => {
      article.favoriteDisabled = false
    })


    return {
      username,
      profile,
      articles,
      articlesCount,
      limit,
      page,
      tab,
      followDisabled: false
    }
  },
  computed: {
      ...mapState(['user']),
      totalPage () {
        return Math.ceil(this.articlesCount / this.limit)
      }
  },
  methods: {
    async onFavorite (article) {
      article.favoriteDisabled = true
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
      article.favoriteDisabled = false
    },
    async changePage (page) {
      this.page = page
      const { data } = await getArticles({
        limit: this.limit,
        offset: (this.page - 1) * this.limit,
        author: this.tab === 'my' ? this.username : '',
        favorited: this.tab === 'favorites' ? this.username : '',
      })
      console.log(data)
      this.articles = data.articles
      this.articlesCount = data.articlesCount
    },
    async follow () {
      if ( !this.user ) {
        this.$router.push({ name: 'login' })
        return
      }
      try {
        this.followDisabled = true
        const { data } = await follow(this.username)
        console.log(data)
        this.profile.following = !this.profile.following
      } catch {

      }
      this.followDisabled = false
      
    }
  }
}
</script>

<style>

</style>