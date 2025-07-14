<template>
	<view class="block" @touchstart="handleTouchStart" @touchmove.stop="handleTouchMove" @touchend="handleTouchEnd">
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
				<view class="menu" :class="{'menu-hover': isMenuActive}" @touchstart="isMenuActive = true"
					@touchend="isMenuActive = false" @touchcancel="isMenuActive = false" @click="openMenu">
					菜单
				</view>
				<view class="recall" :class="{'recall-hover': isRecallActive}" @touchstart="isRecallActive = true"
					@touchend="isRecallActive = false" @touchcancel="isRecallActive = false" @click="recall">
					撤回
				</view>
			</view>
			<!-- 游戏栏 -->
			<view class="game-container">
				<template v-for="(row, rowIndex) in matrix">
					<view class="game-cell"
						:class="cell > 0 && ['has-value',`value-${cell}`, animatedCells.has(`${rowIndex},${cellIndex}`) && 'appear-animation', cell > 512 && 'big-value']"
						v-for="(cell, cellIndex) in row" :key="`${rowIndex}${cellIndex}${cell}`">
						{{cell > 0 ? cell : ''}}
					</view>
				</template>
			</view>
		</view>

		<MenuPopup ref="menuPopup" @newGame="newGame"></MenuPopup>

		<GameoverPopup ref="gameoverPopup" @renewGame="renewGame"></GameoverPopup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				score: 0,
				maxScore: 0,
				lastScore: 0,
				isMenuActive: false,
				isRecallActive: false,
				matrix: [
					[0, 0, 0, 0],
					[0, 0, 0, 0],
					[0, 0, 0, 0],
					[0, 0, 0, 0],
				],
				lastMatrix: [
					[0, 0, 0, 0],
					[0, 0, 0, 0],
					[0, 0, 0, 0],
					[0, 0, 0, 0],
				],
				rows: 4,
				cols: 4,
				gameOver: false,
				touchStartX: 0,
				touchStartY: 0,
				minSwipeDistance: 50, // 最小滑动距离阈值
				direction: null,
				animatedCells: new Set(),
			}
		},
		onLoad() {
			this.maxScore = uni.getStorageSync('maxScore') || 0;
			this.initGame();
		},
		onHide() {
			this.saveMaxScore()
		},
		methods: {
			initGame() {
				this.gameOver = false
				// 重置分数
				this.score = 0;
				// 清空网格
				this.matrix = [
					[0, 0, 0, 0],
					[0, 0, 0, 0],
					[0, 0, 0, 0],
					[0, 0, 0, 0],
				];

				// 随机添加两个数字块
				this.addRandomTile();
				this.addRandomTile();

				this.lastMatrix = JSON.parse(JSON.stringify(this.matrix));
			},

			// 添加随机数字块，同样适用于后续移动后添加数字块
			addRandomTile() {

				// 判断是否还有空的单元格，没有则不添加
				if (!this.hasEmptyCell()) return;

				let value = Math.random() < 0.9 ? 2 : 4;
				// 记录空单元格的位置
				let emptyCells = [];

				for (const [i, row] of this.matrix.entries()) {
					for (const [j, cell] of row.entries()) {
						if (cell === 0) emptyCells.push({
							row: i,
							col: j
						});
					}
				}

				// 随机选择一个空单元格
				let randomCell =
					emptyCells[Math.floor(Math.random() * emptyCells.length)];
				this.$set(this.matrix[randomCell.row], randomCell.col, value);

				this.animatedCells.add(`${randomCell.row},${randomCell.col}`);
			},

			// 检查是否有空单元格
			hasEmptyCell() {
				for (const row of this.matrix) {
					for (const cell of row) {
						if (cell === 0) return true;
					}
				}
				return false;
			},

			getNextPosition(direction, i, j) {
				switch (direction) {
					case 'up':
						return [i + 1, j]
					case 'down':
						return [i - 1, j]
					case 'left':
						return [i, j + 1]
					case 'right':
						return [i, j - 1]
					default:
						return [i, j]
				}
			},

			move(direction) {
				if (this.gameOver) return;

				this.direction = direction

				this.animatedCells = new Set();

				this.lastMatrix = JSON.parse(JSON.stringify(this.matrix));
				this.lastScore = this.score;

				switch (direction) {
					// 向上移动，依次从第一行的四个单元格开始处理
					case "up":
						for (let i = 0; i < this.cols; i++) this.calcTileNewValue(0, i);
						break;
					case "down":
						// 向下移动，依次从第四行的四个单元格开始处理
						for (let i = 0; i < this.cols; i++) this.calcTileNewValue(this.rows - 1, i);
						break;
					case "left":
						// 向左移动，依次从第一列的四个单元格开始处理
						for (let i = 0; i < this.rows; i++) this.calcTileNewValue(i, 0);
						break;
					case "right":
						// 向右移动，依次从第四列的四个单元格开始处理
						for (let i = 0; i < this.rows; i++) this.calcTileNewValue(i, this.cols - 1);
						break;
				}

				// 重新计算最高分
				if (this.score > this.maxScore) {
					this.maxScore = this.score
				}

				// 生成一个新数据块
				if (JSON.stringify(this.lastMatrix) !== JSON.stringify(this.matrix)) {
					setTimeout(() => {
						this.addRandomTile();

						this.gameOver = this.checkGameOver();
						if (this.gameOver) {
							this.$refs.gameoverPopup.open()
						}
					}, 10);
				}
			},

			// 单元格的下一个位置
			nextTile(i, j) {
				const [ni, nj] = this.getNextPosition(this.direction, i, j);
				// 处理边界
				if (!this.inRange(ni, nj)) {
					return null;
				}
				return [ni, nj];
			},

			// 找到单元格的下一个非 0 位置
			nextTileNotZero(i, j) {
				const nextT = this.nextTile(i, j);
				if (!nextT) {
					return null;
				}
				const [ni, nj] = nextT;
				if (this.matrix[ni][nj] !== 0) {
					return nextT;
				}
				// 如果找到的下一个为 0，则继续找
				return this.nextTileNotZero(ni, nj);
			},

			// 判断是否越界
			inRange(i, j) {
				return i >= 0 && i < this.rows && j >= 0 && j < this.cols;
			},

			// 计算 某一行 / 某一列 移动后的值
			calcTileNewValue(i, j) {
				const nextTileNZ = this.nextTileNotZero(i, j);
				if (!nextTileNZ) {
					return;
				}

				const [ni, nj] = nextTileNZ;
				const currentV = this.matrix[i][j];
				const nextV = this.matrix[ni][nj];
				if (currentV === 0) {
					this.$set(this.matrix[i], j, nextV)
					this.$set(this.matrix[ni], nj, 0)
					this.calcTileNewValue(i, j);
				} else if (currentV === nextV) {
					this.$set(this.matrix[i], j, currentV * 2)
					this.$set(this.matrix[ni], nj, 0)
					this.score += currentV * 2;

					this.animatedCells.add(`${i},${j}`);

					// 合并一次了，不需要重复处理
				}

				// 然后处理下一个单元格
				const nextT = this.nextTile(i, j);
				if (nextT) {
					this.calcTileNewValue(...nextT);
				}
			},

			checkGameOver() {
				// 检查是否有空单元格
				if (this.hasEmptyCell()) return false;

				// 检查是否有相邻的相同数字
				for (let i = 0; i < this.rows; i++) {
					for (let j = 0; j < this.cols; j++) {
						// 检查右侧
						if (j < this.cols - 1 && this.matrix[i][j] === this.matrix[i][j + 1]) {
							return false;
						}
						// 检查下方
						if (i < this.rows - 1 && this.matrix[i][j] === this.matrix[i + 1][j]) {
							return false;
						}
					}
				}

				return true;
			},

			recall() {
				if (JSON.stringify(this.lastMatrix) === JSON.stringify(this.matrix)) return;
				this.matrix = JSON.parse(JSON.stringify(this.lastMatrix));

				if (this.score !== this.lastScore) {
					this.score = this.lastScore;
				}
			},

			// 处理触摸开始
			handleTouchStart(e) {
				this.touchStartX = e.touches[0].clientX;
				this.touchStartY = e.touches[0].clientY;
			},

			// 处理触摸移动（用于阻止页面滚动）
			handleTouchMove(e) {
				e.preventDefault && e.preventDefault();
			},

			// 处理触摸结束
			handleTouchEnd(e) {
				const touchEndX = e.changedTouches[0].clientX;
				const touchEndY = e.changedTouches[0].clientY;

				const deltaX = touchEndX - this.touchStartX;
				const deltaY = touchEndY - this.touchStartY;

				// 滑动距离过短时不处理
				if (Math.abs(deltaX) < this.minSwipeDistance &&
					Math.abs(deltaY) < this.minSwipeDistance) {
					return;
				}
				// 识别主滑动方向（水平或垂直）
				if (Math.abs(deltaX) > Math.abs(deltaY)) {
					// 水平滑动
					deltaX > 0 ? this.move('right') : this.move('left');
				} else {
					// 垂直滑动
					deltaY > 0 ? this.move('down') : this.move('up');
				}
			},

			saveMaxScore() {
				uni.setStorageSync("maxScore", this.maxScore)
			},

			openMenu() {
				this.$refs.menuPopup.open()
			},

			newGame() {
				this.initGame()
			},

			renewGame() {
				this.initGame()
			}
		}
	}
</script>

<style lang="scss">
	.block {
		padding: 50rpx;
		padding-top: 150rpx;
	}

	.container {
		background-color: #279a9a;
		color: #fff;
		height: 850rpx;
		box-shadow: 0 0 0.8rem 0.2rem #1d7b7b;
		display: flex;
		flex-direction: column;
		border-radius: 12rpx;
		overflow: hidden;

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
				padding: 0.3rem 0 0.5rem 0;
			}

			.max-score-block {
				grid-column: 3 / 5;
			}

			.score-block,
			.max-score-block,
			.recall,
			.menu {
				background-color: #8ce2e2;
				display: grid;
				place-items: center;
				border-radius: 10rpx;
			}

			.recall,
			.menu {
				box-shadow: 0.15rem 0.15rem 0.3rem #2aacac;
				margin: 0 0.15rem 0.15rem 0;
				cursor: pointer;
			}

			.recall-hover,
			.menu-hover {
				background-color: #578d8d;
			}
		}

		.game-container {
			aspect-ratio: 1/1;
			background-color: #cbf6f7;
			display: grid;
			grid-template: repeat(4, 1fr) / repeat(4, minmax(0, 1fr));
			gap: 0.8rem;
			padding: 0.8rem;
			font-size: 1.7rem;
			text-shadow: 0.1rem 0.1rem 0.2rem #2aacac;

			.game-cell {
				background-color: #c4efef;
				border-radius: 10rpx;
			}

			@keyframes tileAppear {
				0% {
					opacity: 0;
					transform: scale(0.5);
				}

				70% {
					opacity: 1;
					transform: scale(1.1);
				}

				100% {
					opacity: 1;
					transform: scale(1);
				}
			}

			.has-value {
				display: grid;
				place-items: center;
				box-shadow: 0.2rem 0.2rem 0.2rem #a4c5c5;

			}

			.big-value {
				font-size: 1.4rem;
			}

			.appear-animation {
				animation: tileAppear 0.3s ease-out;
			}

			.value-2 {
				background-color: #90ecec;
			}

			.value-4 {
				background-color: #00e5ff;
			}

			.value-8 {
				background-color: #0089d8;
			}

			.value-16 {
				background-color: #947efe;
			}

			.value-32 {
				background-color: #1910be;
			}

			.value-64 {
				background-color: #f0eff3;
			}

			.value-128 {
				background-color: #382a6a;
			}
		}
	}
</style>