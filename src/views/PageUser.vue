<template>
  <main class="main-user">
    <AppHero class="main-user__hero">
      <TheTitleUser />
    </AppHero>
    <h2 class="main-user__caption visually-hidden">Main Content</h2>
    <TheColumnWrapperUser class="main-user__column-wrapper-user" />
    <AppPaginatorList
      v-if="getFeedCount"
      class="main-user__paginator-list"
      :data-item="getDataPaginator"
    />
    <AppPlaceholderPaginator
      v-if="getIsLoadingFeedCount"
      class="main__placeholder-paginator"
    />
  </main>
</template>

<script>
import AppHero from "@/components/AppHero"
import TheTitleUser from "@/components/TheTitleUser"
import TheColumnWrapperUser from "@/components/TheColumnWrapperUser"
import AppPaginatorList from "@/components/AppPaginatorList"
import AppPlaceholderPaginator from "@/components/AppPlaceholderPaginator"

import { mapState } from "vuex"
import { actionTypes as actionTypesUser } from "@/store/modules/user"
import { actionTypes as actionTypesFeedList } from "@/store/modules/feedList"
import { actionTypes as actionTypesFeedCount } from "@/store/modules/feedCount"
import { paginator } from "@/helpers/vars"
import DataPaginator from "@/mixins/dataPaginator"

export default {
  name: "PageUser",

  components: {
    AppHero,
    TheTitleUser,
    TheColumnWrapperUser,
    AppPaginatorList,
    AppPlaceholderPaginator,
  },

  mixins: [DataPaginator],

  computed: {
    ...mapState({
      getFeedCount: ({ feedCount }) => feedCount.feedCount,
      getIsLoadingFeedCount: ({ feedCount }) => feedCount.isLoading,
    }),
  },

  watch: {
    "$route.query"() {
      this.fetchData()
    },
  },

  mounted() {
    this.fetchData()
  },

  methods: {
    async fetchData() {
      try {
        const userName = this.$route.params.user
        const pageNum = this.$route.query.page || 1
        const itemPerPage = paginator.feedList.main

        const userPayload = `userName=${userName}`
        const feedListPayload = `userName=${userName}&_page=${pageNum}&_limit=${itemPerPage}`

        await Promise.allSettled([
          this.$store.dispatch(actionTypesUser.fetchUser, userPayload),
          this.$store.dispatch(
            actionTypesFeedList.fetchFeedList,
            feedListPayload
          ),
          this.$store.dispatch(actionTypesFeedCount.fetchFeedCount, "user"),
        ])
      } catch (err) {
        this.$router.push({ name: "PageError", params: { error: err } })
      }
    },
  },
}
</script>

<style lang="scss">
.main-user {
  @include container;

  padding-bottom: $space-l;
}
</style>
