<template>
    <div v-if="currentPage">
        <div class="page_header" v-if="pageBanner" :style="{ backgroundImage: 'linear-gradient(180deg, rgba(0,0,0,0.3) 0%, rgba(0,0,0,0.5) 61.52%, rgba(0,0,0,0.7) 100%), url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content">
					<h1>{{ currentPage.title }}</h1>
					<h2 style="display:none;">Scroll to  view page details</h2>
					<h3 style="display:none;">View page details</h3>
				</div>
			</div>
		</div>
		<div class="site_container inside_page_content page_content">
            <div class="row event_container"  v-if="accessibilityData"  v-for="promo in accessibilityData">
				<div class="col-sm-12 col-md-12 event_dets_container">
					<h4 class="event_name caps">{{ promo.notice_title }}</h4>
					<div class="event_thick_line"></div>
					<p class="event_dates">{{promo.service_completed_date | moment("MMMM D, YYYY", timezone)}}</p>
					<p class="event_desc" v-html="promo.notice_text_approved"></p>
					
				</div>
				<div class="col-sm-12">
					<hr>
				</div>
			</div>
            <div v-if="accessibilityData" style="padding-top:20px;"></div> 
            <div class="page_body description_text text_left" v-if="locale=='en-ca'" v-html="currentPage.body"></div>
            <div class="page_body description_text text_left" v-else v-html="currentPage.body_2"></div>
        </div>
        
    </div>
    <!--Pages Banner-->
</template>
<style>
    .page_title {
        /*border-top:1px solid #aea99e;*/
        border-bottom:1px solid #aea99e;
        height: 35px;
        line-height: 35px;
    }
    #pages_container img{
        width: 100%;
        height: auto;
    }
    .acc_title {
        margin-top:20px;
    }
    img {
        max-width: 100%;
    }
</style>
<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment"], function(Vue, Vuex, moment, tz, VueMoment) {
        return Vue.component("page-details-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    success_subscribe: false,
                    currentPage: null,
                    pageBanner : null,
                    accessibilityData : null
                }
            },
            props:['id', 'locale'],
            beforeRouteUpdate(to, from, next) {
                next();
                this.updatePageData(to.params.id);
            },
            created(){
               this.updatePageData(this.id);
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findRepoByName'
                ])
            },
            methods: {
                loadData: async function(id) {
                    try {
                        let results = await Promise.all([
                            this.$store.dispatch('LOAD_PAGE_DATA', {url: this.property.mm_host + "/pages/" + id + ".json"}),
                            this.$store.dispatch("getData", "repos")
                        ]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                updatePageData (id) {
                    this.loadData(id).then(response => {
                        if (response == null || response == undefined) {
                            this.$router.replace('/');
                        }
                        this.currentPage = response[0].data;
                        if (this.currentPage.title == "Thank You") {
                            this.currentPage.title = "Newsletter"
                        }
                        console.log("id", id)
                        // Add custom banners for indivial pages
                        var temp_repo = null;
                        if (_.includes(id, 'gift-cards') || _.includes(id, 'guest-services') || _.includes(id, 'community-booth') || _.includes(id, 'family-lounge')) {
                            temp_repo = this.findRepoByName('Guest Services Banner');
                        } else if (_.includes(id, 'sponsorship')) {
                            temp_repo = this.findRepoByName('Contact Banner');
                        }
                        else if( _.includes(id, 'guest-services')) {
                            temp_repo = this.findRepoByName('Guest Services Banner');
                        }
                        else if( _.includes(id, 'leasing')) {
                            temp_repo = this.findRepoByName('Leasing Banner');
                        }
                        else {
                            temp_repo = this.findRepoByName('Pages Banner');
                        }
                        
                        if(temp_repo && temp_repo.images) {
                            this.pageBanner = temp_repo.images[0];
                        } else {
                            this.pageBanner = {};
                            this.pageBanner.image_url = "";
                        }
                    });
                }
            }
        });
    });
</script>