# 接口请求
## request.js封装
```javascript
import {BASE_URL} from "../config/config.js";
import {WEIXIN_MARK} from "../config/config.js";
import Message from "./message.js";
let networkOutTip=false;//用于断网提示，避免多次弹窗
// function confirmTips({title,content,confirmCall}){
// 	uni.showModal({
// 		title:title||"",
// 		content:content||"",
// 		confirmColor:"#3B7AFF",
// 		showCancel:false,
// 		success(res) {
// 			if (res.confirm) {
// 				typeof confirmCall == "function" && confirmCall();
// 			}
// 		}
// 	})
// }
export default function(param, hideLoading) {
	var url = param.url,
		method = param.method,
		header = {},
		data = param.data || {},
		token = param.token || false,
		company = param.company || false,
		companyId = param.companyId || false,
		hideLoading = param.hideLoading || false;
	//拼接完整请求地址
	var requestUrl = BASE_URL + url;
	data['weixinMark'] = WEIXIN_MARK;
	//请求方式:GET或POST(POST需配置header: {'content-type' : "application/x-www-form-urlencoded"},)
	if (method) {
		method = method.toUpperCase(); //小写改为大写
		if (method == "POST") {
			header = {
				'content-type': "application/x-www-form-urlencoded"
			};
		} else {
			header = {
				'content-type': "application/json"
			};
		}
	} else {
		method = "POST";
		header = {
			'content-type': "application/json"
		};
	}
	if (param.header) {
		header = param.header
	}
	// 用户交互:加载圈
	if (!hideLoading) {
		Message.loading({
			mask:true
		})
	}
	// let openid=uni.getStorageSync("openid")||'testopenid'
	// data['openid']=openid;
	if (token) {
		let userToken = uni.getStorageSync("token");
		//未登录
		if (!userToken) {
			if (!hideLoading) {
				uni.hideLoading();
			}
			Message.inquiryTips({
				title:'您还没有登录',
				content:'登录之后即可体验完整功能',
				confirmText:'立即登录',
				cancelText:'暂不登录',
				confirmCall:()=>{
					uni.navigateTo({
						// #ifdef MP-WEIXIN
						url: "/pages/account/login?backMode=self"
						// #endif
						// #ifndef MP-WEIXIN
						//url: "/pages/account/loginByMobile?backMode=self"
						// #endif
					})
				}
			})
			return;
		}
		data['token'] = userToken;
		//统一处理传companyId
		let companyInfo= uni.getStorageSync('company'),
		id=companyInfo?companyInfo.id:'';
		if(company){
			data['company'] = id;
		}
		if(companyId){
			data['companyId'] = id;
		}
	}
	//网络请求     
	return new Promise((resolve,reject)=>{
		uni.request({
			url: requestUrl,
			method: method,
			header: header,
			data: data,
			success: res => {
				if (res.statusCode && res.statusCode != 200) { //api错误
					Message.confirmTips({
						title:"温馨提示",
						content: "系统开小差了，请您稍后再试"
					})
					reject(res)
					console.log("接口服务异常【"+url+"】" + res.statusCode,"，errorMsg:",res.data?res.data.message||res.data.msg:'')
					return;
				}
				let result = res.data;
				if (result.code != "200") {
					Message.confirmTips({
						content: result.msg||result.message,
						confirmCall:()=>{
							typeof param.fail == "function" && param.fail(result);
						}
					})
					// typeof param.fail == "function" && param.fail(result);
					return;
				}
				// typeof param.success == "function" && param.success(result);
				resolve(result)
			},
			fail: (err) => {
				console.log("网络请求fail:【"+url+"】" + JSON.stringify(err));
				Message.confirmTips({
					title:"温馨提示",
					content: "网络请求超时，请检查网络后重试"
				})
				// wx.onNetworkStatusChange((res) => {
				// 	//取反满足条件,跳转已经写好的网络异常页面
				// 	if(!res.isConnected){
				// 		//已经弹窗提示过一次不用弹第二次
				// 		if(networkOutTip){
				// 			return;
				// 		}
				// 		networkOutTip=true;
				// 		Message.confirmTips({
				// 			content:'请检查您的网络',
				// 			confirmCall:()=>{
				// 				networkOutTip=false;
				// 			}
				// 		})
				// 	}else{
				// 		Message.confirmTips({
				// 			content: "【"+url+"】" + err.errMsg
				// 		});
				// 	}
				// })
				// typeof param.fail == "function" && param.fail(e.data);
				reject(err)
			},
			complete: () => {
				if (!hideLoading) {
					uni.hideLoading();
				}
				// typeof param.complete == "function" && param.complete();
				// return;
			}
		});
	})
}

```
## api.js

```javascript
import Request from "@/utils/request.js";

/* 获取token
 * params={
 * 	 phone
 * }
*/
export const getToken = (params,failCall) => { return Request({
	url:"/app/applets/getToken",
	method:"",//默认method = "POST";header = {'content-type': "application/json"}
	//method:"GET",//默认method = "GET";header = {'content-type': "application/json"}
	//method:"POST",//method = "POST";header = {'content-type': "application/x-www-form-urlencoded"}
	data:params,
	token:true,//需要登录传token的接口配置token=true，参数默认加token
	auth:true,//需要认证成功之后才能调用的接口配置auth=true，参数默认加企业id（这块逻辑正在写）
	hideLoading:true,//不需要默认的接口请求时loading弹窗配置hideLoading=true
	fail:failCall//接口code!=200时的处理回调
}).then(res=>res.data) }

/* 获取字典
 * params={
 * 	 type: 准驾车型：driverClass，卸车类型：truckType
 * }
*/
export const getDicValList = params => { return Request({
	url:"/app/applets/getDicValList",
	data:params
}).then(res=>res) }//想要自行处理返回值的这么写.then(res=>res)

```

## 引用
```javascript
// api使用方法
import {getToken} from '@/api/common.js'

getToken().then(res=>{
	uni.setStorageSync('token',res.token);
})
```
