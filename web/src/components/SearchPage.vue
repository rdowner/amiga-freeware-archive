<template>
    <div>

        <b-input-group class="mt-3 mb-3" size="lg">
            <b-form-input placeholder="Search term" v-model="query"></b-form-input>
            <b-input-group-append>
                <b-button v-on:click="search" variant="info">Search &gt;</b-button>
            </b-input-group-append>
        </b-input-group>

        <div v-if="!ready" class="text-center">
            <b-spinner variant="primary" label="Searching..."></b-spinner>
        </div>

        <div v-if="ready">
            <div class="float-right">
                <b-pagination-nav v-model="page" :number-of-pages="totalPages" :no-page-detect="true"
                                  :link-gen="linkGen"></b-pagination-nav>
            </div>
            <p>Found <strong>{{ totalHits }}</strong> results. Showing page {{ page }}, results {{ startResult }} to {{
                endResult }}:</p>
            <ol v-bind:start="startResult">
                <li v-for="item of searchHits" v-bind:key="item._id">
                    <SearchResult v-bind:product="item"></SearchResult>
                </li>
            </ol>
        </div>

    </div>
</template>

<script>
    import axios from 'axios'
    import SearchResult from './SearchResult.vue'

    export default {
        name: 'SearchPage',
        components: {
            SearchResult
        },
        data: function () {
            return {
                page: 1,
                searchHits: [],
                totalHits: 0,
                startResult: 0,
                endResult: 0,
                totalPages: 0,
                ready: false
            }
        },
        methods: {
            search: function () {
                let me = this;
                me.ready = false;
                let newPage = parseInt(this.$route.query.page || 1) || 1;
                let from = ((newPage - 1) * 20);
                axios.get('/search?q=' + this.$route.query.q + '&from=' + from)
                    .then(response => {
                        me.searchHits = response.data.result.hits.hits;
                        me.totalHits = response.data.result.hits.total;
                        me.startResult = from + 1;
                        me.endResult = Math.min(me.totalHits, ((newPage) * 20));
                        me.totalPages = Math.max(1, Math.ceil(me.totalHits / 20));
                        me.page = newPage;
                        me.ready = true;
                    });
            },
            linkGen: function (pageNum) {
                return {name: 'search', query: {q: this.$route.query.q, page: pageNum}}
            }
        },
        created: function () {
            this.search();
        },
        watch: {
            // eslint-disable-next-line
            '$route.query.page': function (page) {
                this.search();
            }
        }
    }
</script>

<style scoped>
</style>
