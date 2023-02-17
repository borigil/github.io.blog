<template>
  <v-container fluid>
    <intro class="mt-5 mb-8">
      <h1 class="text-h2">
        borigil's codes
      </h1>

      <h2 class="mt-3">
        emoji test <span class="emoji">1212</span>
      </h2>

      <v-row class="posts-container">
        <v-col cols="12">
          <div class="filter">
            <!-- v-if="categories.length" -->
            <v-select
              
              v-model="category"
              style="width:160px;"
              outlines
              dense
              hide-details="auto"
              :items="categories"
            />
          </div>
        </v-col>

        <v-col v-for="post in posts" :key="post.slug" cols="12" md="6">
          <v-card>
            <v-card-title> {{ post.title }}</v-card-title>
            <v-card-subtitle>
              {{
                new Intl.DateTimeFormat('ko-KR', {
                  year: 'numeric',
                  month: 'numeric',
                  day: 'numeric',
                  hour: 'numeric',
                  minute: 'numeric',
                  second: 'numeric',
                }).format(new Date( post.createdAt ))
              }}
            </v-card-subtitle>
            <v-card-text>
              <nuxt-content :document="{ body: post.excerpt }" />
            </v-card-text>
            <v-card-actions>
              <v-btn text :to="post.path">Read More</v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>

      <v-row v-if="posts.length" class="posts-pagenation">
        <v-col class="text-right" cols="12">
          <v-btn :disabled="page === 1" @click="fetchPrevious">
            <v-icon small> mdi-arrow-left </v-icon>
            Previous
          </v-btn>


          <v-btn :disabled="!nextPage" @click="fetchNext">
            Next
            <v-icon small> mdi-arrow-right </v-icon>
          </v-btn>


        </v-col>
      </v-row>

    </intro>

  </v-container>
</template>
<script>
export default {
  name: 'HomePage', 
  layout: 'DefalutLayout',

  async asyncData({ $content }) {
    const limit = 5;
    const page = 1;

    const fetchedPosts = await $content()
      .limit(limit)
      .sortBy('creartedAt', 'desc')
      .skip( (limit - 1) * ( page - 1 ) )
      .fetch()

    const nextPage = fetchedPosts.length === limit
    const posts = nextPage ? fetchedPosts.slice(0, -1) : fetchedPosts

    return {
      page,
      limit,
      posts,
      nextPage
    }
  },

  data:() => ({
    category: 'all',
    categories: [],
  }),

  fetch() {
    this.$content
      .only(['category']) 
      .fetch()
      .then((categories) => {
        const payload = Array.from( new Set(categories.map((c) => c.category )) )
        this.categories = [ 'all', ...payload]

      })
  },

  computed: {
    searchQuery() {
      return this.$store.state.query
    },
  },

  watch: {
    async searchQuery( newValue ) {
      this.page = 1
      await this.fetchedPosts( newValue )
    },
    async category() {
      await this.fetchedPosts(this.searchQuery)
    },
  },

  methods: {
    async fetchNext() {
      this.page += 1;
      await this.fetchedPosts();
    },
    async fetchPrevious() {
      this.page -= 1;
      await this.fetchedPosts();
    },
    async fetchedPosts(query=''){
      let baseFetch = await this.$content()
        .limit(this.limit)
        .sortBy('creartedAt', 'desc')
        
      if(this.categoriy !== 'all'){
        baseFetch = baseFetch.wgere({ category : this.category})
      }


      const fetchedPosts = await baseFetch
        .sortBy('creartedAt', 'desc')
        .search(query)
        .skip( ( this.limit - 1 ) * ( this.page - 1 ) )
        .fetch()

      this.nextPage = fetchedPosts.length === this.limit
      const posts = this.nextPage ? fetchedPosts.slice(0, -1) : fetchedPosts

      this.posts = posts
    }
  },

}
</script>
<style>
  
</style>