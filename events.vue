<template>
	<div v-if="dataloaded">
		<div class="page_header" v-if="pageBanner" v-lazy:background-image="pageBanner.image_url">
			<!--http://via.placeholder.com/1920x300-->
			<div class="site_container">
				<div class="header_content caps">
					<h1>{{$t("events_page.events")}}</h1>
					<h2 style="display:none;">Scroll to  view events</h2>
				</div>
			</div>
		</div>
		<div class="site_container page_content" id="events_container">
			<div v-if="events.length > 0">
        		<div class="row event_container" v-if="showMore > index" v-for="(promo, index) in promos">				
        		    <div class="col-sm-6 col-md-3 event_image_container">
						<div class="image_container details_store_image">
						    <div v-if="!checkImageURL(promo)" class="store_details_image center-block">
                                <div class="no_logo">
                                    <img class="store_img" src="//www.mallmaverick.com/system/site_images/photos/000/041/782/original/transparent_logo.png?1533845225" alt="">
                                    <h2 class="store_details_name">{{ promo.store.name }}</h2>
                                </div>    
                            </div>
                            <div v-else class="store_details_image center-block">
                                <img :src="checkImageURL(promo)" :alt="promo.name" />
                            </div>
						</div>
					</div>
					<div class="col-sm-6 col-md-9 event_dets_container">
						<h4 class="event_name caps">{{promo.name}}</h4>
						<p class="event_dates">						    
						    <span v-if="isMultiDay(promo)"><i class="fa fa-calendar"></i> {{promo.start_date | moment("MMM D", timezone)}} - {{promo.end_date | moment("MMM D", timezone)}}</span>
						    <span v-else><i class="fa fa-calendar"></i> {{ promo.start_date | moment("MMM D", timezone) }}</span>
					    </p>
						<p class="event_desc">{{promo.description_short}}</p>
						<div class="text-right col-md-6 col-sm-12 no_padding" v-if="promo">
							<router-link :to="'/events/'+ promo.slug" class="event_learn_more pull-left hvr-icon-wobble-horizontal" :aria="promo.name">
							    {{$t("events_page.read_more")}} <i class="fa fa-angle-right hvr-icon" aria-hidden="true"></i>
						    </router-link>
							<social-sharing :url="$root.shareURL('events',promo.slug)" :title="promo.title" :description="promo.body" :quote="_.truncate(promo.description, {'length': 99})" :twitter-user="$root.twitter_user" :media="promo.image_url" inline-template >
								<div class="blog-social-share pull_right">
									<div class="social_share">
										<network network="facebook">
											<i class="fa fa-facebook social_icons" aria-hidden="true"></i>
										</network>
										<network network="twitter">
											<i class="fa fa-twitter social_icons" aria-hidden="true"></i>
										</network>
									</div>
								</div>
							</social-sharing>
						</div>
					</div>
					<div class="col-sm-12">
						<hr>
					</div>
				</div>
			</div>
			<div id="no_events" class="row" v-else>
				<div class="col-md-12">
					<p>{{$t("events_page.no_event_message")}}</p>
				</div>
			</div>
			<div class="show_more"  v-if="promos && showMore < promos.length">
                <div class="pointer" @click="loadMoreItems()">{{$t("events_page.more_promos")}}</div>
            </div>
		</div>
	</div>
</template>

<style>
    .events_container .date_bar{
        /* Today: */
        background: #D3D3D3;
        height: 40px;
        line-height: 40px;
        margin: auto;
        text-align: center;
    }
    .events_container .date_bar .fa{
        cursor: pointer;
    }
    .events_container .current_date{
        color: #636363;
        padding: 0 10px;
    }
    .events_container .all_dates {
        border-bottom: 1px solid #aea99e;
    }
    .events_container .all_dates span {
        font-size: 16px;
        color: #000000;
        letter-spacing: 1.5px;
        height: 30px;
        line-height: 30px;
        padding: 0 5px;
        cursor: pointer;
    }
    .events_container .all_dates [class*="date_"]:focus, [class*="date_"]:hover { 
        background-color: #D3D3D3;
    }
    .events_container .all_dates span.active { 
        background-color: #bababa;
    }
    .events_container .promo_dets {
        border-bottom: 1px solid #aea99e;
    }
    .events_container .row.is-table-row {
        margin: 0;
    }
    .events_container .row.is-table-row [class*="col-"] {
        padding:0;
    }
    .events_container .feature_read_more {
        width : auto;
    }
    .events_container .social_share network {
        display:inline-block;
        width: 24px;
        height: 24px;
        cursor: pointer;
    }
    .events_container .social_share .social_icons{
        width : 24px;
        height : 24px;
        display:inline;
        margin: 0 2px;
    }
    
</style>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", "vue-lazy-load", "vue-paginate"], function(Vue, Vuex, moment, tz, VueMoment, Meta, VueLazyload, VuePaginate) {
        Vue.use(Meta);
        Vue.use(VueLazyload);
        Vue.use(VuePaginate);
        return Vue.component("events-component", {
            template: template, // the variable template will be injected
            props:['locale'],
            data: function() {
                return {
                    dataloaded: false,
                    pageBanner: null,
                    // paginate: ['events'],
                    incrementBy: 5,
                    showMore: 5,
                    promos: null
                }
            },
            created() {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Events Banner');
                    if (temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    } else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                    this.promos = this.events;
                    this.dataloaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedEvents',
                    'findRepoByName',
                ]),
                events() {
                    var vm = this;
                    var temp_promo = [];
                    _.forEach(this.processedEvents, function(value, key) {
                        today = moment().tz(vm.timezone);
                        webDate = moment(value.show_on_web_date).tz(vm.timezone);
                        if (today >= webDate) {
                            value.description_short = _.truncate(value.description, { 'length': 150 });

                            if (value.store != null && value.store != undefined && _.includes(value.store.store_front_url_abs, 'missing')) {
                                value.store.store_front_url_abs = vm.property.default_logo_url_black;
                            } else if (value.store == null || value.store == undefined) {
                                value.store = {};
                                value.store.store_front_url_abs =  vm.property.default_logo_url_black;
                            }
                            
                            temp_promo.push(value);
                        }
                    });
                    temp_promo = _.sortBy(temp_promo, ['created_at', 'start_date']).reverse();
                    return temp_promo;
                },
            },
            methods: {
                loadData: async function() {
                    try {
                        let results = await Promise.all([
                            this.$store.dispatch("getData", "events"), 
                            this.$store.dispatch("getData", "repos")
                        ]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                loadMoreItems() {
                    if (this.showMore <= this.promos.length) {
                        var num = this.showMore + this.incrementBy;
                        this.showMore = num;
                    }
                },
                isMultiDay(promo) {
                    var timezone = this.timezone
                    var start_date = moment(promo.start_date).tz(timezone).format("MM-DD-YYYY")
                    var end_date = moment(promo.end_date).tz(timezone).format("MM-DD-YYYY")
                    if (start_date === end_date) {
                        return false
                    } else {
                        return true
                    }
                },
                checkImageURL(value) {
                    if (_.includes(value.image_url, "missing")) {
                        if (value.store === null || value.store === undefined) {
                            return this.property.default_logo_url_black;
                        } else if (
                            value.store != null &&
                            value.store != undefined &&
                            _.includes(value.store.store_front_url_abs, "missing")
                        ) {
                            return null;
                        } else {
                            return value.store.store_front_url_abs;
                        }
                    } else {
                        return value.event_image_url_abs;
                    }
                }
            }
        });
    });
</script>