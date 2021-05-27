<template>
		<view class="poster">
		  <view class="poster-sc">
		  	<view class="poster-img" :style="{height:height+'px',width:width+'px'}" @touchstart="touchstartCallback"
		  				@touchmove="touchmoveCallback" 
		  				@touchend="touchendCallback">
		  		<image :src="posterPicUrl" :style="{
		  				  transform: `translate(${stv.offsetX}px,${stv.offsetY}px) scale(${stv.scale}) rotate(${ stv.rotate }deg)`,
		  				  width:originImg.width+'px',
		  				  height:originImg.height+'px'
		  	}"></image>
		  	<view class="poster-bgmask" :style="{height:height+'px',width:width+'px',backgroundImage: 'url(' + posterBgImageURL + ')','background-repeat':'no-repeat', backgroundSize:'100%'}">
		  	</view>
		  	</view>
		  </view>
		</view>
</template>

<script>
	var twoPoint = {
	  x1: 0,
	  y1: 0,
	  x2: 0,
	  y2: 0
	}
export default {
	name:'fy-poster',
	props: {
		posterPicUrl:{
			type: String,
			default() {
			    return '';
			}
		},
		posterBgImageURL:{
			type: String,
			default() {
			    return '';
			}
		}
	},
	data() {
		return {
			picUrlInfo:{},
			addHeight:false,
			ipad:false,
			originImg: {},                                //存放原图信息
			width: 0,                //剪裁框的宽度
			height: 0, //剪裁框的长度
			stv: {
			  offsetX: 0,                                   //剪裁图片左上角坐标x
			  offsetY: 0,                                   //剪裁图片左上角坐标y
			  zoom: false,                                  //是否缩放状态
			  distance: 0,                                  //两指距离
			  scale: 1,                                     //缩放倍数
			  rotate: 0                                     //旋转角度
			},
			bgImageInfo:{},
			bgscle:0,
			startX:0,
			startY:0
		};
	},
	watch:{
		posterPicUrl(newVal) {
			console.log(newVal,'newVal')
		  this.$nextTick(() => {
		    this.initImg(newVal)
		  })
		},
		posterBgImageURL(newVal) {
			console.log(newVal,'newVal')
		  this.$nextTick(() => {
		    this.getPosters(newVal)
		  })
		}
	},
	methods: {
		rotate() {
			let _this = this;
			_this.stv.rotate % 90 == 0 ? _this.stv.rotate = _this.stv.rotate + 90 : _this.stv.rotate = 0
		   
		},
		getImageInfo(image) {
			return new Promise((req, rej) => {
				uni.getImageInfo({
					src: image,
					success: function(res) {
						req(res)
					},
				});
			})
		},
		async getPosters(bgImageURL){
			let bgImageInfo = await this.getImageInfo(bgImageURL)
			this.width = bgImageInfo.width/2
			this.height = bgImageInfo.height/2
		},
		initImg(url) {
		    let _this = this;
		    uni.getImageInfo({
		      src: url,
		      success(resopne) {
		        let innerAspectRadio = resopne.width / resopne.height;
				console.log(resopne)
		        if (innerAspectRadio < _this.width / _this.height) {
					_this.originImg.src = url
					_this.originImg.width = Number(resopne.width)
					_this.originImg.height =  Number(resopne.height)
					_this.stv.offsetX = 0
					_this.stv.offsetY = 0
					_this.stv.zoom = false
					_this.stv.distance = 0
					_this.stv.scale = 1
					_this.stv.rotate = 0
		        } else {
					_this.originImg.src = url
					_this.originImg.width = Number(resopne.width)
					_this.originImg.height = Number(resopne.height)
					_this.stv.offsetX = 0
					_this.stv.offsetY = 0
					_this.stv.zoom = false
					_this.stv.distance = 0
					_this.stv.scale = 1
					_this.stv.rotate = 0
		        }
		      }
			  
		    })
		  },
		  //事件处理函数
		  touchstartCallback: function (e) {
		    if (e.touches.length === 1) {
		      let { clientX, clientY } = e.touches[0];
		      this.startX = clientX;
		      this.startY = clientY;
		      this.touchStartEvent = e.touches;
		    } else {
		      let xMove = e.touches[1].clientX - e.touches[0].clientX;
		      let yMove = e.touches[1].clientY - e.touches[0].clientY;
		      let distance = Math.sqrt(xMove * xMove + yMove * yMove);
		      twoPoint.x1 = e.touches[0].pageX * 2
		      twoPoint.y1 = e.touches[0].pageY * 2
		      twoPoint.x2 = e.touches[1].pageX * 2
		      twoPoint.y2 = e.touches[1].pageY * 2
			  this.stv.distance = distance
			  this.stv.zoom = true
			  _this.stv.rotate = 0
		    }
		  },
		  //图片手势动态缩放
		  touchmoveCallback: function (e) {
		    let _this = this
		    if (e.touches.length === 1) {
		      //单指移动
		      if (_this.stv.zoom) {
		        //缩放状态，不处理单指
		        return;
		      }
		      let { clientX, clientY } = e.touches[0];
		      let offsetX = clientX - _this.startX;
		      let offsetY = clientY - _this.startY;
		      _this.startX = clientX;
		      _this.startY = clientY;
		      let { stv } = _this;
		      stv.offsetX += offsetX;
		      stv.offsetY += offsetY;
				stv.offsetX = Math.floor(stv.offsetX);
				stv.offsetY = Math.floor(stv.offsetY);
		      stv.offsetLeftX = -stv.offsetX;
		      stv.offsetLeftY = -stv.offsetY;
			  stv.picleftX = stv.offsetX
			  stv.picleftY = stv.offsetY
		    		_this.stv = stv
		    	
		    } else if (e.touches.length === 2) {
		      //计算旋转
		      let preTwoPoint = JSON.parse(JSON.stringify(twoPoint))
		      twoPoint.x1 = e.touches[0].pageX * 2
		      twoPoint.y1 = e.touches[0].pageY * 2
		      twoPoint.x2 = e.touches[1].pageX * 2
		    	
		      function vector(x1, y1, x2, y2) {
		        this.x = x2 - x1;
		        this.y = y2 - y1;
		      };
		    	
		      //计算点乘
		      function calculateVM(vector1, vector2) {
		        return (vector1.x * vector2.x + vector1.y * vector2.y) / (Math.sqrt(vector1.x * vector1.x + vector1.y * vector1.y) * Math.sqrt(vector2.x * vector2.x + vector2.y * vector2.y));
		    	
		      }
		      //计算叉乘
		      function calculateVC(vector1, vector2) {
		        return (vector1.x * vector2.y - vector2.x * vector1.y) > 0 ? 1 : -1;
		      }
		    	
		      let vector1 = new vector(preTwoPoint.x1, preTwoPoint.y1, preTwoPoint.x2, preTwoPoint.y2);
		      let vector2 = new vector(twoPoint.x1, twoPoint.y1, twoPoint.x2, twoPoint.y2);
		      let cos = calculateVM(vector1, vector2);
		      let angle = Math.acos(cos) * 180 / Math.PI;
		    	
		      let direction = calculateVC(vector1, vector2);
		      let _allDeg = direction * angle;
		    	
		    	
		      // 双指缩放
		      let xMove = e.touches[1].clientX - e.touches[0].clientX;
		      let yMove = e.touches[1].clientY - e.touches[0].clientY;
		      let distance = Math.ceil(Math.sqrt(xMove * xMove + yMove * yMove));
		      let distanceDiff = distance - _this.stv.distance;
		      let newScale = _this.stv.scale + 0.005 * distanceDiff;
		    	
		      if (Math.abs(_allDeg) > 1) {
		    		_this.stv.rotate =  _this.stv.rotate + _allDeg
		      } else {
		        //双指缩放
		        let xMove = e.touches[1].clientX - e.touches[0].clientX;
		        let yMove = e.touches[1].clientY - e.touches[0].clientY;
		        let distance = Math.sqrt(xMove * xMove + yMove * yMove);
		        let distanceDiff = distance - _this.stv.distance;
		        let newScale = _this.stv.scale + 0.005 * distanceDiff;
		        if (newScale < 0.2 || newScale > 2.5) {
		          return;
		        }
		    	_this.stv.distance = distance
		    	_this.stv.scale = newScale
		    	_this.stv.rotate = 0
		      }
		    } else {
		      return;
		    }
		  },
		  touchendCallback: function (e) {
		    //触摸结束
		    if (e.touches.length === 0) {
				this.stv.zoom = false
		    }
		  }
		}
	}
</script>

<style lang="scss" scoped>
		.poster {
		width: 100%;
		.poster-sc{
			width: 100%;
			height: 100%;
			margin-top: 30px;
			.poster-img {
			   overflow: hidden;
			   background: #fff;
			   border-radius: 32rpx;
			   margin: 0 auto;
			   .poster-bgmask{
			   		height: 830rpx;
			   		width: 540rpx;
			   		overflow: hidden;
			   		// margin:auto;
			   		position:absolute;
			   		overflow: hidden;
			   		top: 30px;
			   		left: 50%;
			   		transform: translateX(-50%);
			   		z-index: 1000;
			   		background-size: 100%;
			   		pointer-events: none;
			   		image{
			   			width: 55px;
			   			position: absolute;
			   			border-radius: 50%;
			   		}
			   	}	
			}
		}
	}
</style>