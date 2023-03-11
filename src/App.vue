<template>
  <div id="app">
	<!-- 背景音乐播放/暂停按钮 -->
		<img src="image/play.png" class="play" @click="playMusic && bk_play()"/>
	<!-- 气球小猪 -->
		<BallPig v-for="bp in bps" 
			:unit="bp.unit_name" 
			:key="bp.unit_name"
			:image="bp.image"
			:style="{
					position: 'absolute',
					display: 'block',					
					left:`${bp.x}px`, 
					top:bp.y+'px'
			}"
		></BallPig>
		<!-- 歼击机 -->
		<FighterPlane 
			:unit="fp.unit_name"
			:line_if="line_if"
			:style="{
					position: 'absolute',
					display: 'block', 
					left:fp.x+'px',
					top:fp.y+'px'
			}"
		/>
		<!-- 飞镖子弹 -->
			<FlyDarts v-for="(fd,index) in flydarts" :key="index" :style="{left:fd.x+'px',top:fd.y+'px',position: 'absolute',display: 'block'}"/>
		<!-- 过关信息展示 -->
			<ShowScore :score="live_num" v-show="disp"/>
			<ShowScore2 :score="live_num" :round_num="round_num" :total_num="total_num" v-show="disp2" />

	</div>
</template>

<script>
	import BallPig from "./components/BallPig.vue";
	import FighterPlane from "./components/FighterPlane.vue";
	import FlyDarts from "./components/FlyDarts.vue";
	import ShowScore from "./components/ShowScore.vue";
	import ShowScore2 from "./components/ShowScore2.vue";

	export default {
		name: 'App',
		components:{BallPig,FighterPlane,FlyDarts,ShowScore,ShowScore2},
		data(){
			return {
				scr_w:document.documentElement.clientWidth,	 //屏幕宽度
				scr_h:document.documentElement.clientHeight, //屏幕高度
				// scr_w: window.innerWidth? window.innerWidth:((document.body) && (document.body.clientWidth))?document.body.clientWidth:null,
				// scr_h: window.innerHeight? window.innerHeight:((document.body) && (document.body.clientHeight))?document.body.clientHeight:null,

				round_num:0,				//轮次数
				level_num:0,				//关卡数
				live_num:7,					//当前关中活着的气球数
				total_num:0,				//总得分

				units_us:['km','m','dm','cm','mm','μm','nm'],								//单位数组-英文
				units_cn:['千米','米','分米','厘米','毫米','微米','纳米'],		//单位数组-中文

				//气球小猪相关数据项
				bps:[],																				//气球小猪数组
				speed:[0.1,0.3,0.15,0.2,0.4,0.25,0.35],				//各个气球小猪上升的速度
				speed_x:[0.1,-0.3,0.15,-0.2,0.4,0.25,-0.35],
				bp_img:"image/ballute.png",										//气球小猪图片
				bp_img_die:"image/ballute_die.png",						//破裂气球图片
				bp_sou: new Audio(), 													//小猪的叫声
				bps_sou:new Audio(),													//小猪群叫的声音
				pass_sou:new Audio(),													//过关声音

				bk_sou: new Audio(),	//背景音乐对象
				playMusic:true,				//用于解除背景音乐播放按钮的单击事件
				vic_sou:new Audio(),	//过关进入下一轮的声音
				line_if:false,					//是否显示瞄准线
				disp:false,						//分数显示控制
				disp2:false,						//过关信息显示控制

				//歼击机相关数据项
				// fp_coord:{x:document.documentElement.clientWidth/2-31,y:30},
				fp:{x:document.documentElement.clientWidth/2-31,y:30,unit_name:'千米',unit_index:0},
				rdn_fp:[0,1,2,3,4,5,6],	//用于打乱歼击机单位的随机数组

				//子弹相关数据
				flydarts:[],
				// thrum_sou: document.createElement("audio"), //子弹发射的声音对象
				thrum_sou: new Audio(), //子弹发射时的声音对象

				//动画ID
				anim_id:null,
				anim_if:false				
			}
		},
		computed: {
			//气球之间的间距
			interval:function(){
				return (this.scr_w-62)/6
			}
		},
		created() {
			//设置声音
			this.thrum_sou.src = "sound/feibiao.wav";	//子弹发射的声音
			this.bp_sou.src = "sound/wa.mp3";					//气球被击中小猪的叫声
			this.bps_sou.src = "sound/wa7.mp3";				//击中首领后小猪群体的叫声
			this.pass_sou.src = "sound/pass.wav"
			this.bk_sou.src = "sound/bk.mp3";					//背景音乐来源
			this.bk_sou.loop=true;										//设置背景音乐循环播放
			this.vic_sou.src="sound/vic1.mp3";				//过关进入下一轮声音

			//初始化气球小猪
			for(var i=0;i<7;i++){
				this.bps.push({
					x:this.interval*i,
					y:this.scr_h-150,
					image:this.bp_img,
					unit_name:this.units_us[i],
					unit_index:i,
					speed:this.speed[i],
					speed_x:this.speed[i]
				})
			}

			//给气球小猪数组以及歼击机的数据对象上设置一个units属性记录当前使用的是中文单位还是英文单位
			this.bps.units=this.units_us;
			this.fp.units=this.units_cn;
		},
		watch: {
			anim_if:function(){
				if(this.anim_if){
						window.cancelAnimationFrame(this.anim_id);
						window.removeEventListener("keydown",this.moveAircraft);
						this.bk_sou.pause();	//暂停背景音乐
						this.playMusic=false;	//解除背景音乐播放按钮的单击事件
				}
			}
		},
		mounted() {
			//当调整界面时重新获取屏幕的宽高，从而从新布局
			window.onresize = ()=>{
				// let new_scr_w=window.innerWidth? window.innerWidth:((document.body) && (document.body.clientWidth))?document.body.clientWidth:null;//获取屏幕新的宽度
				// let new_scr_h= window.innerHeight? window.innerHeight:((document.body) && (document.body.clientHeight))?document.body.clientHeight:null;//获取屏幕新的高度
				if(!this.timer){	//使用节流机制，降低函数被触发的频率
					this.timer=true;
					setTimeout(()=>{
						let new_scr_w=document.documentElement.clientWidth;
						let new_scr_h=document.documentElement.clientHeight;
						this.fp.x=this.fp.x/(this.scr_w-62)*(new_scr_w-62);			//设置新窗口下歼击机的x坐标
						this.fp.y=this.fp.y/(this.scr_h-72)*(new_scr_h-72);			//设置新窗口下歼击机的y坐标
						for(let i=0;i<this.bps.length;i++){									//设置新窗口下各个气球的坐标
							this.bps[i].x=this.bps[i].x/(this.scr_w-62)*(new_scr_w-62);	//新的x坐标
							this.bps[i].y=this.bps[i].y/(this.scr_h-150)*(new_scr_h-150);//新的y坐标
						}
						for(let i=0;i<this.flydarts.length;i++){		//设置子弹的新坐标
							this.flydarts[i].x=this.flydarts[i].x/(this.scr_w-31)*(new_scr_w-31);
							this.flydarts[i].y=this.flydarts[i].y/this.scr_h*new_scr_h;
						}

						this.scr_w = document.documentElement.clientWidth;//获取屏幕新的宽度
						this.scr_h = document.documentElement.clientHeight;//获取屏幕新的高度

						this.timer=false;
					},400)					
				}	

			};

			this.$nextTick(function(){this.animator()});				//调用animator函数，开始动画，开始游戏		

			//键盘事件，移动歼击机
			window.addEventListener("keydown",this.moveAircraft, false);
		},
		methods:{
			animator(){		
				//气球运动
				for(var i=0;i<7;i++){
					this.bps[i].y -= this.bps[i].speed;
					if(this.round_num>=4){//从第5轮起开始飘动
						this.bps[i].x += this.bps[i].speed_x;//水平移动
						if(this.bps[i].x>=this.scr_w-62 || this.bps[i].x<=0){this.bps[i].speed_x=-this.bps[i].speed_x}//左右边界检测
					}
					if(this.bps[i].y<=0 || this.touch_det(this.fp,this.bps[i],62,62,72,192)){this.anim_if=true;}//停止动画
				}
				// 删除超出边界的子弹
				for (let i = 0; i < this.flydarts.length; i++){
					if(this.flydarts[i].y>=this.scr_h){
							this.flydarts.splice(i,1)
					}
				}
				//子弹向下飞
				for(var j=0;j<this.flydarts.length;j++){
					this.flydarts[j].y += 10;
					for(var k=0;k<this.bps.length;k++){
						if(this.touch_det(this.flydarts[j],this.bps[k],10,62,40,192)){//子弹是否击中了气球
							if(this.fp.unit_index===this.bps[k].unit_index){	//打中了同名单位的气球								
								for(let i=0;i<this.bps.length;i++){
									this.bps[i].image=this.bp_img_die;	//设置破裂气球图像
									this.bps[i].speed = -8;							//改变运动方向及速度的大小
								}
								
								this.flydarts.length=0;							//删除所有子弹
								this.total_num+=this.live_num;			//把玩家该关的得分加入总分中
								this.bps_sou.play();									//小猪群叫声
								this.pass_sou.play();									//过关声音
								
								if(++this.level_num>=7){
									this.vic_sou.play();    //播放过关进入下一轮的声音
									this.round_num++;				//轮次增加
									this.disp2=true;							//显示过关信息
									setTimeout(() => {
										this.passAbarrier();//进入下一关。。。。。。	
									}, 5000);
								}else{
									this.disp=true;							//显示得分分数
									setTimeout(() => {
										this.passAbarrier();//进入下一关。。。。。。	
									}, 1000);
								}

								break;
							}else{
								//打中的不是同名单位的气球
								this.bps[k].image=this.bp_img_die;//设置破裂气球图像
								this.bps[k].speed = -8;						//改变运动方向及速度的大小
								this. bp_sou.play();							//播放小猪的叫声
								this.live_num--;									//减小当前关剩下的有效气球数

								this.flydarts.splice(j--,1);
								break;
							}
						}
					}				
				}
				this.anim_id=window.requestAnimationFrame(this.animator);//requestAnimationFrame() 他的作用就是代替定时器做更加流畅高性能的动画，做可以匹配设备刷新率的动画，他解决了定时器做动画时间间隔不稳定的问题
			},
			//碰撞检测(外接矩形判定法）
			touch_det(A,B,wa,wb,ha,hb) {
					return (A.x+wa >= B.x) && (A.x <= B.x+wb) && (A.y+ha >= B.y) && (A.y <= B.y+hb)
			},
			//背景音乐播放控制
			bk_play(){
				if(this.bk_sou.paused){
					this.bk_sou.play();				//播放背景音乐
				}else{
					this.bk_sou.pause();		//暂停背景音乐的播放
				}
			},
			//关卡控制
			passAbarrier(){
				this.disp=false;		//隐藏本关得分牌
				this.disp2=false;		//隐藏过关信息
				// this.total_num+=this.live_num;			//把玩家该关的得分加入总分中
				// this.level_num++;			//下一关卡数
				this.live_num=7;			//下一关的有效气球数
				this.shuffle(this.speed);		//打乱速度
				this.shuffle(this.speed_x);		//打乱水平速度

				if(this.level_num>=7){			//关卡数大于等于7说明本轮已打完，应进入下一轮
					// this.round_num++;				//轮次增加
					this.level_num=0;				//重置为一轮7关中的第一关
					//交换单位
					let temp=this.fp.units;		
					this.fp.units=this.bps.units;	//交换单位
					this.bps.units=temp;
					//console.log("第",this.round_num+1,"关");
					if(this.round_num>=2){	
						//打乱供歼击机使用的中间数组rdn_fp，以使歼击机的单位随机出现
						this.shuffle(this.rdn_fp);

						if(this.round_num>=5){//第6轮起加快气球上升速度
							for(let i=0;i<this.speed.length;i++){
								this.speed[i]+=0.1;
							}
						}
					}
					//把每个气球的单位名换成另一套单位名
					for(let i=0;i<7;i++){
						this.bps[i].unit_name=this.bps.units[this.bps[i].unit_index];
					}
					// this.vic_sou.play();    //播放过关进入下一轮的声音
					if(this.round_num>=4 && !this.line_if){
						this.line_if=true;	//从第五轮起显示歼击机的瞄准线
						this.bk_sou.play();	//打开背景音乐
					}
				}				
				
				//重置歼击机数据
				this.fp.unit_name=this.fp.units[this.rdn_fp[this.level_num]];
				this.fp.unit_index=this.rdn_fp[this.level_num];

				//打乱气球小猪数组
				if(this.round_num>=3){						
						this.shuffle(this.bps);
				}
				//重置气球小猪的相关数据
				for(let i=0;i<this.bps.length;i++){
					this.bps[i].x=this.interval*i;
					this.bps[i].y=this.scr_h-150;
					this.bps[i].image=this.bp_img;
					// this.bps[i].unit_name=this.bps.units[i];
					// this.bps[i].unit_name=this.bps.units[this.bps[i].unit_index];
					// this.bps[i].unit_index=i;
					this.bps[i].speed=this.speed[i];
				}
			},
			//随机打乱数组
			shuffle(arr) {
				for (let i = 0; i < arr.length; i++) {
					const j = this.getRandomNum(0, i + 1);
					const temp = arr[i];
					arr[i] = arr[j];
					arr[j] = temp;
				}
			},
			// 产生从min到max之间的随机数,但不包含max
			getRandomNum(min, max) {
				return Math.floor(Math.random() * (max - min) + min);
			},
			//歼击机移动
			moveAircraft(e){
				switch (e.keyCode) {
					case 32: //按空白键产生子弹并存入数组
						this.flydarts.push({x:this.fp.x+26,y:this.fp.y+30})//发射的子弹存入flydarts数组中，存入数组的是此子弹的x、y坐标值
						this.thrum_sou.play();//播放子弹发射的声音
					break;
					case 37:
					case 65:	//向左
						switch (true){
							case this.round_num>=11:
								this.fp.x=this.fp.x<-52?this.scr_w-10:this.fp.x-31;
								break;
							case this.round_num>=4:
								this.fp.x=this.fp.x-31<=0?0:this.fp.x-31;
								break;
							default:
								this.fp.x=this.fp.x-this.interval<=0?0:this.fp.x-this.interval;
						}
						break;
					case 38:
					case 87:	//向上
						this.fp.y -= 100;
						if (this.fp.y <= 30) {
							this.fp.y = 30
						}
						break;
					case 39:
					case 68:	//向右
						switch (true){
							case this.round_num>=11:
								this.fp.x=this.fp.x>this.scr_w-10?-52:this.fp.x+31;
								break;
							case this.round_num>=4:
								this.fp.x=this.fp.x>=this.scr_w-62?this.scr_w-62:this.fp.x+31;
								break;
							default:
								// this.fp.x += this.interval;
								this.fp.x=this.fp.x>=this.scr_w-62?this.scr_w-62:this.fp.x+this.interval;
						}
						break;
					case 40:
					case 83:	//向下
						this.fp.y += 50;
						if (this.fp.y >= this.scr_h - 72) {
							this.fp.y = this.scr_h - 72
						}
						break;
				}
			}
		},
		destroyed() {
			window.onresize=null;
		},
	}
</script>

<style>
	* {
		margin: 0;
		padding: 0;
	}
	html {
		height: 100%;
		width: 100%;
		background-image: url(../public/image/timg.jpg);
		background-size: 100% 100%;
		overflow: hidden;
	}
	.play{
		/* position: 'absolute'; */
		float:left;
		margin-top:35px;
		width:40px;
		height:40px;
		cursor:pointer		/* 鼠标移上时变成手型 */
	}
</style>
