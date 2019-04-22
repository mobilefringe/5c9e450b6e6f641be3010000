<template>
	<div class="event_dets_container" v-if="currentPromo">
		<div class="page_header" v-if="pageBanner" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content caps">
					<h1>{{ $t("promos_page.promotions") }}</h1>
					<h2 style="display:none;">Scroll to view event details</h2>
				</div>
			</div>
		</div>
		<div class="site_container">
			<div class="row margin_40">
				<div class="col-sm-12">
					<router-link to="/promotions">
					    <i class="fa fa-angle-left"></i> {{ $t("promos_page.back_to_promos") }}
					</router-link>
					<h3 class="promo_name" style="margin: 20px auto 0;">{{ currentPromo.name }}</h3>
					<p class="promo_div_date">
					    <span v-if="isMultiDay(currentPromo)"><i class="fa fa-calendar"></i>{{ currentPromo.start_date | moment("MMM D", timezone) }} - {{ currentPromo.end_date | moment("MMM D", timezone) }}</span>
						<span v-else><i class="fa fa-calendar"></i>{{ currentPromo.start_date | moment("MMM D", timezone) }}</span>
					</p>
    				<social-sharing :url="$root.shareURL('promos',currentPromo.slug)" :title="currentPromo.title" :description="currentPromo.body" :quote="_.truncate(currentPromo.description, {'length': 99})" :twitter-user="$root.twitter_user" :media="currentPromo.image_url" inline-template >
    					<div class="blog-social-share" style="margin: 0 auto 20px">
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
				<div class="col-sm-12">
					<img v-if="currentPromo.image_url" class="promo_img" :src="currentPromo.image_url" :alt="currentPromo.name"/>
					<div class="text-left promo_description" v-html="currentPromo.rich_description"></div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
    define(['Vue', 'vuex', 'moment'], function(Vue, Vuex, moment) {
        return Vue.component("promo-details-component", {
            template: template, // the variable template will be injected,
            props:['id', 'locale'],
            data: function() {
                return {
                    currentPromo: null,
                    storePromos : null,
                    storeHours : null,
                    pageBanner : null
                }
            },
            beforeRouteUpdate(to, from, next) {
                this.currentPromo = this.findPromoBySlug(to.params.id);
                    if (this.currentPromo === null || this.currentPromo === undefined){
                        this.$router.replace('/');
                    }
                next();
            },
            created(){
                this.loadData().then(response => {
                    this.updateCurrentPromo(this.id);
                    var temp_repo = this.findRepoByName('Promos Banner');
                    if(temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    }
                    else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                    this.promos = this.promotions;
                });
            },
            watch: {
                currentPromo : function (){
                    if(this.currentPromo != null) {
                        if (_.includes(this.currentPromo.image_url, 'missing')) {
                            this.currentPromo.image_url = "";
                        }

                        var vm = this;
                        var temp_promo = [];
                        var current_id =_.toNumber(this.currentPromo.id);
                        _.forEach(this.currentPromo.store.promotions, function(value, key) {
                            if (_.toNumber(value) != current_id) {
                                var current_promo = vm.findPromoById(value);
                                current_promo.description_short = _.truncate(current_promo.description, {'length': 70});
                                temp_promo.push(current_promo);
                            }
                        });
                        this.storePromos = temp_promo;
                    }
                    if(this.currentPromo.store) {
                        var storeHours = [];
                        var vm = this;
                        _.forEach(this.currentPromo.store.store_hours, function (value, key) {
                            var hour = vm.findHourById(value);
                            if(hour.day_of_week === 0){
                                hour.order = 7;
                            }
                            else {
                                hour.order = hour.day_of_week;
                            }
                            storeHours.push();
                        });
                        this.storeHours = _.sortBy(storeHours, [function(o) { return o.order; }]);
                    }
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'processedPromos',
                    'findPromoBySlug',
                    'findPromoById',
                    'timezone',
                    'findRepoByName',
                    'findHourById'
                ])
            },
            methods: {
                updateCurrentPromo(id) {
                    this.currentPromo = this.findPromoBySlug(id);
                    if (this.currentPromo === null || this.currentPromo === undefined) {
                        this.$router.replace('/promotions');
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
                loadData: async function() {
                    try {
                        let results = await Promise.all([
                            this.$store.dispatch("getData", "promotions"), 
                            this.$store.dispatch("getData", "repos")
                        ]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                }
            }
        });
    });
</script>