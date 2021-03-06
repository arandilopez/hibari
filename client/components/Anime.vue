<template lang='pug'>
  main.anime
    section.error(v-if='error') Error: {{ error }}

    section.content(v-else)
      .cover(
        v-if='anime && anime.attributes.slug === query'
        v-bind:style='{ backgroundImage: "url(" + anime.attributes.coverImage.original + ")"}'
      )
        img(
          v-if='anime.attributes.posterImage.large'
          v-bind:src='anime.attributes.posterImage.large')
      .cover(v-else)

      nav
        div
          a(:href='"//kitsu.io/anime/" + query' rel='noopener' target='_blank') Kitsu

      spinner(v-if='loading')

      .container
        hr
        p Anime
        pre(v-if='anime') {{ anime }}
</template>

<script>
  import Spinner from 'components/svg/Spinner'

  export default {
    metaInfo () {
      return {
        title: `${this.anime ? this.anime.attributes.canonicalTitle : this.query}`
      }
    },
    components: {
      Spinner
    },
    data () {
      return {
        loading: false,
        error: null,
        query: this.$route.params.query,
        anime: null
      }
    },
    created () {
      this.checkStore()
    },
    methods: {
      checkStore () {
        this.loading = true
        if (this.$store.state.anime[this.$route.params.query] !== undefined) {
          console.info('[HB]: Data retrived from store')
          this.displayData(true)
        } else {
          console.info('[HB]: Data retrived from API')
          this.fetchData()
        }
      },
      displayData (cached, anime) {
        this.anime = cached ? this.$store.state.anime[this.query] : anime
      },
      fetchData () {
        this.error = this.anime = null
        this.loading = true
        this.$http.get(`https://kitsu.io/api/edge/anime?filter[slug]=${this.$route.params.query}`, {}, {
          headers: {
            'Content-Type': 'application/vnd.api+json',
            'Accept': 'application/vnd.api+json',
            'User-Agent': 'hibari'
          }
        })
        .then((data) => {
          this.loading = false
          if (data.body.meta.count === 0) {
            this.error = 'No anime exists'
          } else {
            let anime

            anime = data.body.data[0]
            // delete anime.relationships
            this.$store.commit('ANIME', [anime, this.query])

            this.displayData(false, anime)
          }
        })
        .catch((error) => {
          this.error = error.toString()
        })
      }
    }
  }
</script>

<style lang='sass'>
  @import ~bootstrap/scss/variables
  @import ~bootstrap/scss/mixins
  @import ~bootstrap/scss/grid
  @import ~bootstrap/scss/nav
  @import ~bootstrap/scss/navbar
  @import ~assets/variables

  main.anime
    section.content
      .cover
        width: 100vw
        height: 400px
        background-color: $primary
        background-size: cover
        background-position: center
        position: relative

        img
          left: calc((100vw - 10rem) / 2)
          border-radius: 3px
          width: 10rem
          top: calc(25% + 28px)
          position: absolute
          display: block
          z-index: 501
          pointer-events: none

      nav
        @extend .navbar
        background: rgba(darken(white, 3), .9)
        border-bottom: 1px solid rgba(black, .05)
        width: 100vw
        position: sticky
        top: 56px
        height: 46px
        z-index: 100
        backdrop-filter: blur(2px)
        transition: background 300ms ease-out

        &:hover
          background: darken(white, 3)
          transition: background 200ms ease-in

        div
          @extend .container
          @extend .nav

        a
          @extend .nav-link
          color: black
          transition: color 200ms ease-out
          padding: 0.16em 1em

          &:hover
            color: $primary
            transition: color 100ms ease-out
</style>
