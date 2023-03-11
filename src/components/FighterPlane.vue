<template>
	<span>
		<span class="sty">{{unit}}</span>
		<img :src="image"/>
		<div v-if="line_if" class="sty2"></div>
	</span>
</template>

<script>
	export default {
		name:'FighterPlane',
		props: ['unit','line_if'],
		data() {
			return {
				image: "image/warcraft.png"
			}
		}
	}
</script>

<style scoped>
	.sty{
		position: absolute;
		top:-20px;
		left:15px;
		color:white
	}
	.sty2{
		position: absolute;
		top:80px;
		left:31px;
		width:1px;
		height:1000px;
		border-left:1px dashed rgba(255, 255, 255, 0.3)
	}
</style>