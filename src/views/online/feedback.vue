<template>
	<mt-header fixed isgrey title="在线留言">
		<mt-button v-link="{path: '/home', query: {tohome: true}}" icon="arr-left" slot="left"></mt-button>
	</mt-header>
	<mt-content class="page-popup">
		<div class="page-cell online-msg-ipt-box">
			<a class="mint-cell">
				<label class="mint-cell-title">
					<span class="mint-cell-text leh-c-green">选择医生</span>
					<!--<input type="text" :value="name" :id="doctor_id" readonly/>-->
					<div class="online-msg-select-doc" @click="show_name = !show_name" v-text="name" :id="doctor_id"></div>
					<ul class="leh-select-drag-box" v-show="show_name">
						<li v-for="items in users"  @click="getName(items.name, items.id)">{{ items.name }}</li>
					</ul>
				</label>
				<div class="mint-cell-value">
					<span class="iconfont icon-wx-arr-down"></span>
				</div>
			</a>
			<a class="mint-cell">
				<label class="mint-cell-title">
					<span class="mint-cell-text leh-c-green">留言内容</span>
					<textarea class="msg-val" rows="1" placeholder="请输入你需要留言的内容" v-model="msg_val" maxlength="100"></textarea>
				</label>
				<div class="mint-cell-value"></div>
			</a>
			<a class="mint-cell">
				<label class="mint-cell-title">
					<span class="mint-cell-text leh-c-green">上传图片</span>
					<span class="leh-c-grey leh-fs-twelve">(最多仅允许上传5张照片)</span>
				</label>
				<div class="mint-cell-value"></div>
			</a>
		</div>
		<div class="online-msg-tap-box">
			<div class="online-msg-tap">

				<div class="photo-tap">
					<mt-picture>
						<!--<mt-pic-list v-for="items in picItems" :reddot="items.unread">
							<img :src="items"  @click="showPic(items)"/>
							<span class="leh-img-del-btn iconfont icon-wx-reduce" @click="delPic($event)"></span>
						</mt-pic-list>-->
						<ul class="weui_uploader_files">
							<li class="weui_uploader_file" v-for="items in picItems" track-by="$index" :reddot="items.unread">
								<img :src="items"  @click="showPic(items)"/>
								<span class="leh-img-del-btn iconfont icon-wx-reduce" @click="delPic($index)"></span>
							</li>
						</ul>
						<div class="weui_uploader_input_wrp" v-if="!isUpload" @click="addPic">
							<span class="iconfont icon-wx-camera"></span>
						</div>
						<div class="weui_uploader_input_wrp" v-if="isUpload && !isFull" @click="addPic">
							<span class="iconfont icon-wx-add"></span>
						</div>
					</mt-picture>
				</div>
			</div>
		</div>
		<div class="leh-full-btn">
			<mt-button size="large" type="green" @click="saveMsgPop">发布</mt-button>
		</div>


	</mt-content>

	<div class="page-popup">
		<mt-popup v-show="show_popup" position="top" class="mint-popup-2" :modal="false">
			<p v-text="tips"></p>
		</mt-popup>
	</div>
	<!-- 用于展示插件的容器 -->
	<div class="overlay" id="overlay"></div>
</template>
<script>
	import MtContent from '../../components/content'
	import MtHeader from '../../components/header.vue'
	import MtButton from '../../components/button.vue'
	import MtTranslate from '../../components/translate.vue'
	import MtTranslateItem from '../../components/translateItem.vue'
	import MtPopup from '../../components/popup.vue'
	import MtPicture from '../../components/picture.vue'
	import MtPicList from '../../components/picList.vue'
	import MessageBox from 'vue-msgbox'
	import {getJson, postJson, wrapPic} from 'util'
	import {pageConfig} from 'wxconfig'
	import $ from 'zepto'

	export default{
		route: {
			data ({from,next}) {

				let _self = this
				pageConfig()
				_self.isUpload = false
				_self.from_path = from.path
				next()
			}
		},

		data () {
			return{
				msg_val:'',
				tips: '',
				show_popup: false,
				from_path: '',
				show_name: 0,
				name: '', // 医生名称
				doctor_id: '', // 医生ID
				old_name: '',
				users: [], // 医生列表
				picItems: [], // 图片数组
				serverId: '', // 上传图片返回的serverId
				serverIds: [], // 存储多图的serverId
				isUpload: false, // 判断是否上传图片
				isFull: false, // 判断是否传满所有图片
			}
		},

		created () {

			let _self = this
			// 患者评价
			getJson('api/doctors/simple', '', (rsp)=>{
				_self.users = rsp
				_self.name = rsp[0].name
				_self.doctor_id = rsp[0].id
				_self.getName(_self.name, _self.doctor_id)
			},_self)
		},

		methods: {

			getName (names, ids) {
				this.name = names
				this.doctor_id = ids
				this.old_name = names
				this.show_name = 0
			},

			showPic () {

				wrapPic(this.picItems, '在线留言') // 查看图片
			},

			// 添加图片
			addPic () {

				let _self = this
				let len = _self.picItems.length
				if(len === 5 ){
					this.tips = '最多仅允许上传5张照片'
					this.show_popup = true
					return
				}
				wx.chooseImage({
					count: 5 - len, // 默认9
					sizeType: ['original', 'compressed'], // 可以指定是原图还是压缩图，默认二者都有
					sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
					success: function(res) {

						let localIds = res.localIds; // 返回选定照片的本地ID列表，localId可以作为img标签的src属性显示图片
						let picItems = _self.picItems.concat(localIds) // 合并分次上传的localId

						// 已经上传图片切换上传按钮
						if(picItems.length > 0){

							_self.isUpload = true
						}

						// 满5张隐藏上传按钮
						if(picItems.length == 5 ){

							_self.isFull = true
						}

						// 上传图片不能超过5张
						if(picItems.length > 5 ){

							this.tips = '最多仅允许上传5张照片'
							this.show_popup = true
							return
						}else{
							_self.picItems = _self.picItems.concat(localIds)
						}

					},
					fail: function (res) {
						alert(JSON.stringify(res));
					}
				});
			},

			// 保存图片
			saveMsgPop () {

				let _self = this
				if(_self.name === ''){
					MessageBox('提示', '请先绑定专属医生才能使用留言功能，如有疑问，可致电：400-966-8838')
					return
				}

				if(_self.msg_val === '') {
					_self.tips = '请填写留言内容'
					_self.show_popup = true
					return
				}

				if(_self.picItems.length == 0){

					// 上传
					let params = {
						     "drId": _self.doctor_id,
						  "content": _self.msg_val,
						"serverIds": ''
					}

					postJson('api/PatientMessages', params, (rsp, recode, msg)=>{

						if(recode == '1'){
							alert(msg)
						}else{
							_self.msg_val = ''
							_self.$route.router.go({path: '/user/noteDetail', query: {id: rsp, isclose: false}})

						}

					},_self)

				}else {

					// 上传图片
					this.uploadImages()
				}

			},

			// 上传图片
			uploadImages () {

				// 获取serverId
				let _self = this
				let localId = _self.picItems.pop();
				wx.uploadImage({
					localId: localId, // 需要上传的图片的本地ID，由chooseImage接口获得
					isShowProgressTips: 1, // 默认为1，显示进度提示
					success: function (res) {

						//	_self.serverId = res.serverId; // 返回图片的服务器端ID
						_self.serverIds.push(res.serverId)

						//其他对serverId做处理的代码
						if(_self.picItems.length > 0){

							_self.uploadImages();
						}else{

							// 上传
							let params = {
								     "drId": _self.doctor_id,
								  "content": _self.msg_val,
								"serverIds": _self.serverIds
							}

							postJson('api/PatientMessages', params, (rsp, recode, msg)=>{

								if(recode == '1'){
									alert(msg)
								}else{
									_self.msg_val = ''
									_self.serverIds.length = 0
									_self.$route.router.go({path: '/user/noteDetail', query: {id: rsp, isclose: false}})
								}

							},_self)

						}

					},
					fail: function (res) {
						alert(JSON.stringify(res));
					}
				});
			},

			// 删除图片
			delPic (ind) {
				let removePic = this.picItems[ind]
				this.picItems.$remove(removePic)

				// 已经上传图片切换上传按钮
				if(this.picItems.length == 0){

					this.isUpload = false
				}

				// 不满5张显示上传按钮
				if(this.picItems.length < 5 ){

					this.isFull = false
				}
			}
		},

		events: {
			'footer-button-event' () {

				/*let msg = $(this.$refs.msg)

				this.msg_val = msg.val()*/


			}
		},

		watch: {
			show_popup(val) {
				if (val) {
					setTimeout(() => {
						this.show_popup = false;
					}, 2000);
				}
			},
			'msg_val' (newVal){
				let msgTest = $('.msg-val')
				let scrollH = msgTest[0].scrollHeight
				msgTest.height(scrollH)
				if(!newVal){
					msgTest.height(24)
				}
			}
			/*name (val, oldVal) {

				if(val == this.old_name) {
					this.show_name = 0
				}else {
					this.show_name = 1
				}
			}*/
		},

		components: {
			MtContent,
			MtHeader,
			MtButton,
			MtTranslate,
			MtTranslateItem,
			MtPopup,
			MtPicture,
			MtPicList
		}
	}
</script>

<style>
	@import '../../assets/css/normalize.css';
	@import '../../assets/css/MPreview.mobile.css';

	.online-msg-ipt-box{overflow: hidden}
	.online-msg-ipt-box .mint-cell{padding-bottom: 10px;overflow: visible}
	.online-msg-ipt-box .mint-cell:after,.online-msg-ipt-box .mint-cell:nth-last-of-type(1):before{border: 0;}
	.online-msg-ipt-box .mint-cell:before{left: 10px;transform: scaleY(1);}
	.online-msg-ipt-box input,.online-msg-ipt-box textarea{width: 100%;margin-top: 15px;border: 0;font-size: 14px;}
	.online-msg-ipt-box .mint-cell .mint-cell-value span{margin-top: 35px;padding-left: 10px;color: #aaa;}
	.online-msg-tap .weui_cells{margin-top: 0;}
	.online-msg-tap .weui_cells:before,.online-msg-tap .weui_cells:after{border: 0;}
	.online-msg-tap .weui_uploader_input_wrp,.online-msg-tap .weui_uploader_file{margin-top: 8px;}
	.online-msg-select-doc{margin-top: 15px;font-size: 14px;height: 18px;line-height: 18px;text-overflow:ellipsis; overflow:hidden; white-space:nowrap}
	.online-msg-ipt-box textarea{line-height: 20px;overflow-y: hidden;min-height: 24px;height: 24px;border:1px solid transparent;}

</style>