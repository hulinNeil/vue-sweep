<template>
	<div id="app">
		<div v-if="!isShowSeep" class="input">
			<input type="number" @input="bindInputCol" placeholder="列数">
			<input type="number" @input="bindInputRow" placeholder="行数">
			<input type="number" @input="bindInputNum" placeholder="雷数">
			<button @click="bindClick">确定</button>
		</div>
		<Sweep :maxRow="row" :maxCol="col" :sweepNum="num" v-else />
	</div>
</template>

<script>
	import Sweep from './components/Sweep.vue'

	export default {
		components: {
			Sweep
		},
		data() {
			return {
				row: 0,
				col: 0,
				num: 0,
				isShowSeep: false
			}
		},
		methods: {
			bindInputCol(e){
				this.col = Number(e.target.value);
			},
			bindInputRow(e){
				this.row = Number(e.target.value);
			},
			bindInputNum(e){
				this.num = Number(e.target.value);
			},
			bindClick() {
				if (this.col * this.row * this.num !== 0) {
					this.isShowSeep = true;
				}else{
					alert('请输入正确值~。。。~')
				}
			}
		}
	}
</script>

<style>
	.input {
		display: flex;
		flex-direction: column;
		height: 100px;
		width: 200px;
		margin: 0 auto;
		justify-content: space-around;
	}
</style>
