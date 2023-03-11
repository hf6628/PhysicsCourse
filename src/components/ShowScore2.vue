<template>
	<span>
		<div class='sty1'>
			<div class='sty2'>
				<div class="sty3">你正确地击中了{{round_num*7}}次<br>通过了第{{round_num}}关<br>总共得了 {{total_num}} 分（满分是{{round_num*7*7}}分）</div>
			</div>
		</div>
	</span>
</template>

<script>
	export default {
		name:'ShowScore2',
		props:['score','round_num','total_num']	
	}
</script>

<style scoped>
	.sty1{
		width:100%;
		height:100%;
    position: absolute;
		background-color:rgba(0, 0, 0, 0.5);		
	}
	.sty2{
		width:60%;
		height:60%;
    position: absolute;
		color:rgba(255, 255, 255, 0.8);
		background-color:rgba(0, 0, 0, 0.3);
		top:50%;left:50%;transform:translate(-50%,-50%);
		font-size:30px;		
		text-align:center;
		line-height:100px
	}
	.sty3{
		width:60%;
		height:55%;
    position: absolute;
		color:rgba(255, 255, 255, 0.8);
		background-color:rgba(0, 0, 0, 0.3);
		top:50%;left:50%;transform:translate(-50%,-50%);
		font-size:30px;		
		text-align:center;
		line-height:100px
	}
	
</style>