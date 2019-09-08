<template>
	<div class="item" @mousedown.prevent="bindMouseDown" :style="{backgroundColor:isShowContent ? '#fff' : ''}">
		<span v-show="isShowContent">
			{{text | fliterText}}
		</span>
		<img src="../assets/flag.png" width="24" v-if="isShowFlag" />
	</div>
</template>

<script>
	export default {
		name: 'sweep-item',
		props: {
			isShowContent: Boolean,
			isShowFlag: Boolean,
			text: Number,
			pos: {
				type: Array,
				default: () => []
			}
		},
		filters:{
			fliterText(num){
				return num === 0 ? '' : num === 99 ? '*' : num;
			}
		},
		methods: {
			bindMouseDown(e) {
				if(this.isShowContent){
					return;
				}
				let button = e.button;
				switch (button) {
					case 2:
						this.$emit('setFlag',{
							pos: this.pos
						});
						break;
					default:
						if (this.isShowFlag) {
							break;
						}
						this.$emit('openGrid', {
							pos: this.pos
						});
						break;
				}
			}
		}
	}
</script>

<style>
	.item {
		width: 24px;
		height: 24px;
		text-align: center;
		line-height: 24px;
		font-size: 14px;
		margin: 2px;
		border: 1px solid;
		background: #42B983;
	}
</style>
