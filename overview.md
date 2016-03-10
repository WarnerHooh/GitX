# 近期作品整理

## Apia Health Insurance
URL: [http://www.apia.com.au/health-insurance](http://www.apia.com.au/health-insurance)

Partial: 整个站是基于Drupal开发的，主要完成的是页面里面的Calculator(图片下面部分，可以组合选择不同的保险种类)，没有什么技术难点。

Desc: 该部分采用的React + Redux完成开发，由于客户的保密要求，我们都是在客户的虚拟机上完成的开发，所以没有源码展示。

## Haier 招投标系统
URL: [http://hope.haier.com/bid/#/](http://hope.haier.com/bid/#/)

Partial: 开发周期4个月，该系统包含海尔内部招标管理以及供应商投标两部分，主要难点是标的状态(招标中、截标、评标、议价、中标审核、法务、流标、废标等)管理以及管理人员权限配置。

Desc: 该项目采用的Angular + Bootstrap开发，使用了gulp/bower/sass/jade等技术点。由于权限与标的状态等多条件组合控制前端展示，使得前端必须灵活可配置的渲染页面。所以主要采用Angular的Directive抽取和组合。

Code: 

``` javascript
'use strict';

// 标的状态组件【是否可废标】
module.exports = function(ngModule) {
	ngModule.directive('bidBoxHead', function(User, AppConfig, bidHelper) {
		return {
			restrict: 'EA',
	        scope: {
	        	'bidInfo': '='
	        },
	        replace: true,
	        template: require('./templates/bid-box-head.jade'),
	        link: function ($scope, $element, $attrs, ngModelCtrl) {
	        	var vm = $scope.vm = {},
	        		bidInfo = $scope.bidInfo,
	        		PROCESS = AppConfig.BID_PROCESS,
	        		PERMISSIONS = AppConfig.PERMISSION;

	        	_.extend(vm, {
	        		bidInfo: bidInfo,

	                sendAction: function(btn) {
	                	bidHelper.sendAction(btn);
	                },

	                canAbolish: function() {
	                		    //检查是标的创建人
		                return bidHelper.isOwner(bidInfo.projectManagerUuid)
	                		    //检查废标权限
		                	   && User.checkPermission(PERMISSIONS['BIDDING_ABOLISH'], bidInfo.regionUuid)
		                		//检查标的进度
		                	   && bidHelper.checkBidProcessInTime(bidInfo.status, PROCESS['CHECK'], PROCESS['PUBLISHED_RESULTS'])
		                	    //检查标的子状态[是否已经废标待审]
		                	   && bidInfo.subStatus !== "WAIT_ABOLISH"
		                	   	//检查是否流标
		                	   && ['BID_CLOSED', 'BID_END'].indexOf(bidInfo.status) === -1;
	                }
	        	});

				//标的状态
				bidHelper.getProjectStatus().then(function (data) {
					vm.projectStatusList = data;
				});
	        }
		}
	});
}
```


``` javascript
"use strict";

//User 处理用户登录、权限等
module.exports = function (ngModule) {
    ngModule.service('User', function (HttpResource, Storage, $http, $location, $timeout, $state, AppConfig, utilService) {

        return {
            //获取权限数据
            getPermissions: function () {
                if (!_destructed) {
                    this.destructuUserData();
                }
                return _permissions;
            },
            //获取业务领域
            getBusinessFields: function () {
                var regions = this.getRegions(),
                    businessFields = [];
                _.values(regions).forEach(function (region) {
                    if (region) {
                        businessFields = businessFields.concat(region.businessFieldList);
                    }
                });
                return _.uniq(businessFields, 'uuid');
            },
            /*
             * 检查是否有权限
             * permission: 权限KEY
             * region: 权限对应部门[为空则在所有的部门查找，只要有一个匹配则为true]
             */
            checkPermission: function (permission, region) {
                var __permissions = this.getPermissions();
                if (!!region) {
                    return __permissions[region] && __permissions[region].indexOf(permission) > -1;
                } else {
                    var arr = _.values(__permissions);
                    for (var i = 0; i < arr.length; i++) {
                        if (arr[i].indexOf(permission) !== -1) {
                            return true;
                        }
                    }

                    return false;
                }
            }
        };

    });
};

```

## 上个公司的项目
URL: 

客服版 [http://im.zhe800.com/index.html#login?refer=http%3A%2F%2Fhelp.zhe800.com%2F&type=service&busUid=1](http://im.zhe800.com/index.html#login?refer=http%3A%2F%2Fhelp.zhe800.com%2F&type=service&busUid=1)

商家版 [http://im.zhe800.com/index.html#/buyer?type=buyer&busUid=1502525%230&refer=http:%2F%2Fshop.zhe800.com%2Fproducts%2Fze151211160434000348%3Fjump_source%3D1%26qd_key%3DqyOwt6Jn](http://im.zhe800.com/index.html#/buyer?type=buyer&busUid=1502525%230&refer=http:%2F%2Fshop.zhe800.com%2Fproducts%2Fze151211160434000348%3Fjump_source%3D1%26qd_key%3DqyOwt6Jn)

Partial: 开始是3个人一起开发，基本聊天功能调通之后由给我单独负责开发。主要开发功能：聊天输入框、客服排队功能、商品预览、历史记录、评价等

Desc: 基于的Angular开发，采用了jsjac与Tigase服务器XMPP协议通讯，主要开发周期在5个月左右。集成了UMEditor并修改了其部分源码以完成定制需求(截屏粘贴功能等)。

Code:

``` javascript
/**
 * tabpane 指令，和tabitem 配合使用
 * tab页签选中的时候会有一个on的样式
 */
imdirectives.directive('tabpane', function() {
	var html = [];
	html.push('<div class="im-tabpane" id="{{id}}">');
	html.push('<ul class="tabtitle">');
	html.push('<li ng-repeat="item in items" style="width:{{w}}" class="{{item.itemclass}} {{item.on}}" id="{{item.itemid}}" ng-click="select(item)">');
	html.push('<em></em><a href="javascript:void(0)" onclick="return false;" target="_self">{{item.tabtitle}}</a>');
	html.push('</li>');
	html.push('</ul>');
	html.push('<div ng-transclude></div>');
	html.push('</div>');

	return {
		restrict: 'E',
		template: html.join(""),
		transclude: true,
		scope: {
			id: '@zId'
		},
		replace: true,
		controller: ['$scope', function($scope) {
			$scope.items = [];
			$scope.select = function(item) {
				angular.forEach($scope.items, function(i) {
					i.selected = false;
					i.on = "";
				});
				item.selected = true;
				item.on = "on";
			};

			this.addpane = function(scope) {
				if ($scope.items.length === 0) {
					$scope.select(scope);
				}
				$scope.items.push(scope);
			};
		}],
		link: function(scope, element) {
			scope.w = (100 / scope.items.length) - 1 + "%";
		}
	};
});

/**
 * tabitem 指令，为tabpane的每一个选项卡
 * 需要依赖tabpane
 * [attribute]z-tabtitle : tabitem的页签名称
 * [attribute]z-class : tabitem的class名称
 */
imdirectives.directive('tabitem', function() {
	var html = [];
	html.push('<div class="im-tabitem" ng-show="selected" ng-transclude></div>');

	return {
		restrict: 'E',
		require: '^tabpane',
		template: html.join(""),
		transclude: true,
		replace: true,
		scope: {
			tabtitle: '@zTabtitle',
			itemclass: '@zItemclass',
			itemid: '@zId'
		},
		link: function(scope, element, attrs, tabpaneCtrl) {
			tabpaneCtrl.addpane(scope);
		}
	};
});
```

``` javascript
/**
* IM桌面通知消息
**/
imservices.service('$notification', ['CONSTANT', function(CONSTANT){
	//消息通知5秒后自动消失
	var AUTO_CLOSE_DELAY_SECONDS = 5,
	//是否弹出消息通知(当IM窗口激活时不弹出通知)
		_notiFlag = false;

	//目前Chrome和FF支持
	if (!("webkitNotifications" in window)) {
		if (("Notification" in window)){
			window.webkitNotifications = window.Notification;
			//浏览器发出允许弹出通知
			window.webkitNotifications.checkPermission = function(){
				switch (window.webkitNotifications.permission) {
					case "granted":
					return 0;
					case "denied":
					return 2;
					case "default":
					return 1;
				}
				return 2;
			}
			//创建通知消息[title: 通知标题；icon：通知图标；body：通知内容；tag：通知标识(相同tag的消息将会覆盖)]
			window.webkitNotifications.createNotification = function(var1,var2,var3,var4){
				return new Notification(var2,{icon:var1, body:var3, tag: var4});
			}
		}
	}

	function notify(e){
	    if (window.webkitNotifications) {
	    	//如果已获得浏览器弹出通知权限，则创建新消息通知
	        if (window.webkitNotifications.checkPermission() == 0) {
	        	if(!_notiFlag) return;
	            var popup = window.webkitNotifications.createNotification(e.icon, e.title, (UM.browser.gecko && e.body.length > 20 ? e.body.substring(0, 20)+'...' : e.body), 'IM_NEW_MSG');
	            popup.onclick = function(){
	            	//使IM窗口居中，在FF下可激活IM页面
	              	// window.moveTo((window.screen.availWidth-document.body.clientWidth)/2, (window.screen.availHeight-document.body.clientHeight)/2);
	              	window.focus();
	            }
	            popup.onshow = function(event) {
	                setTimeout(function() {
	                    event.target.close();
	                }, AUTO_CLOSE_DELAY_SECONDS * 1000);
	            }
	            if (("show" in popup)){
	              	popup.show();
	            }
	        //如果未获得浏览器弹出通知权限，则弹出浏览器允许提示
	        } else {
	            window.webkitNotifications.requestPermission();
	            return;
	        }
	    }
	}

	function permission() {
		if(window.webkitNotifications) {
			//监听IM窗口是否激活，判断是否弹出消息通知
			$(window).bind( 'blur', function() {
				_notiFlag = true;
			}).bind( 'focus', function(){
				_notiFlag = false;
			}); 

			//向浏览器发出允许弹出消息通知请求权限
			if(window.webkitNotifications.checkPermission() !== 0) {
			 	window.webkitNotifications.requestPermission();
			}
		}
	}

	return {
		permission: permission,
		notify: notify
	}
}]);
```