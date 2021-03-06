<template>
  <v-card :class="cardClass"
          tag="article">
    <!-- <v-card-media>
    </v-card-media> -->
    <v-card-title>
      <v-flex xs12 v-if="notHome">
        <router-link :to="page.path"
                     class="headline post-title-link"
                     v-if="isList">{{page.title}}</router-link>
        <h2 class="display-1 mb-3"
            v-else>{{page.title}}</h2>
        <div class="post-meta">
          <PostTime v-if="page.frontmatter" :date="page.frontmatter.date"></PostTime>
          <span class="title_views">文章访问量:{{page.titleViews}}</span>
        </div>
      </v-flex>
    </v-card-title>
    <v-card-text class="pt-0 pb-0">
      <v-flex xs12>
        <slot>{{page.excerpt}}</slot>
      </v-flex>
    </v-card-text>
    <v-card-actions>
      <v-flex xs12>
        <Tag v-if="page.frontmatter" v-for="tag in page.frontmatter.tags"
             :key="tag"
             :slug="tag">{{tag}}</Tag>
      </v-flex>
    </v-card-actions>
  </v-card>
</template>
<script>
import Tag from './Tag'
import PostTime from './PostTime'
import { matchSlug } from '../libs/utils'

export default {
  data() {
    return {
      page: {},
    }
  },
  created() {
    this.getTitleViews()
    try {
      window && this.haveTitleViews()
    } catch (e) {
      console.error(e.message)
    }
  },
  components: {
    Tag,
    PostTime
  },
  props: {
    post: {
      type: [String, Object],
      required: true
    },
    shadowZ: Number,
    layout: {
      type: String,
      required: true
    }
  },
  computed: {
    isList() {
      return this.layout === 'list'
    },
    notHome() {
      return this.page.frontmatter && this.page.frontmatter.layout !== 'home'
    },
    cardClass() {
      return [
        this.shadowZ ? `elevation-${this.shadowZ}` : '',
        `${this.layout}-card`
      ]
    }
  },
  methods: {
    haveTitleViews() {
      setTimeout(() => {if(typeof this.page.titleViews === 'undefined') {this.getTitleViews(); this.haveTitleViews();}}, 500)
    },
    getTitleViews() {
      this.page = Object.assign({}, typeof this.post === 'string' ? this.$blog.posts[this.post] : this.post)
      if(typeof this.page.titleViews === 'undefined' && Object.keys(this.$blog.pageViews).length) {
        const slug = matchSlug(this.$route.path)
        this.page = Object.assign(this.page, {titleViews: this.$blog.pageViews.titleViewsMap[slug]})
      }
    }
  }
}
</script>
<style lang="stylus">
@import '../styles/config.styl';

.post-card {
  // padding: 0 16px 16px;
}

.list-card {
  .card__title {
    padding-bottom: 0;
  }
}

.post-title-link {
  position: relative;
  display: inline-block;
  text-decoration: none;

  &:after {
    content: '';
    position: absolute;
    width: 100%;
    height: 2px;
    bottom: 0;
    left: 0;
    background-color: $primary-color;
    visibility: hidden;
    transform: scaleX(0);
    transition: 0.3s ease-in-out;
  }

  &:hover, &:active {
    &:after {
      visibility: visible;
      transform: scaleX(1);
    }
  }
}

.title_views
  margin 0 0 0 10px
  color #6d6d6d
</style>
