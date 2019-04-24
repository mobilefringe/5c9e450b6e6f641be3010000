<template>
    <div v-if="dataLoaded" v-cloak>
        <div v-if="pageBanner" class="page_header" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content">
					<h1>Search Results</h1>
					<h2 style="display:none;">Scroll to  view page details</h2>
				</div>
			</div>
		</div>
        <div class="site_container">
    		<div v-if="searchResults && searchResults.length > 0" id="searchResults" class="margin_40">
    		    <div class="row">
    		        <div class="col-md-12">
    		            <p v-if="searchResults.length > 1" class="search_result_title">Found {{ searchResults.length }} results matching "{{ searchQuery }}"</p>
                        <p v-else class="search_result_title">Found {{ searchResults.length }} result matching "{{ searchQuery }}"</p>
    		        </div>
    		    </div>
    			<div v-for="(result,index) in searchResults" :key="index">
                    <div class="row result_container_row">
                        <div v-if="result.is_store" class="col-sm-3">
                            <div class="store_details_image center-block">
                                <div v-if="(result.image_url && _.includes(result.image_url,'missing')) || (!result.image_url && _.includes(result.store.store_front_url_abs,'missing'))">
                                    <div class="no_logo">
                                        <img src="//codecloud.cdn.speedyrails.net/sites/5b88438d6e6f641e8d3c0000/image/png/1536092029690/transparent_logo.png">
                                        <p class="store_details_name">
                                            <span v-if="result.store_front_url_abs">{{ result.name }}</span>
                                            <span v-else>{{ result.store.name }}</span>
                                        </p>
                                    </div>    
                                </div> 
                                <div v-else>
                                    <img v-if="result.store" class="result_logo" :src="result.store.store_front_url_abs"/>
                                    <img v-else-if="result.store_front_url_abs" class="result_logo" :src="result.store_front_url_abs"/>
                                </div>
                            </div>
                        </div>
                        <div v-else class="col-sm-3">
                            <div class="store_details_image center-block">
                                <img class="result_logo" :src="siteInfo.default_logo_url"/>    
                            </div>
                        </div>
                        <div class="col-sm-9 search_result_content">
                            <h4>{{ result.name }}</h4>
                            <p>{{ truncated(result.description) }}</p>
                            <router-link v-if="result.store_front_url_abs" class="event_learn_more pull-left hvr-icon-wobble-horizontal" :to="{ name: 'storeDetails', params:{ id:result.slug }}">
                                View Store Details <i class="fa fa-angle-right hvr-icon" aria-hidden="true"></i>
                            </router-link>
                            <router-link v-else-if="result.promo_image_url_abs" class="event_learn_more pull-left hvr-icon-wobble-horizontal" :to="{ name: 'promotionDetails', params: { id: result.slug }}">
                                View Promotion Details <i class="fa fa-angle-right hvr-icon" aria-hidden="true"></i>
                            </router-link>
                            <router-link v-else-if="result.event_image_url_abs" class="event_learn_more pull-left hvr-icon-wobble-horizontal" :to="{ name: 'eventDetails', params: { id: result.slug }}">
                                View Event Details <i class="fa fa-angle-right hvr-icon" aria-hidden="true"></i>
                            </router-link>
                            <router-link v-else-if="result.jobable_id" class="event_learn_more pull-left hvr-icon-wobble-horizontal" :to="{ name: 'jobDetails', params: { id: result.slug }}">
                                View Job Details <i class="fa fa-angle-right hvr-icon" aria-hidden="true"></i>
                            </router-link>
                        </div>
                    </div>                
                </div>               
            </div>
            <div v-else class="row margin_40">
                <div class="col-md-12" > 
                    <p>Sorry, there are no results that match your search. Please try again.</p>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    define(["Vue", "vuex", "jquery"], function(Vue, Vuex, $) {
        return Vue.component("search-results", {
            template: template, // the variable template will be injected
            data() {
                return {
                    dataLoaded: true,
                    searchResults: null,
                    searchQuery: null
                }
            },
            beforeRouteUpdate(to, from, next) {
                this.$nextTick(function() {
                    this.updateResults();
                });
                next();
            },
            created() {
                this.updateResults();
                if (this.$route.params.results == null && this.$route.params.results == undefined) {
                    this.$router.push("/");
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName'
                ]),
                pageBanner() {
                    var pageBanner = null;
                    var temp_repo = this.findRepoByName('Pages Banner');
                    if (temp_repo && temp_repo.images) {
                        pageBanner = temp_repo.images[0];
                    } else {
                        pageBanner = {};
                        pageBanner.image_url = "";
                    }
                    return pageBanner;
                }
            },
            methods: {
                truncated(string) {
                    return _.truncate(string, { length: 150,  separator: " " });
                },
                updateResults() {
                    if (this.$route.query.searchQuery !== null && this.$route.query.searchQuery !== undefined && this.$route.query.searchQuery.length > 0) {
                        if (
                            this.$route.params.results !== null &&
                            this.$route.params.results !== undefined &&
                            Array.isArray(this.$route.params.results)
                        ) {
                            this.searchResults = this.$route.params.results;
                            this.searchQuery = this.$route.query.searchQuery;
                        }
                    } else {
                        this.$router.push("/");
                    }
                    console.log(this.searchResults)
                },
                checkResultImage(result){
                    if (result.store_front_url_abs) {
                        // this is a store with no image
                        if (_.includes(result.store_front_url_abs,'missing')) {
                            return true
                        }
                    } else if (result.store && _.includes(result.store.store_front_url_abs,'missing')) {
                        return true
                    }
                    return false
                },
                checkPromoImage(result) {
                    if (result.promo_image_url_abs) {
                        // this is a store with no image
                        if (_.includes(result.promo_image_url_abs,'missing')) {
                            return false
                        } else {
                            return true
                        }
                    }
                },
                checkEventImage(result) {
                    if(result.event_image_url_abs){
                        // this is a store with no image
                        if (_.includes(result.event_image_url_abs,'missing')) {
                            return true
                        } else {
                            return false
                        }
                    }
                }
            }
        });
    });
</script>