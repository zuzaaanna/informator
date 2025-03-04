<template>
  <PdfView v-if='article.pdf' :url='article.pdf'></PdfView>
  <div v-else class='fill-height d-flex flex-wrap flex-column justify-space-between'>
    <div class='article pa-4 pa-sm-6 pa-md-8' style='width: 100%'>
      <bread-crumbs />
      <article>
        <h1 class='text-h2 mb-5'>{{ article.title }}</h1>
        <nuxt-content :document='article' />
        <table-of-content :toc='article.toc' />
        <div class='d-flex flex-row justify-space-between mt-7'>
          <div class='text-caption align-self-center'>
            {{ $t('pageLastUpdated') }}: {{ (new Date(modificationDate)).toISOString().split('T')[0] }}
          </div>
          <v-btn outlined color='primary' :href='editLink' small>
            <v-icon
              left
            >
              mdi-pencil
            </v-icon>
            {{ $t('proposeChanges') }}
          </v-btn>
        </div>
      </article>
    </div>
    <v-footer padless color='rgba(0, 0, 0, 0)' class='mb-2'>
      <v-col
        class='text-center'
        cols='12'
      >
        {{ new Date().getFullYear() }} — <strong><a class='text-decoration-none' href='https://samorzad.uj.edu.pl/'>
        {{ $t('ssuj') }}</a></strong>
      </v-col>
    </v-footer>
  </div>
</template>

<script>

import modificationDates from '~/content/modificationDates'

export default {
  name: 'ContentView',
  async asyncData({ $content, app, params, error }) {
    const path = `${app.i18n.locale}/${params.pathMatch}/index`
    const pathWithExt = `${app.i18n.locale}/${params.pathMatch}/index.md`
    const pathInCollection = `${params.pathMatch}/index`
    try {
      const article = await $content(path).where({ stub: { $ne: true } }).fetch()
      if (!article) {
        return error({ statusCode: 404, message: 'Article not found' })
      }

      const modificationDate = (modificationDates[pathWithExt] !== undefined)
        ? modificationDates[pathWithExt] : article.updatedAt

      return {
        article, modificationDate, pathInCollection
      }
    } catch (ex) {
      return error({ statusCode: 404, message: 'Article not found' })
    }
  },
  data: () => ({}),
  head() {
    return {
      title: this.article.title,
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: this.article.description
        }
      ]
    }
  },
  computed: {
    editLink() {
      return `/edit/#/collections/pages_${this.$i18n.locale}/entries/${this.pathInCollection}`
    }
  }
}
</script>

<style lang='sass'>
.article
  max-width: 1200px
  margin-left: auto
  margin-right: auto

  p
    margin: 0.5rem 0

  blockquote
    border-left: 5px solid #eee
    padding: 8px 0 8px 24px !important

  hr
    margin-top: 20px
    margin-bottom: 20px
    border: 0
    border-top: 1px solid #eee

  .heading-link
    content: '#'

</style>
