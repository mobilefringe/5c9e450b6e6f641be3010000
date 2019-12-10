<template>
    <div v-if="currentPage">
		<div class="page_header" v-if="pageBanner" :style="{ backgroundImage: 'linear-gradient(rgba(255, 255, 255, 0.3) 0%, rgba(169, 169, 169, 0.5) 61.52%, rgba(117, 117, 117, 0.7) 100%), url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content">
					<h1>{{ currentPage.title }}</h1>
					<h2 style="display:none;">Scroll to  view page details</h2>
					<h3 style="display:none;">View page details</h3>
				</div>
			</div>
		</div>
		<div class="site_container">
		    <div class="row page_content">
                <div class="col-md-12 page_body" :class="{ sponsor: sponsorship }" v-html="currentPage.body"></div>
            </div>
        </div>
    </div>
</template>

<style>
    .sponsor img {
        width: 50%;
    }
</style>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment"], function(Vue, Vuex, moment, tz, VueMoment) {
        return Vue.component("page-details-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    pageBanner : null,
                    currentPage: null,
                    sponsorship: false
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
                            this.$store.dispatch('LOAD_PAGE_DATA', { url: this.property.mm_host + "/pages/" + id + ".json" }),
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

                        // Add custom banners for indivial pages
                        var temp_repo = null;
                        if (_.includes(id, 'gift-cards') || _.includes(id, 'guest-services') || _.includes(id, 'community-booth') || _.includes(id, 'family-lounge')) {
                            temp_repo = this.findRepoByName('Guest Services Banner');
                        } else if (_.includes(id, 'about') || _.includes(id, 'accessibility')) {
                            temp_repo = this.findRepoByName('Information Banner');
                        } else if (_.includes(id, 'sponsorship') || _.includes(id, 'thank-you') || _.includes(id, 'leasing')) {
                            if (_.includes(id, 'sponsorship')) {
                                this.sponsorship = true;
                            }
                            temp_repo = this.findRepoByName('Contact Banner');
                        }

                        if (temp_repo && temp_repo.images) {
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