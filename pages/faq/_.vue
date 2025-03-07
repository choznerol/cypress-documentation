<script>
import AppSidebar from '../../components/AppSidebar'
import AppHeader from '../../components/AppHeader'
import Footer from '../../components/Footer'
import TableOfContents from '../../components/TableOfContents'
import TableOfContentsList from '../../components/TableOfContentsList.vue'
import { getMetaData, getMetaDescription, getTitle } from '../../utils'
import { fetchBanner } from '../../utils/sanity'

export default {
  components: {
    AppSidebar,
    AppHeader,
    Footer,
    TableOfContents,
    TableOfContentsList,
  },
  async asyncData({ $content, app, params, error }) {
    const path = `/faq/${params.pathMatch || 'index'}`
    const { algolia: algoliaSettings } = await $content('settings').fetch()
    const [faqItem] = await $content({ deep: true }).where({ path }).fetch()
    const { faq } = await $content('_data/sidebar').fetch()
    const sidebarItems = faq[0].children

    if (!faqItem) {
      return error({ statusCode: 404, message: 'FAQ not found' })
    }

    const [rawContent] = await $content({ deep: true, text: true })
      .where({ path })
      .fetch()
    const metaDescription = await getMetaDescription(rawContent.text)

    const banner = await fetchBanner()

    return {
      algoliaSettings,
      faqItem,
      sidebarItems,
      metaDescription,
      path: params.pathMatch,
      banner,
    }
  },
  head() {
    return {
      title: getTitle(this.faqItem.title),
      meta: this.meta,
      link: [
        {
          hid: 'canonical',
          rel: 'canonical',
          href: `https://docs.cypress.io/faq/${this.$route.params.pathMatch}`,
        },
      ],
    }
  },
  computed: {
    meta() {
      const metaData = {
        type: 'article',
        title: getTitle(this.faqItem.title),
        description: this.metaDescription,
        url: `https://docs.cypress.io/faq/${this.$route.params.pathMatch}`,
      }

      return getMetaData(metaData)
    },
  },
}
</script>

<template>
  <div class="w-full">
    <AppHeader
      :mobile-menu-items="sidebarItems"
      section="faq"
      :algolia-settings="algoliaSettings"
      :banner="banner"
    />
    <main :class="Boolean(banner) ? 'banner-margin' : ''" class="main-content">
      <AppSidebar
        :items="sidebarItems"
        section="faq"
        :path="path"
        :has-banner="Boolean(banner)"
      />
      <div class="main-content-article-wrapper">
        <article class="main-content-article hide-scroll">
          <h1 class="main-content-title">{{ faqItem.title }}</h1>
          <div class="w-full flex flex-col justify-between">
            <TableOfContentsList :toc="faqItem.toc" />
            <nuxt-content :document="faqItem"></nuxt-content>
            <Footer />
          </div>
        </article>
      </div>
      <TableOfContents />
    </main>
  </div>
</template>
