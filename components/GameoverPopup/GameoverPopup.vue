<template>
	<view>
		<uni-popup ref="gPopup" :animation="false" :is-mask-click="false">
			<view class="counter">
				<view class="game-over">游戏结束</view>
				<view class="game-over-icon">
					<uni-icons type="refreshempty" size="30" color="#ffffff" @click="renewGame"></uni-icons>
				</view>
			</view>
		</uni-popup>

	</view>
</template>

<script>
	export default {
		name: "GameoverPopup",
		data() {
			return {

			};
		},
		methods: {
			open() {
				this.$refs.gPopup.open()
			},
			close() {
				this.$refs.gPopup.close()
			},
			renewGame() {
				this.close()
				this.$emit('renewGame')
			}
		}
	}
</script>

<style lang="scss">
	.counter {
		margin-bottom: 150rpx;
		background-color: #8ce2e2;
		box-shadow: 0.15rem 0.15rem 0.4rem 0 #1d7b7b;
		border-radius: 5rpx;
		color: #ffffff;
		padding: 30rpx;
		display: grid;
		grid-template-columns: 300rpx;
		grid-template-rows: 80rpx 80rpx;
		gap: 30rpx;

		.game-over,
		.game-over-icon {
			display: grid;
			place-items: center;
			font-size: 1.5rem;
		}

		.game-over-icon {
			cursor: pointer;
		}
	}
</style>