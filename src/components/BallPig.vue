<template>
	<span>
		<span class='sty'>{{unit}}</span>
		<img :src="image"/>
	</span>
</template>

<script>
	export default {
		name:'BallPig',
		props:['unit','image']	
	}
</script>

<style scoped>
	.sty{
		position: absolute;
		top:19px;
		left:18px;
		color:white
	}
	
</style>