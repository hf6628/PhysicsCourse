<template>
	<span>
		<div class='sty'>得分 {{score}} 分</div>
	</span>
</template>

<script>
	export default {
		name:'ShowScore',
		props:['score']	
	}
</script>

<style scoped>
	.sty{
		width:200px;
		height:100px;
    position: absolute;
		color:rgba(255, 255, 255, 0.8);
		background-color:rgba(0, 0, 0, 0.3);
		top:50%;left:50%;transform:translate(-50%,-50%);
		font-size:30px;		
		text-align:center;
		line-height:100px
	}
	
</style>