<template lang='pug'>
  header(v-bind:class='{ transparent: position }')
    nav
      h1 {{ $t('hibari') }}
        span(v-if='development') Dev
      router-link(:to='{name: "Home"}') {{ $t('navigation.home') }}
      language-switcher
</template>

<script>
  import LanguageSwitcher from 'components/app/LanguageSwitcher'

  export default {
    components: {
      LanguageSwitcher
    },
    computed: {
      development () {
        return window.location.hostname === 'localhost'
      }
    },
    data () {
      return {
        position: (this.$route.params.slug || this.$route.params.query) ? ((window.pageYOffset || document.documentElement.scrollTop) < 100) : false
      }
    },
    methods: {
      handleScroll (event) {
        if (this.$route.params.slug || this.$route.params.query) {
          this.position = (window.pageYOffset || document.documentElement.scrollTop) < 100
        }
      }
    },
    beforeDestroy: function () {
      window.removeEventListener('scroll', this.handleScroll)
    },
    mounted () {
      window.addEventListener('scroll', () => {
        setTimeout(this.handleScroll, 250)
      })
    }
  }
</script>

<style lang='sass'>
  @import ~bootstrap/scss/variables
  @import ~bootstrap/scss/mixins
  @import ~bootstrap/scss/grid
  @import ~bootstrap/scss/nav
  @import ~bootstrap/scss/navbar
  @import ~bootstrap/scss/utilities
  @import ~assets/variables

  header
    @extend .navbar
    @extend .navbar-light
    @extend .fixed-top
    background: $primary
    transition: background 400ms ease-out
    overflow-y: hidden

    &.transparent
      background: transparent
      transition: background 400ms ease-in-out

      &:hover
        background: rgba($primary, .95)
        transition: background 300ms ease-in-out

    nav
      @extend .nav
      @extend .container
      flex-align: flex-start

    h1
      @extend .navbar-brand
      @extend .mb-0
      padding-left: 15px
      color: $white
      font-weight: 700
      cursor: default
      &:hover
        color: $white
      span
        font-weight: 400

    a
      @extend .nav-link
      transition: color 200ms ease-out
      color: rgba($white, .7)

      &:hover
        color: $white
        transition: color 100ms ease-in
</style>
