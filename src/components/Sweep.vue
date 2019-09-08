<template>
	<div class="sweep" @contextmenu.prevent="">
		<div>
			<span>剩余雷数： {{sweepNum - flagNum}}</span>
			<button class="btn" @click="init">重新开始</button>
			<span>用时： {{time}}</span>
		</div>
		<div class="grid">
			<div class="cols" v-for="(row,index) in gridList" :key="'col' + index">
				<div class="row" v-for="(item,key) in row" :key="'row' + key">
					<SweepItem :pos="[key,index]" :text="item.text" :isShowFlag="item.isShowFlag" :isShowContent="item.isShowContent" @openGrid="openGrid" @setFlag="setFlag" />
				</div>
			</div>
		</div>

	</div>
</template>

<script>
	import SweepItem from './SweepItem';
	export default {
		name: 'sweep',
		components: {
			SweepItem
		},
		props: {
			maxRow: { //行数
				type: Number,
				default: 10
			},
			maxCol: { //列数
				type: Number,
				default: 10
			},
			sweepNum: { //雷数
				type: Number,
				default: 10
			}
		},
		data() {
			return {
				gridList: [],
				flagNum: 0,
				time: 0
			}
		},
		created() {
			this.init();
		},
		methods: {
			init() {
				this.sweepList = []; //雷的集合
				this.firstClick = true;
				this.flagNum = 0;
				if (this.timer) {
					clearInterval(this.timer);
					this.time = 0;
				}
				this.timer = setInterval(() => {
					this.time += 1;
				}, 1000);
				this.creatGridList();
				this.creatSweepList();
				this.setSweepNum();
			},
			creatGridList() {
				let gridList = [];
				for (let i = 0; i < this.maxCol; i++) {
					let col = [];
					for (let j = 0; j < this.maxRow; j++) {
						col.push({
							isShowContent: false,
							isShowFlag: false,
							text: 0
						});
					}
					gridList.push(col);
				}
				this.gridList = gridList;
			},
			creatSweepList() {
				let sweepList = [],
					maxRow = this.maxRow,
					maxCol = this.maxCol,
					sweepNum = this.sweepNum;
				for (let i = 0; i < sweepNum; i++) {
					let value = Math.floor(Math.random() * maxRow * maxCol);
					if (!~sweepList.indexOf(value)) {
						sweepList.push(value)
					} else {
						i--;
					}
				}
				this.sweepList = sweepList;
				sweepList.forEach(value => {
					let col = value % maxCol,
						row = Math.floor(value / maxCol);
					this.gridList[col][row].text = 99;
				});
			},
			setSweepNum() {
				this.gridList.forEach((data, col) => {
					data.forEach((value, row) => {
						if (value.text === 99) {
							return;
						}
						let num = 0;
						this.getOtherGrids([row, col]).forEach(value => {
							let otherGrid = this.gridList[value[1]][value[0]];
							if (otherGrid.text === 99) {
								num += 1;
							}
						});
						value.text = num;
					})
				})
			},
			getOtherGrids(pos) {
				let getOtherPos = [ //其他8个的位置
					[-1, -1],
					[-1, 0],
					[-1, 1],
					[0, -1],
					[0, 1],
					[1, -1],
					[1, 0],
					[1, 1]
				];
				let otherGrids = [];
				getOtherPos.forEach(value => {
					let newRow = value[0] + pos[0],
						newCol = value[1] + pos[1];
					if (newRow < 0 || newCol < 0 || newCol >= this.maxCol || newRow >= this.maxRow) {
						return;
					}
					otherGrids.push([newRow, newCol]);
				});
				return otherGrids;
			},
			openGrid(e) {
				let grid = this.gridList[e.pos[1]][e.pos[0]];
				if (grid.text !== 99 && this.firstClick) {
					this.firstClick = false;
				}
				if (grid.text === 0) {
					this.openOtherGrid(e.pos);
				} else if (grid.text === 99) {
					if (this.firstClick) {
						this.init();
						this.openGrid(e);
						return;
					}
					let maxCol = this.maxCol;
					this.sweepList.forEach(value => {
						let col = value % maxCol,
							row = Math.floor(value / maxCol);
						let grid = this.gridList[col][row];
						grid.isShowContent = true;
						if (grid.isShowFlag) {
							grid.isShowFlag = false;
						}
					});
					clearInterval(this.timer);
					alert('输啦~~');
				}
				this.gridList[e.pos[1]][e.pos[0]].isShowContent = true;
			},
			openOtherGrid(pos) {
				let otherGrids = this.getOtherGrids(pos);
				// 判断周围8个是否有雷
				let haveSweep = otherGrids.some(value => this.gridList[value[1]][value[0]].text === 99);
				otherGrids.forEach(value => {
					let otherRow = value[0],
						otherCol = value[1];
					let otherGrid = this.gridList[otherCol][otherRow];
					if (otherGrid.isShowContent) {
						return;
					}
					if (otherGrid.text !== 99) {
						otherGrid.isShowContent = true;
					}
					if (otherGrid.text === 0 && !haveSweep) {
						this.openOtherGrid([otherRow, otherCol]);
					}
				})
			},
			setFlag(e) {
				let grid = this.gridList[e.pos[1]][e.pos[0]];
				if (grid.isShowFlag === false && this.flagNum === this.sweepNum) {
					return;
				}
				grid.isShowFlag = !grid.isShowFlag;
				let num = grid.isShowFlag ? 1 : -1;
				this.flagNum += num;
				if (this.flagNum === this.sweepNum) {
					let maxCol = this.maxCol;
					let isFail = this.sweepList.some(value => {
						let col = value % maxCol,
							row = Math.floor(value / maxCol);
						return this.gridList[col][row].isShowFlag === false;
					});
					if (isFail) {
						alert('失败咯，有的旗子插错啦~~~')
					} else {
						setTimeout(() => {
							this.showRank();
						}, 1000)
					}
				}
			},
			showRank() {
				clearInterval(this.timer);
				var rank = JSON.parse(localStorage.getItem('rank'));
				if (!rank) {
					rank = {};
				}
				var name = prompt('成功啦~~请输入用户名：');
				if (!name) {
					name = '匿名';
				}
				rank[name] = this.time;
				localStorage.setItem('rank', JSON.stringify(rank));
				let names = Object.keys(rank);
				names.sort((pre, next) => {
					return rank[pre] - rank[next];
				});
				alert(`排名前五的是：${names.splice(0,5).join('、')}`);
			}
		}
	}
</script>

<style>
	.sweep {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.grid {
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.btn {
		margin: 0 20px;
	}
</style>
