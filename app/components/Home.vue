<template>
	<page actionBarHidden="true" backgroundSpanUnderStatusBar="true">
		<!-- @loaded="onLoaded" -->

		<GridLayout rows="auto,auto,*,auto" columns="auto">

			<GridLayout row="0" ref="navStatusBar" class="navStatusBar" backgroundColor="#7957D5" verticalAlignment="top" height="40"
			 width="100%" rows="auto" columns="*,auto,auto,auto">
				<Label col="0" row="0" text="Link Shortner" class="status-title"></Label>
					<!-- <Image col="1" row="0" @tap="search" horizontalAlignment="right" class="status-img"
							src="~/assets/images/search.png" />
					<Image col="2" row="0" @tap="bell" horizontalAlignment="right" class="status-img"
							src="~/assets/images/bell.png" /> -->
					<Image horizontalAlignment="right" @tap="profile" stretch="aspectFill" col="3"
							row="0" class="status-profile" src="~/assets/images/me.png" />
			</GridLayout>

			<GridLayout  row="1" ref="navTab" class="navTab" verticalAlignment="top" height="50"
					width="100%"  rows="auto" columns="auto,auto,auto">

					<GridLayout class="tabview" :class="selectedTabview==0?'active':''"
							@tap="shorten" rows="*,auto" cols="auto" col="0" row="0"
							width="33%">
							<Image v-show="selectedTabview==0" row="0" class="navIcon"
									:src="selectedTabview==0?'~/assets/images/shorten_w.png':''"/>
							<Label :class="selectedTabview==0?'active':''" row="1"
									text="Shorten" class="tabviewText"></Label>
					</GridLayout>
					<GridLayout class="tabview" :class="selectedTabview==1?'active':''"
							@tap="mylinks" rows="*,auto" cols="auto" col="1" row="0"
							width="34%">
							<Image v-show="selectedTabview == 1" row="0" class="navIcon"
									:src="selectedTabview==1?'~/assets/images/list_w.png':''"/>
							<Label :class="selectedTabview==1?'active':''" row="1"
									text="My Links" class="tabviewText"></Label>							
					</GridLayout>
					<GridLayout class="tabview" :class="selectedTabview==2?'active':''"
							@tap="aboutUs" rows="*,auto" cols="auto" col="2" row="0"
							width="33%">
							<Image v-show="selectedTabview == 2" row="0" class="navIcon"
									:src="selectedTabview==2?'~/assets/images/aboutus_w.png':''"/>
							<Label :class="selectedTabview==2?'active':''" row="1"
									text="About Us" class="tabviewText"></Label>							
					</GridLayout>
			</GridLayout>

			<GridLayout v-show="selectedTabview == 0" row="2" width="100%" backgroundColor="white">
				<StackLayout class="form">
					<Image clas="logo" src="~/assets/images/moon.png"></Image>
					<Label class="header" text="Link Shortner"></Label>

					<GridLayout v-if="!showShort" rows="auto, auto, auto">
						<StackLayout row="1" class="input-field">
							<TextField class="input" ref="originalUrl" :isEnabled="!processing" hint="Long URL" v-model="to_short"></TextField>
							<StackLayOut class="hr-light"></StackLayOut>
						</StackLayout>

						<ActivityIndicator rowSpan="3" :busy="processing"></ActivityIndicator>
					</GridLayout>

					<GridLayout v-else="showShort" rows="auto, auto, auto">
						<StackLayout row="1" class="input-field">
							<TextField class="input" ref="shortUrl" :isEnabled="!processing" hint="Long URL" v-model="shorted"></TextField>
							<StackLayOut class="hr-light"></StackLayOut>
						</StackLayout>

						<ActivityIndicator rowSpan="3" :busy="processing"></ActivityIndicator>
					</GridLayout>

					<Button v-if="!showShort" :text="'Shorten!'" :isEnabled="!processing"
                    @tap="submit" class="btn btn-primary m-t-20" backgroundColor="#7957D5"></Button>
                    <Button v-else :text="'Copy to clipboard!'" :isEnabled="!processing"
                    @tap="copyToClip" class="btn btn-primary m-t-20" backgroundColor="#7957D5"></Button>
                    <Button v-if="showShort" :text="'Try another one!'" :isEnabled="!processing"
                    @tap="tryAnother" class="btn btn-primary m-t-20" backgroundColor="#7957D5"></Button>

                    <Label text="Links that mean business" class="footerTitle"></Label>
                    <Label textWrap="true">
					  <FormattedString>
					    <Span text="Create and share trusted,powerful short links" class="footerSubtitle" />
					    <Span :text="response" />
					  </FormattedString>
					</Label>
				</StackLayout>
			</GridLayout>

			<GridLayout v-show="selectedTabview == 1" row="2" width="100%" backgroundColor="white">		
				<ScrollView v-if="links.length > 0">
					<ListView for="item in links" class="list-group">
						<v-template>
							<GridLayout class="list-group-item" rows="*" columns="auto, *">
								<!-- <Image row="0" col="0" :src="item.src" class="thumb img-circle" /> -->
								<Label row="0" col="1" :text="'https://f38a0742.ngrok.io/'+item.urlCode" />
							</GridLayout>
						</v-template>
					</ListView>
				</ScrollView>
				<Label v-else style="margin-top:20;horizontal-align:center;" text="You have zero links."></Label>
			</GridLayout>

			<GridLayout v-show="selectedTabview == 2" row="2" width="100%" backgroundColor="white">		
			</GridLayout>

			<GridLayout v-show="selectedTabview == 3" row="2" width="100%" backgroundColor="white">	
					<ProfileManager @getLinks="getLinks" @updateToken="updateToken" @logout="logout" @links="mylinks" :token="token"></ProfileManager>
			</GridLayout>

			

		</GridLayout>
</page>
</template>
<script>
	// import { SwissArmyKnife } from "nativescript-swiss-army-knife";
	import { isIOS, isAndroid } from 'tns-core-modules/platform'
	import * as http from "http";
	import ProfileManager from './ProfileManager.vue';

	const gestures = require("ui/gestures"); 
	const app = require("application");
	const dialog = require("tns-core-modules/ui/dialogs");
	const appSettings = require("tns-core-modules/application-settings");

export default {
	components: {
		ProfileManager
	},
	data() {
		return {
			lastDelY: 0,
			headerCollapsed: false,
			selectedTabview: 0,
			processing: false,
			to_short: '',
			response: '',
			folder: '',
			file: '',
			showShort: false,
			shorted: '',
			ownerId: '',
			token: '',

			links: []
		};
	},
	methods: {
		profile(){
			this.selectedTabview = 3;
		},
		bell(){
			console.log('bell');
		},
		shorten() {
			this.selectedTabview = 0;
		},
		mylinks() {
			this.selectedTabview = 1;
		},
		aboutUs() {
			this.selectedTabview = 2;
		},
		logout() {
			this.selectedTabview = 0;
			this.token = '';
			this.links = [];

			this.ownerId = '';
			appSettings.remove('ownerId');
		},

		updateToken(e){
			this.token = e;
		},

		getLinks(){
			http.request({
				url: 'https://f38a0742.ngrok.io/api/getMyLinks',
				method: "GET",
				headers: { "access-token": this.token }
			}).then(response => {
				this.links = JSON.parse(response.content);
			}, error => {
				dialog.alert({
					title: "Error",
					message: "Couldn't connect to server."
				});
			});
		},

		copyToClip(){
			var clipboard = require("nativescript-clipboard");
			clipboard.setText(this.shorted).then(function(){
				console.log("OK, copied to the clipboard");
			});
		},
		tryAnother(){
			this.to_short = '';
			this.showShort = false;
		},
		submit(){
			this.processing = true;
			if(this.to_short.length < 1){
				dialog.alert({
					title: 'Error',
					message: 'Please enter a valid URL.',
					okButtonText: 'Retry'
				});
				this.processing = false;
				return false;
			}
			http.request({
				url: "https://f38a0742.ngrok.io/api/item",
				method: "POST",
				headers: {
					"Content-Type": "application/json"
				},
				content: JSON.stringify({
					originalUrl: this.to_short,
					masterkey: this.ownerId
				})
			}).then(resp => {
				let r = JSON.parse(resp.content);
				if(r.error == 1){
					dialog.alert({
						title: "Error",
						message: r.message,
						okButtonText: "Retry"
					});
				}else{
					this.shorted = "https://f38a0742.ngrok.io/" + r.urlCode;
					if(appSettings.getString("ownerId", '') == ''){
						appSettings.setString("ownerId", r.ownerId);
					}
					this.links.push({ urlCode: r.urlCode, createdAt: r.createdAt })
					this.showShort = true;
				}

				this.processing = false;
			}, error => {
				console.log(error);
				this.processing = false;
				dialog.alert({
					title: "Error",
					message: "Error.",
					okButtonText: "Retry"
				});
			});

			
		}
	},
	created(){
		this.token = appSettings.getString('token', '');
		if(this.token.length > 1){
			http.request({
				url: 'https://f38a0742.ngrok.io/api/getMyLinks',
				method: "GET",
				headers: { "access-token": this.token }
			}).then(response => {
				this.links = JSON.parse(response.content);
			}, error => {
				dialog.alert({
					title: "Error",
					message: "Couldn't connect to server."
				});
			});
		}else{
			this.ownerId = appSettings.getString("ownerId", '');
			if(this.ownerId.length != ''){
				http.request({
					url: "https://f38a0742.ngrok.io/api/getGuestLinks?masterkey=" + this.ownerId,
					method: "GET"
				}).then(response => {
					this.links = JSON.parse(response.content);
				}, error => {
					dialog.alert({
						title: "Error",
						message: "Couldn't connect to server."
					});
				})
			}
		}
	},
	watch:{
		token(newval, oldval){
			appSettings.setString('token',newval);
			if(newval == ''){
				this.links = [];
			}
		}
	},
	mounted(){
	}
};
</script>

<style>
.tabview.active {
	border-bottom-color: white;
	border-bottom-width: 3;
	margin: 0;
	height: 50;
}
.tabviewText {
	margin-top: 15;
	margin-bottom: 5;
	font-size: 13;
	color: #d8d2d2;
	horizontal-align: center;
}
.tabviewText.active {
	margin-top: 0;
	margin-bottom: 5;
	font-weight: bold;
	color: white;
	vertical-align: center;
}
.navTab {
	background-color: #7957D5;
}
.navTabview {
	background-color: blue;
}
.status-img {
	margin-top: 4;
	margin-right: 20;
	width: 30;
	height: 30;
}
.status-profile {
	border-width: 1;
	border-color: #ffffff;
	background-color: #f1ebeb;
	border-radius: 50%;
	margin-top: 4;
	margin-right: 15;
	width: 30;
	height: 30;
}
.status-title {
	color: white;
	font-size: 18;
	margin-left: 15;
	margin-top: 8;
	horizontal-align: left;
	vertical-align: center;
}

.form {
    margin-left: 30;
    margin-right: 30;
    margin-top: 40;
    flex-grow: 2;
}

.logo {
    margin-bottom: 12;
    height: 90;
    font-weight: bold;
}

.header {
    horizontal-align: center;
    font-size: 25;
    font-weight: 600;
    margin-bottom: 30;
    text-align: center;
    color: #7957D5;
}

.input-field {
    margin-bottom: 25;
}

.input {
    font-size: 18;
    placeholder-color: #A8A8A8;
}

.input:disabled {
    background-color: white;
    opacity: 0.5;
}

.btn-primary {
    margin: 30 5 15 5;
}

.footerTitle {
	margin-top:20;
	color: #363636;
    font-size: 24;
    font-weight: 600;
    line-height: 600;
}

.footerSubtitle {
	color: #4a4a4a;
    font-size: 19;
    font-weight: 400;
    line-height: 400;
}
</style>