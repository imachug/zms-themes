<template>
	<div class="content-container">
		<div class="content">
			<!-- Edit post -->
			<div class="edit-post" v-if="post.editable || siteInfo.settings.own">
				<icon name="edit" />
				<a @click="$router.navigate(post.editUrl)">Edit post</a>
			</div>
			<!-- Delete post -->
			<div class="delete-post" v-if="post.editable || siteInfo.settings.own">
				<icon name="trash" />
				<a @click="remove">Delete post</a>
			</div>

			<customizable scope="post" name="above-post" :post="post" />

			<div class="post-title">{{post.title}}</div>

			<div class="post-info">
				On {{(new Date(post.date)).toLocaleString()}}
				by <a @click="$router.navigate(post.userUrl)">{{post.user}}</a>
			</div>

			<customizable scope="post" name="above-post-content" :post="post" />

			<div class="post-description" v-html="post.content"></div>

			<customizable scope="post" name="below-post" :post="post" />
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
		margin: 32px auto

	.post-title
		font-size: 32px
		color: #222

	.post-info
		font-size: 16px
		color: #666

	.post-description
		margin: 32px 0

		font-size: 16px
		color: #222


	.edit-post, .delete-post
		display: block

		font-size: 16px
	.edit-post
		margin-top: 32px
	.delete-post
		margin-bottom: 32px
</style>

<script language="text/javascript">
	import Posts from "../libs/posts.js";
	import {zeroPage} from "../zero";

	export default {
		props: [],
		name: "post",
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
			},

			async remove() {
				if(await zeroPage.confirm("Are you sure you want to delete this post?", "Delete")) {
					let post = await Posts.get(this.$router.currentParams.id);
					await Posts.remove(post);

					this.$router.navigate("");
				}
			}
		},

		asyncComputed: {
			post: {
				async get() {
					return await Posts.get(this.$router.currentParams.id);
				},
				default: {
					title: "",
					content: "",
					date: 0,
					userUrl: "",
					user: ""
				}
			}
		}
	};
</script>