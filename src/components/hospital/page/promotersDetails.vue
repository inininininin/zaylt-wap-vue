<template>
	<div class="promotersDetails" ref="promotersDetailsRef">
		<div class="nav" :style="{'padding-top':$store.state.paddingTop}">
			<div class="topNav">
				<div class="leftImg" @click="returnFn" id="navback">
					<img src="../../../assets/image/shape@3x.png" alt="">
				</div>
				<div class="rightImg">
					<img src="../../../assets/image/bianji@2x.png" alt="" @click="modifyFn">
					<img src="../../../assets/image/shanchu@2x.png" alt="" @click="deleteAlertFn">
				</div>
			</div>
			<van-popup v-model="showModify" @close="ReturnHomePageClose">
				<div class="modify">
					<div class="modifyTitle">
						<h4>编辑</h4>
					</div>
					<ul>
						<li>
							<h5>姓名:</h5>
							<input type="text" v-model="promoters.name" placeholder="请输入">
						</li>
						<li>
							<h5>号码:</h5>
							<input type="text" maxlength="11" v-model="promoters.phone" placeholder="请输入">
						</li>
						<li>
							<h5>密码:</h5>
							<input type="password" v-model="promoters.password" placeholder="请输入">
						</li>
						<li>
							<h5>确认密码:</h5>
							<input type="password" v-model="promoters.passwordConfirm" placeholder="请输入">
						</li>
						<li>
							<h5>备注:</h5>
							<input type="text" v-model="promoters.cover" placeholder="请输入">
						</li>
					</ul>
					<button @click="modifyPromotersFn">保存</button>
				</div>
			</van-popup>
			<van-dialog v-model="showDeleteAlert" show-cancel-button @confirm='deteleFn'>
				<h4>删除推广人</h4>
				<span>该推广人名下共有{{clinicNum}}门诊，是否删除</span>
			</van-dialog>
			<div class="titleNav">
				<div class="headPortrait" v-show="promotersImg">
					<img :src="promotersImg" alt="">
					<span>长按可分享和保存</span>
				</div>
				<div class="promotersAbout">
					<h4>{{promoters.name}}</h4>
					<p>编号:&nbsp;&nbsp;<span>{{promoters.phone}}</span></p>
					<p>备注:&nbsp;&nbsp;</p><span>{{promoters.cover}}</span>
				</div>
			</div>
		</div>
		<div class="cumulative"  :style="{'top': (parseInt($store.state.paddingTop.replace('px',''))+193)+'px'}">
			<h4>总共：{{clinicNum}}个门诊</h4>
			<span @click="transferPromotersShowAllFn">全部转移</span>
		</div>
		<van-popup v-model="modifyPromotersAllShow" @close="ReturnHomePageClose" overlay-class='modifyPromotersClass'>
			<div class="modifyPromoters">
				<div class="modifyPromotersTitle">
					<h5>移交门诊</h5>
					<p>将 <span>该推广人名下全部({{clinicNum}}个)门诊</span></p>
					<p>移交给:</p>
					<div class="choicePromoter" @click="choicePromoterFn">
						<h5>{{modify.name}}</h5>
						<img src="../../../assets/image/down@2x.png" alt="">
					</div>
					<button @click="modifySubmitAllFn()">确定</button>
				</div>
			</div>
		</van-popup>
		<div class="zhangwei" :style="{'padding-top':$store.state.paddingTop}"></div>
		<div class="promotersDetails_list" @scroll="handleScroll" ref="promotersDetails_list">
			<van-list  v-model="loading" :finished="finished" finished-text="没有更多了"  @load="onLoad">
				<ul>
					<!-- promotersList -->
					<li v-for="(item,inx) in promotersList" :key='inx'>
						<router-link :to="{path : '/hospital/hospital_clinicDetails' ,query :  {clinicId : item.hospitalClinicId}}">
							<div class="promotersList">
								<h4>{{item.name}}</h4>
								<img src="../../../assets/image/zhuanyi@2x.png" alt="" @click.prevent="transferPromotersShowFn(item)">
							</div>
						</router-link>
						<van-popup v-model="modifyPromotersShow" overlay-class='modifyPromotersClass' @close="ReturnHomePageClose">
							<div class="modifyPromoters">
								<div class="modifyPromotersTitle">
									<h5>移交门诊</h5>
									<p>将 <span>{{item.name}}</span></p>
									<p>移交给:</p>
									<div class="choicePromoter" @click="choicePromoterFn">
										<h5>{{modify.name}}</h5>
										<img src="../../../assets/image/down@2x.png" alt="">
									</div>
									<button @click="modifySubmitFn">确定</button>
								</div>
							</div>
						</van-popup>
						<van-dialog v-model="modify.show" show-cancel-button @confirm='modifyPromoterFn(item)'>
							<h4>门诊转移</h4>
							<span>用户{{promoters.name}}下的{{item.name}}将转移至{{modify.name}}</span>
						</van-dialog>
					</li>
				</ul>
			<div class="popup"  v-if="choicePromoterAllShow">
				<van-picker show-toolbar :columns="option" @cancel="cancel" @confirm="onConfirm"/>
			</div>
			</van-list>
		</div>
		<div class="returnTop" @click="$refs.promotersDetails_list.scrollTop=0;hospitalReturnTopPage = false;" ref="returnTopRef" v-show="hospitalReturnTopPage">
			<img src="../../../assets/image/returnTop.png" alt />
			<span>顶部</span>
		</div>
	</div>
</template>

<script>
import qs from 'qs';
export default {
	name: 'promotersDetails',
	data () {
		return {
			loading: false,		//加载
			// 加载状态结束
			finished: false,	//加载到底显示
			promoters:{			//推广人信息
				name: '',
				phone: '',
				password: '',
				passwordConfirm: '',
				cover: '',
			},
			promotersImg:'',
			promotersList:[],	//推广人列表
			page:1,		//get请求页数
			showModify : false,	//修改显示
			showDeleteAlert: false, //显示删除弹窗值
			clinicNum : 0,
			modifyPromotersAllShow	: false	,//修改推广人弹窗显示
			choicePromoterAllShow : false,	//修改推广人弹窗
			option: [],		//推广人列表
			modify:{		//修改推广人
				name:''	,
				id : '',
				value : '',
				num: '',
				show:false,	//最后确认弹窗显示值
				showAll : false,
			},
			//单个的
			modifyPromotersShow	: false	,//修改推广人弹窗显示
			query:'',
			scrollTop:0,
    		hospitalReturnTopPage:false,

		}
	},
	computed:{
	  hospitalReturnHomePage: {
	  	get: function() {
	  	// 
	  		return this.$store.state.hospitalReturnHomePage
	  	},
	  	set: function (newValue) {
	  	this.$store.state.hospitalReturnHomePage = newValue;
	  	},
	  },
	},
	components:{
		
	},
	created(){
		
	},
  
	mounted(){
		// 获取推广人信息
		// this.$axios.get('/hospital/def/hospital-operator-user/'+this.$route.query.hospitalUserId)
		// .then(res => {
		// 	if(res.data.codeMsg){
		// 		this.$toast(res.data.codeMsg)
		// 	}else{
		// 		this.promoters = {name: res.data.data.name,phone: res.data.data.phone,cover: res.data.data.cover},
		// 		this.modify.name = res.data.data.name;
		// 		this.modify.id = res.data.data.hospitalUserId;
		// 	}
		// })
		// .catch((err)=>{
			
		// })
		// this.$axios.get('/hospital/super-admin/hospital-clinics-sum?'+qs.stringify({hospitalUserId:this.$route.query.hospitalUserId}))
		// .then(res => {
		// 	res.data.codeMsg?	this.$toast(res.data.codeMsg) : this.clinicNum = res.data.data.rowCount;
		// })
		// .catch((err)=>{
			
		// })
		// // 加载dom节点后,获取推广人列表请求
		// this.$axios.get('/hospital/def/hospital-operator-users?')
		// .then(res => {
		// 	if(!res.data.codeMsg){
		// 		// 
		// 		for(let i in res.data.data.rows){
		// 			this.option.push({
		// 				'clinicPromoterId' : res.data.data.rows[i].hospitalUserId,
		// 				'text' : res.data.data.rows[i].name,
		// 				'value' : '00'+i,
		// 			})
		// 		}
				
		// 	}
		// })
		// .catch((err)=>{
			
		// })
	},
	activated() {
		if(this.query != JSON.stringify(this.$route.query)){
			Object.assign(this.$data, this.$options.data());
			this.query = JSON.stringify(this.$route.query);
			if(window.plus){
				//plus.navigator.setStatusBarBackground("#ffffff");
				plus.navigator.setStatusBarStyle("dark")
			}
			this.$axios.get('/hospital/def/hospital-operator-user/'+this.$route.query.hospitalUserId)
			.then(res => {
				if(res.data.codeMsg){
					this.$toast(res.data.codeMsg)
				}else{
					this.promoters = {name: res.data.data.name,phone: res.data.data.phone,cover: res.data.data.cover},
					this.modify.name = res.data.data.name;
					this.modify.id = res.data.data.hospitalUserId;
				}
			})
			.catch((err)=>{
				
			})
			this.$axios.get('/hospital/super-admin/hospital-clinics-sum?'+qs.stringify({hospitalUserId:this.$route.query.hospitalUserId}))
			.then(res => {
				res.data.codeMsg?	this.$toast(res.data.codeMsg) : this.clinicNum = res.data.data.rowCount;
			})
			.catch((err)=>{
				
			})
			// 加载dom节点后,获取推广人列表请求
			this.$axios.get('/hospital/def/hospital-operator-users?')
			.then(res => {
				if(!res.data.codeMsg){
					// 
					for(let i in res.data.data.rows){
						this.option.push({
							'clinicPromoterId' : res.data.data.rows[i].hospitalUserId,
							'text' : res.data.data.rows[i].name,
							'value' : '00'+i,
						})
					}
					
				}
			})
			.catch((err)=>{
				
			})
			// this.onLoad();
		}
		if(this.scrollTop != 0){
			this.$refs.promotersDetails_list.scrollTop = this.scrollTop;
		}
	},
	methods: {
		// 滑动一定距离出现返回顶部按钮
		handleScroll() {
			this.scrollTop = this.$refs.promotersDetails_list.scrollTop || this.$refs.promotersDetails_list.pageYOffset
			if (this.scrollTop > 800) {
				this.hospitalReturnTopPage = true;
			} else {
				this.hospitalReturnTopPage = false;
			}
		},
		cancel(){
			debugger
			this.choicePromoterAllShow = false
		},
		//返回上一级
		returnFn(){
			this.$router.back(-1)
		},
		// 显示修改弹窗
		modifyFn(){
			this.showModify = true;
      this.hospitalReturnHomePage = false;
			
		},
		choicePromoterFn(){
			this.choicePromoterAllShow = true;
		},
		// 修改推广人信息
		modifyPromotersFn(){
			this.$axios.post('/hospital/def/hospital-operator-user-alter',qs.stringify({
				hospitalUserId : this.$route.query.hospitalUserId,
				name : this.promoters.name,
				cover : this.promoters.cover,
				password : this.promoters.password,
				passwordConfirm : this.promoters.passwordConfirm,
				phone : this.promoters.phone,
			}))
			.then(res=>{
				if(!res.data.codeMsg){
					this.$toast.success('操作成功');
					this.showModify = false;
				}else{
					this.$toast(res.data.codeMsg);
				}
			})
			.catch((err)=>{
				
			})
		},
		// 显示删除推广人弹窗
		deleteAlertFn(){
			this.showDeleteAlert = true
		},
		deteleFn(){
			this.$axios.post('/hospital/def/hospital-operator-user-delete?',qs.stringify({
				hospitalOperatorId : this.$route.query.hospitalUserId,
			}))
			.then(res=>{
				if(!res.data.codeMsg){
					this.$toast.success('操作成功');
					this.$router.back(-1)
				}else{
					this.$toast(res.data.codeMsg);
				}
			})
			.catch((err)=>{
				
			})
		},
		// 下拉加载被推广的医院
		onLoad(){
			this.$axios.get('/hospital/super-admin/hospital-clinics?'+qs.stringify({hospitalUserId:this.$route.query.hospitalUserId})+'&'+qs.stringify({pn:this.page})+'&'+qs.stringify({ps:10}))
			.then(res => {
				if(res.data.data.rows.length != 0){
					for(let i in res.data.data.rows){
						if(res.data.data.rows[i]){
							this.promotersList.push(res.data.data.rows[i])
						}
						// 
					}

				this.page++;
				// 加载状态结束
				this.loading = false;
				}else{
					this.loading = false;
					this.finished = true;
				}
				if(this.promotersList.length<7){
					let windowHeight = document.documentElement.clientHeight || document.body.clientHeight;
					// 
					this.$refs.promotersDetailsRef.style.height = windowHeight+ 'px'
				}
				// this.clinic.num = res.data.data.sum.totalCount;
			})
			.catch((err)=>{
				
			})
		},
		// 显示推广人选择弹窗
		transferPromotersShowAllFn(){
			// console.log(this.clinicNum)
			if(this.clinicNum){
				this.modifyPromotersAllShow = true;
     			this.hospitalReturnHomePage = false;
			}else{
				this.$toast('没有可转移的门诊')
			}
		},
    //关闭推广人选择弹窗触发弹窗
    ReturnHomePageClose(){
      this.hospitalReturnHomePage = true;
    },
		// 选择推广人确定按钮
		onConfirm(_value){
			debugger
			let promoter= this.option.find((n)=>n.value == _value.value);
			
			this.modify={
				name:promoter.text	,
				id : promoter.clinicPromoterId,
				value : promoter.value,
				show:false,	//最后确认弹窗显示值
			}
			this.modify.name = promoter.text
			// 
			this.choicePromoterAllShow = false
			// this.modify.showAll = true;
		},
		//修改确定方法的最后确认弹窗显示
		modifySubmitAllFn(){
			this.modify.showAll = true;
			this.$dialog.confirm({
			  title: '全部门诊转移',
			  message: '用户'+this.promoters.name+'名下的'+this.clinicNum+'个门诊将转移至'+this.modify.name
			}).then(() => {
			  // on confirm
				this.$axios.post('/hospital/super-admin/hospital-clinics-alter',qs.stringify({
					hospitalUserId : this.$route.query.hospitalUserId,
					hospitalUserIdNew : this.modify.id,
					expectedRowCount : this.clinicNum,
				}))
				.then(res=>{
					if(!res.data.codeMsg){
						this.$toast.success('操作成功');
						this.$router.back(-1)
					}else{
						this.$toast(res.data.codeMsg);
					}
				})
				.catch((err)=>{
					
				})
			}).catch(() => {
			  // on cancel
			});
			
		},
		//转移推广人
		modifyPromoterAllFn(){
			debugger
			this.$axios.post('/hospital/super-admin/hospital-clinics-alter',qs.stringify({
				hospitalUserId : this.$route.query.hospitalUserId,
				hospitalUserIdNew : this.modify.id,
				expectedRowCount : this.clinicNum,
			}))
			.then(res=>{
				if(!res.data.codeMsg){
					this.$toast.success('操作成功');
					this.$router.back(-1)
				}else{
					this.$toast(res.data.codeMsg);
				}
			})
			.catch((err)=>{
				
			})
		},
		modifyPromoterNoAllFn(){
			debugger
			this.modify.showAll = false;
		},
		//单个的

		// 显示推广人选择弹窗
		transferPromotersShowFn(item){
			
			this.modifyPromotersShow = true;
      this.hospitalReturnHomePage = false;
		},
		modifySubmitFn(){
			this.modify.show = true
		},
		modifyPromoterFn(item){
			this.$axios.post('/hospital/super-admin/hospital-clinics-alter',qs.stringify({
				hospitalClinicId :　item.hospitalClinicId,
				hospitalUserId : this.$route.query.hospitalUserId,
				hospitalUserIdNew : this.modify.id,
				expectedRowCount : 1,
			}))
			.then(res=>{
				if(!res.data.codeMsg){
					this.$toast.success('操作成功');
					this.$router.back(-1)
				}else{
					this.$toast(res.data.codeMsg);
				}
			})
			.catch((err)=>{
				
			})
		}
	},
}
</script>

<style scoped>
.promotersDetails{
	width: 100%;
	height: 100%;
	background-color: #F5F5F5;
}
.nav{
	width: 100%;
	height: 1.93rem;
	position: fixed;
	top:0;
	z-index: 999;
	background-color: #FFFFFF;
}
.topNav{
	width: 100%;
	height: .53rem;
	line-height: .53rem;
}
.leftImg{
	height: 100%;
	display: inline-block;
	padding-left: .16rem;
}
.leftImg img{
	width: .09rem;
	height: .15rem;
}
.rightImg{
	float: right;
}
.rightImg img{
	width: .18rem;
	height: .17rem;
}
.rightImg img:first-child{
	margin-right: .2rem;
}
.rightImg img:last-child{
	padding-right: .16rem;
}
.titleNav{
	width: 100%;
	margin-top: .1rem;
}
.headPortrait{
	padding-left: .42rem;
	float: left;
}
.headPortrait img{
	width: .88rem;
	height: .88rem;
	object-fit: cover;
	display: block;
}
.headPortrait span{
	display: block;
	font-size: .12rem;
	transform:scale(0.9);
	-webkit-transform-origin-x: 0;
	color: #999999;
}
.promotersAbout{
	width: 51.4%;
	float: left;
	padding-left: .2rem;
}
.promotersAbout h4{
	font-size: .16rem;
	font-weight: bold;
	color: #333333;
	padding-bottom: .05rem;
}
.promotersAbout p{
	font-size: .12rem;
	color: #999999;

}
.promotersAbout p span{
	color: #333333;
}
.promotersAbout p:nth-child(3){
	/* display: inline-block; */
	float: left;
	height: .32rem;
}
.promotersAbout>span:last-child{
	width: 70%;
	display: -webkit-box;
	overflow: hidden;
	text-overflow: ellipsis;
	word-wrap: break-word;
	-webkit-line-clamp: 2;
	-webkit-box-orient: vertical;
}
.cumulative{
	width: 100%;
	height: .53rem;
	line-height: .53rem;
	position: fixed;
	top: 2.03rem;
	font-size: .15rem;
	z-index: 99;
	background-color: #F5F5F5;
}
.cumulative h4{
	/* float: left; */
	display: inline-block;
	padding-left: .16rem;
}
.cumulative span{
	float: right;
	padding-right: .16rem;
	color: #2B77EF;
}
.zhangwei{
	width: 100%;
	height: 2.46rem;
}
.promotersDetails ul{
	width: 100%;
	background-color: #FFFFFF;
}
.promotersDetails ul>li{
	width: 100%;
	height: .53rem;
	line-height: .53rem;
	position: relative;
}
.promotersDetails ul li h4{
	padding-left: .18rem;
	display: inline-block;
}
.promotersDetails ul li img{
	width: .16rem;
	height: .12rem;
	position: absolute;
	right: .16rem;
	top: 0;
	bottom: 0;
	margin: auto 0rem;
}
>>>.nav>.van-popup--center{
	width: 80%;
	height: 80.27%;
	border-radius: .05rem;
}
.modify{
	height: 100%;
	width: 100%;
	position: relative;
	overflow: hidden;
}
.modify h4{
	font-size: .15rem;
	height: .46rem;
	line-height: .46rem;
	text-align: center;
	border: 1px solid #E5E5E5;
}
.modify ul{
	margin-top: .17rem;
}
.modify ul li{
	height: .63rem;
	width: 81.67%;
	margin: 0rem auto .12rem;
	line-height: normal;
}
.promotersList{
	width: 100%;
	height: 100%;
}
.modify ul li h5{
	color: #666666;
	font-size: .13rem;
	padding-bottom: .05rem;
}
.modify ul li input{
	width: 100%;
	height: .38rem;
	/* padding: 0; */
	border: 1px solid #D8D8D8;
	text-align: center;
}
.modify button{
	width: 81.67%;
	height: .4rem;
	color: #FFFFFF;
	border: none;
	border-radius: .2rem;
	background-color: #2B77EF;
	/* position: absolute; */
	/* bottom: 7%; */
	/* left: 0; */
	/* right: 0; */
	margin: .3rem 9.165% .25rem;
	font-size: .14rem;
}
.van-dialog__content>h4{
	height: .55rem;
	line-height: .55rem;
	text-align: center;
	font-size: .17rem;
	/* padding: .15rem auto; */
}
.van-dialog__content>span{
	display: block;
	margin: 0rem auto .15rem;
	text-align: center;
}
.cumulative>.modifyPromotersClass{
	width: 80%;	height: 50.27%;
}
.van-popup--center{
	width: 80%;
	height: 41.5%;
	border-radius: .05rem;
}
.modifyPromoters{
	width: 100%;
	height: 100%;
}
.modifyPromotersTitle{
	width: 100%;
	/* height: 100%; */
}
.modifyPromotersTitle h5{
	height: .46rem;
	color:#333333;
	font-weight: bold;
	line-height: .46rem;
	text-align: center;
	font-size: .145rem;
	border-bottom: 1px solid #E5E5E5;
}
.modifyPromotersTitle p{
	font-size: .135rem;
	color: #666666;
	padding-left: .28rem;
	line-height: normal;
}
.modifyPromotersTitle p:nth-child(2){
	padding-top: .17rem;
	font-size: .135rem;

}
.modifyPromotersTitle p>span{
	color: #2B77EF;
}
.modifyPromotersTitle button{
	width: 91.67%;
	height: .4rem;
	line-height: .4rem;
	position: absolute;
	bottom: 9%;
	left: 0;
	right: 0;
	margin: 0rem auto;
	border-radius: .2rem;
	background-color: #2B77EF;
	color: #FFFFFF;
	border: none;
	font-size: .145rem;
	font-weight: bold;
}
.choicePromoter{
	width: 81.67%;
	height: .4rem;
	border: 1px solid #D8D8D8;
	position: relative;
	margin: .1rem auto 0rem;
	text-align: center;
}
.choicePromoter h5{
	border: none;
	display: inline-block;
}
.choicePromoter img{
	width: .115rem;
	height: .065rem;
	position: absolute;
	right:.15rem;
	top:0;
	bottom: 0;
	margin: auto 0rem;
}
>>>.van-dialog__content{
	text-align: center;
}

>>>.van-dialog {
	z-index: 9999!important;
}

.popup {
	z-index: 9995;
	position: fixed;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	background-color: rgba(0,0,0,.7);
}
.diaLog{
	position: absolute;
	height: 2rem;
	width: 320px;
	background-color: #FFFFFF;
	top:0;
	bottom: 0;
	left: 0;
	right: 0;
	margin: auto;
	border-radius: .15rem;
	overflow: hidden;
	z-index: 9997;
}
.diaLog h4{
	height: .55rem;
	line-height: .55rem;
	text-align: center;
	font-size: .17rem;
}
.diaLog p{
	display: block;
	/* height: .55rem; */
	line-height: .35rem;
	margin: 0rem auto .15rem;
	text-align: center;
	font-size: .15rem;
}
.dialogButton{
	position: absolute;
	bottom: 0;
	height: .5rem;
	width: 100%;
}
.dialogButton button{
	border: 1px solid #ebedf0;
	background-color: #FFFFFF;
	height: 100%;
	line-height: 100%;
	width: 50%;
	font-size: 16px;
}
.dialogButton button:last-child{
	float: right;
	color: #1989fa;
	z-index: 9999;
}
>>>.van-picker{
	position: absolute;
	bottom: 0;
	width: 100%;
}
.promotersDetails_list{
	height: calc(100% - 2.46rem);
	touch-action: pan-y;
	-webkit-overflow-scrolling: touch;
	overflow: scroll;
	overflow-x: hidden;
	width: 100%;
}
</style>
