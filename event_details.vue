<template>
	<div class="event_dets_container" v-if="currentEvent">
		<div class="page_header" v-if="pageBanner" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content caps">
					<h1>{{$t("events_page.events")}}</h1>
					<h2 style="display:none;">Scroll to  view event details</h2>
				</div>
			</div>
		</div>
		<div class="site_container">
			<div class="row margin_40">
				<div class="col-sm-12">
					<router-link to="/events"><i class="fa fa-angle-left"></i> {{$t("events_page.back_to_events")}}</router-link>
					<h3 class="promo_name" style="margin: 20px auto 0px;">{{ currentEvent.name }}</h3>
					<p class="promo_div_date">
					    <span v-if="isMultiDay(currentEvent)"><i class="fa fa-calendar"></i>{{currentEvent.start_date | moment("MMM D", timezone)}} - {{currentEvent.end_date | moment("MMM D", timezone)}}</span>
						<span v-else><i class="fa fa-calendar"></i>{{currentEvent.start_date | moment("MMM D", timezone) }}</span>
					</p>
					<social-sharing :url="$root.shareURL('events',currentEvent.slug)" :title="currentEvent.title" :description="currentEvent.body" :quote="_.truncate(currentEvent.description, {'length': 99})" :twitter-user="$root.twitter_user" :media="currentEvent.image_url" inline-template >
						<div class="blog-social-share" style="margin: 0 auto 20px;">
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
					<img v-if="currentEvent.image_url" class="image" :src="currentEvent.image_url" :alt="currentEvent.name"/>
					<div class="text-left promo_description" v-html="currentEvent.rich_description"></div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
    define(['Vue', 'vuex', 'moment'], function(Vue, Vuex, moment) {
        return Vue.component("event-details-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    currentEvent: null,
                    storeEvents : null,
                    storeHours : null,
                    pageBanner : null
                }
            },
            props:['id', 'locale'],
            beforeRouteUpdate(to, from, next) {
                this.currentEvent = this.findEventBySlug(to.params.id);
                    if (this.currentEvent === null || this.currentEvent === undefined) {
                        this.$router.replace('/events');
                    }
                next();
            },
            created(){
                this.loadData().then(response => {
                    this.updatecurrentEvent(this.id);
                    var temp_repo = this.findRepoByName('Events Banner');
                    if (temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    } else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                    this.events = this.event;
                });
            },
            watch: {
                currentEvent : function (){
                    if (this.currentEvent != null) {
                        if (_.includes(this.currentEvent.image_url, 'missing')) {
                            this.currentEvent.image_url = "";
                        } else {
                            this.currentEvent.image_url = this.currentEvent.image_url;
                        }
                        
                        var vm = this;
                        var temp_event = [];
                        var current_id =_.toNumber(this.currentEvent.id);
                        _.forEach(this.currentEvent.store.event, function(value, key) {
                            if(_.toNumber(value) != current_id){
                                var current_event = vm.findEventById(value);
                                current_event.description_short = _.truncate(current_event.description, { 'length': 70 });
                                temp_event.push(current_event);
                            }
                        });
                        this.storeEvents = temp_event;
                    }
                    if (this.currentEvent.store) {
                        var storeHours = [];
                        var vm = this;
                        _.forEach(this.currentEvent.store.store_hours, function (value, key) {
                            var hour = vm.findHourById(value);
                            if(hour.day_of_week === 0){
                                hour.order = 7;
                            } else {
                                hour.order = hour.day_of_week;
                            }
                            storeHours.push();
                        });
                        this.storeHours = _.sortBy(storeHours, [function(o) { return o.order; }]);;
                    }
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'processedEents',
                    'findEventBySlug',
                    'findEventById',
                    'timezone',
                    'findRepoByName',
                    'findHourById'
                ])
            },
            methods: {
                updatecurrentEvent(id) {
                    this.currentEvent = this.findEventBySlug(id);
                    if (this.currentEvent === null || this.currentEvent === undefined) {
                        this.$router.replace('/events');
                    }
                },
                isMultiDay(currentEvent) {
                    var timezone = this.timezone
                    var start_date = moment(currentEvent.start_date).tz(timezone).format("MM-DD-YYYY")
                    var end_date = moment(currentEvent.end_date).tz(timezone).format("MM-DD-YYYY")
                    if (start_date === end_date) {
                        return false
                    } else {
                        return true
                    }
                },
                loadData: async function() {
                    try {
                        let results = await Promise.all([
                            this.$store.dispatch("getData", "events"), 
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