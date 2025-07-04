<template>
	<view>
		<uni-popup ref="mPopup" :animation="false" :is-mask-click="false">
			<view class="counter">
				<view class="continue-game" :class="{'continue-game-hover': isContinueActive}"
					@touchstart="isContinueActive = true" @touchend="isContinueActive = false"
					@touchcancel="isContinueActive = false" @click="continueGame">
					继续游戏
				</view>
				<view class="new-game" :class="{'new-game-hover': isNewActive}" @touchstart="isNewActive = true"
					@touchend="isNewActive = false" @touchcancel="isNewActive = false" @click="startNewGame">
					新游戏
				</view>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		name: "MenuPopup",
		data() {
			return {
				isContinueActive: false,
				isNewActive: false
			};
		},
		methods: {
			open() {
				this.$refs.mPopup.open()
			},
			close() {
				this.$refs.mPopup.close()
			},
			continueGame() {
				this.close()
			},
			startNewGame() {
				this.close()
				this.$emit('newGame')
			}
		}
	}
</script>

<style lang="scss">
	.counter {
		margin-bottom: 150rpx;
		color: #ffffff;
		display: grid;
		grid-template-columns: 300rpx;
		grid-template-rows: 80rpx 80rpx;
		gap: 30rpx;

		.continue-game,
		.new-game {
			background-color: #8ce2e2;
			box-shadow: 0.15rem 0.15rem 0.3rem #2aacac;
			cursor: pointer;
			display: grid;
			place-items: center;
			border-radius: 5rpx;
		}

		.continue-game-hover,
		.new-game-hover {
			background-color: #578d8d;
		}
	}
</style>