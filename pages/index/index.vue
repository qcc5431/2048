<template>
	<view class="container">
		<!-- 菜单栏 -->
		<view class="menu-container">
			<view class="score-block">
				<text class="score-font">当前分数</text>
				<text class="score">{{score}}</text>
			</view>
			<view class="max-score-block">
				<text>最高：<text>{{maxScore}}</text></text>
			</view>
			<view class="recall" :class="{'recall-hover': isHoverActive}" @touchstart="isHoverActive = true"
				@touchend="isHoverActive = false" @touchcancel="isRecallHover = false">
				撤回
			</view>
			<view class="menu" hover-class="menu-hover">
				<text>菜单</text>
			</view>
		</view>
		<!-- 游戏栏 -->
		<view class="game-container">

		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				score: 0,
				maxScore: 0,
				isHoverActive: false
			}
		},
		onLoad() {

		},
		methods: {}
	}
</script>

<style lang="scss">
	.container {
		background-color: #279a9a;
		font-family: "Comic Sans MS", cursive;
		color: #fff;
		height: 850rpx;
		box-shadow: 0.6rem 0.6rem 0 #1d7b7b;
		display: flex;
		flex-direction: column;

		.menu-container {
			flex: 1;
			background-color: #35dada;
			padding: 0.8rem;
			display: grid;
			grid-template-columns: 1fr 1fr 1fr 1fr;
			grid-template-rows: 1fr 1fr;
			gap: 0.8rem;

			.score-block {
				grid-column: 1 / 3;
				grid-row: 1 / 3;
				position: relative;
				font-size: 1.5rem;
				padding-bottom: 10rpx;

				.score-font {}
			}

			.max-score-block {
				grid-column: 3 / 5;
				// font-size: 2.4rem;
			}

			.score-block,
			.max-score-block,
			.recall,
			.menu {
				background-color: #8ce2e2;
				display: grid;
				place-items: center;
			}

			.recall,
			.menu {
				box-shadow: 0.15rem 0.15rem 0 #2aacac;
				margin: 0 0.15rem 0.15rem 0;
				cursor: pointer;
			}

			.recall-hover,
			.menu-hover {
				background-color: #578d8d !important;

			}
		}



		.game-container {
			aspect-ratio: 1/1;
			background-color: #cbf6f7;
			display: grid;
			grid-template: repeat(4, 1fr) / repeat(4, 1fr);
			gap: 0.8rem;
			padding: 0.8rem;
		}



	}
</style>