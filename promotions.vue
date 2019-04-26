<template>
	<div v-if="dataloaded">
		<div class="page_header" v-if="pageBanner" :style="{ backgroundImage: 'linear-gradient(180deg, rgba(0,0,0,0.3) 0%, rgba(0,0,0,0.5) 61.52%, rgba(0,0,0,0.7) 100%), url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content caps">
					<h1>{{$t("promos_page.promotions")}}</h1>
					<h2 style="display:none;">Scroll to  view promotions</h2>
				</div>
			</div>
		</div>
		<div class="site_container page_content" id="events_container">
			<div v-if="promos.length > 0">
				<div class="row event_container" v-if="showMore > index" v-for="(promo, index) in promos">
					<div class="col-sm-6 col-md-3 event_image_container">
						<div class="image_container details_store_image">
							<div v-if="promo.no_logo" class="store_details_image center-block">
                                <div class="no_logo">
                                    <img class="store_img" src="//www.mallmaverick.com/system/site_images/photos/000/041/782/original/transparent_logo.png?1533845225" alt="">
                                    <h2 class="store_details_name">{{ promo.store.name }}</h2>
                                </div>    
                            </div>
                            <div v-else class="store_details_image center-block">
                                <img :src="promo.image_url" :alt="promo.name" />
                            </div>
                        </div>
					</div>
					<div class="col-sm-6 col-md-9 event_dets_container">
						<h4 class="event_name caps">{{promo.name}}</h4>
						<div v-if="promo.promotionable_type == 'Store'">
						    <h4 class="event_store_name caps" >{{promo.store.name}}</h4>
						</div>
						<div class="event_thick_line"></div>
						<p class="event_dates">
						    <span v-if="isMultiDay(promo)"><i class="fa fa-calendar"></i> {{promo.start_date | moment("MMM D", timezone)}} - {{promo.end_date | moment("MMM D", timezone)}}</span>
						    <span v-else><i class="fa fa-calendar"></i> {{ promo.start_date | moment("MMM D", timezone) }}</span>
					    </p>
						<p class="event_desc">{{promo.description_short}}</p>
						<div class="text-right col-md-6 col-sm-12 no_padding" v-if="promo">
							<router-link :to="'/promotions/'+ promo.slug" class="event_learn_more pull-left hvr-icon-wobble-horizontal" :aria="promo.name">
							   {{ $t("promos_page.read_more") }} <i class="fa fa-angle-right hvr-icon" aria-hidden="true"></i>
						    </router-link>
							<social-sharing :url="$root.shareURL('promotions',promo.slug)" :title="promo.title" :description="promo.body" :quote="_.truncate(promo.description, {'length': 99})" :twitter-user="$root.twitter_user" :media="promo.image_url" inline-template >
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
					<p>{{$t("promos_page.no_promo_message")}}</p>
				</div>
			</div>
			<div class="show_more"  v-if="promos && showMore < promos.length">
                <div class="contest_btn" @click="loadMoreItems()">{{$t("promos_page.more_promos")}}</div>
            </div>
		</div>
	</div>
</template>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment"], function(Vue, Vuex, moment, tz, VueMoment) {
        return Vue.component("promos-component", {
            template: template, // the variable template will be injected
            props:['locale'],
            data: function() {
                return {
                    selectedDate: null,
                    filteredPromos:[],
                    dataloaded: false,
                    pageBanner: null,
                    promos : null,
                    incrementBy: 5,
                    showMore: 5,
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
                    
                    this.promos = this.promotions;
                    this.dataloaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedPromos',
                    'findRepoByName',
                ]),
                promotions() {
                    var vm = this;
                    var temp_promo = [];
                    _.forEach(this.processedPromos, function(value, key) {
                        today = moment().tz(vm.timezone);
                        webDate = moment(value.show_on_web_date).tz(vm.timezone);
                        if (today >= webDate) {
                            value.description_short = _.truncate(value.description, { 'length': 150 });
                            
                            if (_.isEmpty(value.store)) {
                                value.image_url = vm.property.default_logo_url_black;
                            } else {
                                if (_.includes(value.store.store_front_url_abs, 'missing')) {
                                    value.no_logo = true;
                                    value.store_name = value.store.name;
                                } else {
                                    value.image_url = value.store.store_front_url_abs;
                                }
                            }

                            temp_promo.push(value);
                        }
                    });
                    temp_promo = _.sortBy(temp_promo, ['end_date'];
                    return temp_promo;
                },
            },
            methods: {
                loadData: async function() {
                    try {
                        let results = await Promise.all([
                            this.$store.dispatch("getData", "promotions"), 
                            this.$store.dispatch("getData", "repos")
                        ]);
                        console.log(results)
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
                            return this.property.default_logo_url;
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
                        return value.promo_image_url_abs;
                    }
                }
            }
        });
    });
</script>