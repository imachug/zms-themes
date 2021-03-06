<template>
	<div class="content-container">
		<div class="content">
			<!-- New post -->
			<div class="new-post" v-if="isAuthor || siteInfo.settings.own">
				<icon name="pencil-alt" />
				<a @click="$router.navigate('admin/posts/new-post')">New post</a>
			</div>

			<div class="post" v-for="post in posts">
				<customizable scope="post" name="above-post" :post="post" />

				<div class="post-title">
					<a @click="$router.navigate(post.url)">{{post.title}}</a>
				</div>
				<div class="post-info">
					On {{(new Date(post.date)).toLocaleString()}}
					by <a @click="$router.navigate(post.userUrl)">{{post.user}}</a>
				</div>

				<customizable scope="post" name="above-post-content" :post="post" />

				<div class="post-description">
					<div v-html="post.cut"></div>

					<a @click="$router.navigate(post.url)">Read more</a>
				</div>

				<customizable scope="post" name="below-read-more" :post="post" />
			</div>

			<!-- Pagination -->
			<div class="pagination">
				<div v-for="page in pageCount" :class="{page: true, 'page-current': currentPage === page}" @click="$router.navigate(`page/${page}`)">
					{{page}}
				</div>
			</div>
		</div>
	</div>
</template>

<style lang="sass" scoped>
	@import "variables.sass"

	.content-container
		display: block
	.content
		display: block
		width: $view-width
		max-width: calc(100% - #{$hspacing})
		margin: 0 auto

	.post
		display: block
		margin: 32px 0


	.post-title
		font-size: 32px

	.post-info
		font-size: 16px
		color: #666

	.post-description
		margin: 32px 0

		font-size: 16px
		color: #222


	.new-post
		display: block
		margin: 32px 0

		font-size: 16px


	// Pagination
	.pagination
		font-size: 0
		margin-bottom: 16px
	.page
		display: inline-block
		margin-right: 8px
		padding: 4px 8px

		font-size: 16px
		color: #FFF
		background-color: #222
	.page:hover
		background-color: #333
		cursor: pointer
	.page-current
		background-color: #803
	.page-current:hover
		background-color: lighten(#803, 10%)
</style>

<script language="text/javascript">
	import {zeroAuth} from "../zero";
	import Posts from "../libs/posts.js";
	import Settings from "../libs/settings.js";
	import Users from "../libs/users.js";

	export default {
		props: [],
		name: "list",

		data() {
			return {
				siteInfo: {
					settings: {
						own: false
					}
				}
			};
		},
		mounted() {
			this.$eventBus.$on("setSiteInfo", this.setSiteInfo);
			this.$eventBus.$emit("needSiteInfo");
		},
		destroyed() {
			this.$eventBus.$off("setSiteInfo", this.setSiteInfo);
		},
		methods: {
			setSiteInfo(siteInfo) {
				this.siteInfo = siteInfo;
			}
		},

		asyncComputed: {
			async pageCount() {
				const PAGE = await Settings.get("posts_per_page", 10);

				let all = await Posts.getCount();
				return Math.ceil(all / PAGE);
			},

			async currentPage() {
				let page = this.$router.currentParams.page || "1";
				return parseInt(page);
			},

			async posts() {
				let page = this.$router.currentParams.page || "1";
				page = parseInt(page);
				page--;

				const PAGE = await Settings.get("posts_per_page", 10);
				return await Posts.getList("", page * PAGE, PAGE);
			},

			async isAuthor() {
				const auth = zeroAuth.getAuth();
				if(!auth) {
					return false;
				}

				return await Users.hasRoleByAddress(auth.address, "author");
			}
		}
	};
</script>