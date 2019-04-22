<template>
	<div class="promo_dets_container" v-if="currentJob">
		<div class="page_header" v-if="pageBanner" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content caps">
					<h1>{{$t("jobs_page.jobs")}}</h1>
					<h2 style="display:none;">Scroll to  view job details</h2>
				</div>
			</div>
		</div>
		<div class="site_container">
		    <div class="row">
		        <div class="col-sm-12 previous_router">
    		        <i class="fa fa-angle-left"></i> <router-link to="/jobs">{{$t("jobs_page.back_to_jobs")}}</router-link>
    		    </div>
		    </div>
			<div class="row margin_40">
				<div class="col-sm-4 promo_logo_container hidden_phone">
					<div class="image_container details_store_image">
						<div v-if="currentJob.store.no_store_logo" class="store_details_image center-block">
                            <div class="no_logo">
                                <img class="store_img" src="//www.mallmaverick.com/system/site_images/photos/000/041/782/original/transparent_logo.png?1533845225" alt="">
                                <h2 class="store_details_name">{{ currentJob.store.name }}</h2>
                            </div>    
                        </div>
                        <div v-else class="store_details_image center-block">
                            <img :src="currentJob.store.store_front_url_abs" :alt="currentJob.store.name + ' Logo'" />
                        </div>
					</div>
					<div class="text-center" v-if="currentJob.store.name">
					    <h4 v-if="currentJob.jobable_type == 'Store'" class="event_store_name caps">{{ currentJob.store.name }}</h4>
						<h4 v-if="currentJob.store.phone" class="store_dets_title">{{ currentJob.store.phone }}</h4>
						<h4 v-if="currentJob.store.website" class="store_dets_title"> 
						    <a :href="'//' + currentJob.store.website" target="_blank">{{ $t("stores_page.store_website") }}</a>
					    </h4>
						<h4 v-if="storeHours.length > 0" class="store_dets_title">{{ $t("stores_page.store_hours") }}</h4>
						<ul class="store_hours_list">
							<li v-if="storeHours" v-for="hour in storeHours">
								{{ hour.day_of_week | moment("dddd", timezone) }}: {{hour.open_time | moment("h A", timezone)}} - {{hour.close_time | moment("h A", timezone)}}
							</li>
						</ul>
						<div class="contest_btn" v-if="currentJob.jobable_type == 'Store'">
							<router-link :to="'/stores/' + currentJob.store.slug"> {{ $t("stores_page.store_dets_loc") }}</router-link>
						</div>
					</div>
				</div>
				<div class="col-sm-8 promo_image_container text-left">
					<h3 class="promo_name">{{currentJob.name}}</h3>
				    <div v-if="currentJob.jobable_type == 'Store'" class="visible_phone">
					    <h4 class="event_store_name caps">{{currentJob.store.name}}</h4>
					</div>
					<p class="promo_div_date" v-if="isMultiDay(currentJob)">
					    <i class="fa fa-calendar"></i>{{currentJob.start_date | moment("MMM D", timezone)}} - {{currentJob.end_date | moment("MMM D", timezone)}}
				    </p>
					<p class="promo_div_date pull-left" v-else>
					    <i class="fa fa-calendar"></i>{{currentJob.start_date | moment("MMM D", timezone) }}
				    </p>
					<social-sharing :url="$root.shareURL('jobs',currentJob.slug)" :title="currentJob.title" :description="currentJob.body" :quote="_.truncate(currentJob.description, {'length': 99})" :twitter-user="$root.twitter_user" :media="currentJob.image_url" inline-template >
						<div class="blog-social-share" style="margin: 0 auto 15px;">
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
					<div class="text-left promo_description">
						<p v-html="currentJob.rich_description"></p>
					</div>
					<hr v-if="currentJob.message || currentJob.contact_name || currentJob.contact_email || currentJob.contact_phone || currentJob.contact_website">
                    <div v-if="currentJob.message">
                        <p>Additional Message: {{ currentJob.message }}</p>
                    </div>
                    <div v-if="currentJob.contact_name">
                        <p>Contact Name: {{ currentJob.contact_name }}</p>
                    </div>
                    <div v-if="currentJob.contact_email">
                        <p>Contact Email: {{ currentJob.contact_email }}</p>
                    </div>
                    <div v-if="currentJob.contact_phone">
                        <p>Contact Phone: {{ currentJob.contact_phone }}</p>
                    </div>
                    <div v-if="currentJob.contact_website">
                        <p>Contact Website: <a :href="currentJob.contact_website">{{ currentJob.contact_website }}</a></p>
                    </div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
    define(['Vue', 'vuex', 'moment'], function(Vue, Vuex, moment) {
        return Vue.component("job-details-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    currentJob: null,
                    storeJobs : null,
                    storeHours : null,
                    pageBanner : null
                }
            },
            props:['id', 'locale'],
            beforeRouteUpdate(to, from, next) {
                this.currentJob = this.findJobBySlug(to.params.id);
                    if (this.currentJob === null || this.currentJob === undefined){
                        this.$router.replace('/jobs');
                    }
                next();
            },
            created(){
                this.loadData().then(response => {
                    this.updateCurrentJob(this.id);
                    var temp_repo = this.findRepoByName('Jobs Banner');
                    if (temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    } else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                });
            },
            watch: {
                currentJob : function (){
                    if (this.currentJob != null) {
                        if (this.currentJob.store  && _.includes(this.currentJob.store.store_front_url_abs, 'missing')) {
                            // this.currentPromo.store.store_front_url_abs = this.property.default_logo_url;
                            this.currentJob.store.no_store_logo = true;
                        } else if (!this.currentJob.store) {
                            this.currentJob.store = {};
                            this.currentJob.store.store_front_url_abs = this.property.default_logo_url_black;
                        }
                        
                        var vm = this;
                        var temp_job = [];
                        var current_id =_.toNumber(this.currentJob.id);
                        _.forEach(this.currentJob.store.jobs, function(value, key) {
                            if(_.toNumber(value) != current_id){
                                var current_promo = vm.findJobById(value);
                                current_promo.description_short = _.truncate(current_promo.description, { 'length': 70 });
                                temp_job.push(current_promo);
                            }
                        });
                        this.storeJobs = temp_job;
                    }
                    if (this.currentJob.store) {
                        var storeHours = [];
                        var vm = this;
                        _.forEach(this.currentJob.store.store_hours, function (value, key) {
                            var hour = vm.findHourById(value);
                            if(hour.day_of_week === 0){
                                hour.order = 7;
                            } else {
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
                    'processedJobs',
                    'findJobBySlug',
                    'findJobById',
                    'timezone',
                    'findRepoByName',
                    'findHourById'
                ])
            },
            methods: {
                updateCurrentJob(id) {
                    this.currentJob = this.findJobBySlug(id);
                    if (this.currentJob === null || this.currentJob === undefined){
                        this.$router.replace('/jobs');
                    }
                },
                isMultiDay(currentJob) {
                    var timezone = this.timezone
                    var start_date = moment(currentJob.start_date).tz(timezone).format("MM-DD-YYYY")
                    var end_date = moment(currentJob.end_date).tz(timezone).format("MM-DD-YYYY")
                    if (start_date === end_date) {
                        return false
                    } else {
                        return true
                    }
                },
                loadData: async function() {
                    try {
                        let results = await Promise.all([
                            this.$store.dispatch("getData", "jobs"), 
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