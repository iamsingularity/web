<template lang="pug">
  ais-index(:search-store="searchStore" index-name="videos")
    header
      .container
        nav.level
          .level-left
            .level-item
              a(href="//dev.tube"): img.logo(src="./logo.png")
          .level-item.has-text-centered
              ais-input(placeholder="Search for videos...")
          .level-right(style="font-family: Lato")
            .level-item.links.is-size-10.has-text-white
              a(href="https://devternity.com" target="_blank")
                span inspired by  
                strong DevTernity
    section.section
          .container
            .columns
              .column.is-one-quarter.is-hidden-touch
                .columns(v-if="newVideos.length && !newOnly")
                  .column
                    .content
                      p 
                        b {{newVideos.length}} 
                        | new videos since yesterday!
                      a.button.is-info.is-outlined(v-on:click="showNewVideos()") Show me
                .columns
                  .column
                    h1.title Tags
                    ais-refinement-list.is-capitalized(:class-names="{'ais-refinement-list__count': 'tag'}" attribute-name="tags" :sort-by="['count:desc', 'name:asc']")
                .columns(v-if="!speaker")
                  .column
                    h1.title Speaker
                    ais-refinement-list.is-capitalized(:class-names="{'ais-refinement-list__count': 'tag'}" attribute-name="speaker.name" :sort-by="['count:desc', 'name:asc']")
                .columns
                  .column
                    h1.title Channel
                    ais-refinement-list.is-is-capitalized(:class-names="{'ais-refinement-list__count': 'tag'}" attribute-name="channelTitle" :sort-by="['count:desc', 'name:asc']")
                .columns
                  .column
                    h1.title Year
                    YearRange
              .column
                .columns
                  .column
                    .columns
                      .column
                        ActiveFilters(:speaker="speaker")
                      .column.has-text-right
                        .addthis_inline_share_toolbox
                    ais-no-results
                      template(slot-scope="props")
                        //- .notification(v-if="props.query")
                        .notification
                          h1.title
                            i.far.fa-times-circle 
                            |  No videos matching your query
                          a.button(href="https://github.com/watch-devtube/contrib" target="_blank")
                            span.icon: i.fab.fa-github
                            span Contribute
                        //- .notification.is-danger(v-if="!props.query") 
                          p Sorry, search is not available now. We're working on the solution.
                    ais-results#videos.columns.is-multiline
                      template(slot-scope="{ result }")
                        .column.is-6.is-4-widescreen
                          VideoCard(:tags="result.tags" :featured="result.featured" :tagsClickable="true" :speaker="result.speaker" :creationDate="result.creationDate" :recordingDate="result.recordingDate" :duration="result.duration" :views="result.views" :satisfaction="result.satisfaction" :title="result.title" :id="result.objectID" :channel="result.channelTitle")
    section.section
      .container
        .columns
          .column.has-text-right
            a(href="https://www.algolia.com" target="_blank"): img(src="/search-by-algolia.png")
            br
            br
            nav.paging(role="navigation" aria-label="pagination")
             ais-pagination.pagination(:class-names="{'ais-pagination': 'pagination-list', 'ais-pagination__item': 'page', 'ais-pagination__link': 'pagination-link', 'ais-pagination__item--previous': 'is-hidden', 'ais-pagination__item--next': 'is-hidden', 'ais-pagination__item--active': 'is-current'}")

    Footer
</template>
<style lang="scss">

header {
  background-color: #343d46;
  padding: 30px;

  @media only screen and (max-width: 768px) {
    .logo {
      width: 70px;
      margin-bottom: 10px;
    }
  }

  .links a {
    color: white;
  }

  input {
    -webkit-appearance: none;
    outline: none;
    color: white;
    font-size: 15px;
    font-weight: 100;
    background-color: #343d46;
    padding: 16px 26px 16px 52px;
    border: 1px solid #6498cf;
    border-radius: 3px;
  }
}

  input::placeholder{
    color: #fff;
  }

.paging  {
  .pagination-list {
    justify-content: center;
  }

  .is-current a {
    background-color: #343d46;
    color: white;
  } 
}

#videos {

}
</style>
<script>
import { createFromAlgoliaCredentials } from 'vue-instantsearch'
import VideoCard from './VideoCard.vue'
import Footer from './Footer.vue'
import ActiveFilters from './ActiveFilters.vue'
import YearRange from './YearRange.vue'

const searchStore = createFromAlgoliaCredentials(
  'DR90AOGGE9',
  'c2655fa0f331ebf28c89f16ec8268565'
);

searchStore.queryParameters = { hitsPerPage : 21 }

if (window.speaker) {
  searchStore.queryParameters = { disjunctiveFacets: ['speaker.twitter'] };
  searchStore.algoliaHelper.addDisjunctiveFacetRefinement('speaker.twitter', window.speaker)
}

export default { 
  data: function() {
    return {
      newVideos: window.newVideos,
      searchStore
    };
  },
  watch: {
    '$route': 'fetch'
  },  
  created() {
    this.fetch()
  },
  computed: {
    newOnly() {
      return this.searchStore.algoliaHelper.state.filters && 
        this.searchStore.algoliaHelper.state.filters.includes('objectID')
    }
  },
  methods: {
    fetch() {
      this.speaker = window.speaker;
      this.newVideos = window.newVideos;
    },
    showNewVideos: function() {
      let newOnly = this.newVideos.map(id => `objectID:${id}`).join(' OR ')
      this.searchStore.algoliaHelper.setQueryParameter('filters', `(${newOnly})`)
    }
  },
  components: { ActiveFilters, VideoCard, Footer, YearRange }
};
</script>
