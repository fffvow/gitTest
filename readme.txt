
/**颜色定义**/
var EVENTSUBTYPE = {
	101:"一般交通事故",
	102:"严重交通事故",
	103:"故障车",
	201:"道路施工",
	301:"交通管制",
	302:"道路关闭",
	303:"出口匝道关闭",
	304:"入口匝道关闭",
	102302:"严重交通事故&道路关闭",
	201302:"道路施工&道路关闭",
	404302:"大雾&道路关闭",
	406302:"大雨&道路关闭",
	409302:"大雪&道路关闭",
	501:"路面积水",
	901:"公告",
	902:"通车",
	903:"完成改建",
	904:"实景路况",
	907:"定制播报"
}

var EVENTBIGTYPE = {
	100:"事故",
	200:"施工",
	300:"管制",
	400:"关闭",
	500:"其他"
}

var EVENTBIGTYPEBGIMG = {
	100:"../../img/event_sg.png",
	200:"../../img/eventmark.png",
	300:"../../img/event_gz.png",
	400:"../../img/event_gb.png",
	500:"../../img/event_qt.png"
}

/**
 * 道路颜色
 */
var ROADCOLORS = [
	"#ff0000",
	"#2828ff",
	"#ff60af",
	"#28ff28",
	"#ff8f59",
	"#b9b973",
	"#808040",
	"#796400",
	"#7afec6",
	"#9f0050"
];

/**
 * 桥梁、隧道颜色
 */
var ROADLINKEDCOLORS = [
	"#8E388E",
	"#CD6839",
	"#ADFF2F",
	"#8B008B",
	"#8B2252"
];

/**
 * 红绿灯交叉口颜色
 */
var RELATIONCOLORS = [
	"#AEEEEE",
	"#00CD00",
	"#5CACEE",
	"#FF34B3",
	"#A3A3A3",
	"#969696"
];

/**
 * 道路等级名称
 */
var ROADNAME = [
	"高速路",
	"城市快速路",
	"国道",
	"主要道路",
	"省道",
	"次要道路",
	"普通道路",
	"县道",
	"乡公路",
	"县乡村内部道路"
	
];


var ROADDIRECTIONNAME = [
	"双向通行",
	"进口",
	"出口",
	"禁止通行"
];

var DIRANGLENAME = [
	"北",
	"东北",
	"东",
	"东南",
	"南",
	"西南",
	"西",
	"西北"
];

var ROADSTATECOLORMap = {
				0: {text:"未知",color:"#34B000"},
				1: {text:"畅通",color:"#61b250"},
				2: {text:"缓行",color:"#f2ae00"},
				3: {text:"拥堵",color:"#DF0100"},
				4: {text:"严重拥堵",color:"#8E0E0B"},
				5: {text:"无交通",color:"#34B000"}
			};
var ROADSTATECOLOR =[
				"#34B000",
				"#61b250",
				"#f2ae00",
				"#DF0100",
				"#8E0E0B",
				"#34B000"
			];  

///////////颜色定义结束//////////////////

//--------自动建议道路-------------------

function doSuggest(obj,id,callback) {
		var keyword = obj.value,
			listId = document.getElementById(id),
			city = "武汉";
		while(listId.hasChildNodes()) {
			listId.removeChild(listId.firstChild);
		}
		if(!keyword) {
			return;
		}
		map.plugin(['IMAP.Suggest'], function() {
			suggest = new IMAP.Suggest();
			suggest.search(keyword, city, function(status, results) {
				if(status == 0) {
					var records = results.results,
						record, li;
					listId.style.display = "block";
					var ul = document.createElement("ul");
					// ul.style.backgroundColor = "rgb(181, 53, 53)";
					for(var i = 0, l = records.length; i < l; ++i) {
						record = records[i];
						li = document.createElement("li");
						li.innerHTML = record.name;
						addSuggestEvt(record.name, li,obj,id,callback);
						ul.appendChild(li);
					}
					listId.appendChild(ul);
				}
			});
		});
	}

	function addSuggestEvt(name,li,obj,id,callback){
		li.onclick=function(){
			var listId=document.getElementById(id);
			while(listId.hasChildNodes()){
				listId.removeChild(listId.firstChild);
			}
			listId.style.display="none";
			obj.value=name;
			callback()
		}
		
		// obj.onclick = function(){
		// 	document.getElementById('J_roadName').value = this.innerHTML
		// 	// alert()
		// }
	}


//是否移除覆盖物对象
var isRomve = {};

//////////////视频功能开始///////////////////////////////////////////


//添加监控
var dataCluster_camera;
var isRomeCamera = false;

function isAddCamera(){
	if (!isRomeCamera) {
		isRomeCamera = true;

		dataCluster_camera = [];
		addCamera();
	}else{
		isRomeCamera = false;
		
		dataCluster_camera.clearMarkers();
		dataCluster_camera = [];
	};
}

//在地图添加摄像头
function addCamera(){
	$.ajax({
		type:"get",
		url:SERVICE_URL,
		dataType:"json",
		data:{
			sid:50005,
			serviceKey:SERVICE_KEY,
			deviceId : ''
		},
		success:function(result){
			var datas = result.data.rows;
			for(var i =0; i<datas.length; i ++){
				var temp = addEventMarker_my(datas[i].xy,'../../img/camera.png',openCamera1,datas[i]);
				dataCluster_camera.push(temp)
			}

			markersClusterCamera(dataCluster_camera)
		}
	});
}

function openCamera(item){
	
	var the_url = "chrome-extension://fnfnbeppfinmnjnjhedifcfllpcfgeea/navigate.html?chromeurl=[escape]";
	the_url += cameraHtmlUrl;
	if (item.type == "2") {
		the_url += "WebDemo.html";
		the_url += "?paraname=";
		the_url += item.note + "," + item.port;
	};
	if (item.type == "3") {
		the_url += "WebDemo.html";
		the_url += "?paraname=";
		the_url += item.note;
	};
	
	//the_url = "http://www.baidu.com";
	window.open(the_url,'','width=500,height=400')
}

function openCamera1(item){
	// if ($('.cameraWrap')) {
		// alert('ss')
	// }else{
		var body = document.getElementsByTagName('body')[0]
		var wrap = document.createElement('div');
		var box = document.createElement('div');
		var inner = document.createElement('div');
		var closeIcon = document.createElement('img');
		wrap.className = 'cameraWrap';
		box.className = 'cameraBox';
		inner.className = 'cameraInner';
		inner.innerHTML = '<video src="../../img/video_2.mp4" autoPlay="true"  width="100%" height="100%" >'
		closeIcon.className = 'cameraDloseIcon';
		closeIcon.src = '../../img/cemaraClose.png'
		closeIcon.onclick = function(){
			body.removeChild(wrap)
		}
		inner.appendChild(closeIcon)
		box.appendChild(inner)
		wrap.appendChild(box)
		body.appendChild(wrap)
	// };

}


//////////////视频功能结束//////////////////////////////////////////


/////////////////////////路况功能/////////////////////////////////
isRomve["isLayerRomve"] = false;
function isAddTrafficLayer(){
	var isLayerRomve = isRomve["isLayerRomve"];
	if(!isLayerRomve){
		addTrafficLayer();
		isRomve["isLayerRomve"] = true;
	}else{
		removeTrafficLayer();
		isRomve["isLayerRomve"] = false;
	}
}

//添加路况
function addTrafficLayer() {
	//if (trafficLayer){return;}
	
	var trafficLayer = new IMAP.TileLayer({
		maxZoom:18,
		minZoom:1,
		tileSize : 256
	});
	trafficLayer.setTileUrlFunc(getTileUrl);
	trafficLayer.setOpacity(0.7);//设置图层透明度，取值范围0-1
	map.addLayer(trafficLayer);
	isRomve["trafficLayer"] = trafficLayer;
}


 //移除自定义图层
function removeTrafficLayer(){
	var trafficLayer = isRomve["trafficLayer"];
	if (!trafficLayer){return;}
	map.removeLayer(trafficLayer);
	trafficLayer = null;
}

function getTileUrl(x,y,z){
    return TRAFFIC_URL.replace("{x}",x).replace("{y}",y).replace("{z}",z);;
}



function eventBoxSel(dom){
	if (dom.value == 'all') {
		var doms = document.getElementsByName("eventSel");
		if (dom.checked) {
			for (var i = 0; i < doms.length; i++) {
				if (!doms[i].checked) {
					doms[i].checked = dom.checked
					var json = "[{\"bigType\":"+doms[i].value+"}]";
					searchEventList(doms[i],json);
				};
			};
		}else{
			for (var i = 0; i < doms.length; i++) {
				if (doms[i].checked) {
					doms[i].checked = dom.checked
					map.getOverlayLayer().clear(isRomve['event'+doms[i].value]);
				};
			};
		};
		
	}else{
		debugger
		if (dom.checked) {
			var json = "[{\"bigType\":"+dom.value+"}]";
			searchEventList(dom,json);
		}else{
			console.log(isRomve)
			map.getOverlayLayer().clear(isRomve['event'+dom.value]);
		};
	};
}


/**
 * 删除覆盖物
 * @param {Object} type
 */
var removeOnMap= function(dom){
	var cacheName = dom.name+dom.value;
	var listline = isRomve[cacheName];
	if(listline ){
		map.getOverlayLayer().clear(listline);
	}
}

//事件查询
function searchEventList(dom,json){
	$.ajax({
		type:"get",
		url:SERVICE_URL,
		dataType:"json",
		data:{
			sid:'30003',
			cityCode:'420100',
			serviceKey:SERVICE_KEY,
    	  	pageNo : 0,
    	  	eventType : json,
    	  	pageSize: 9999999
		},
		success:function(result){
			isRomve['event'+dom.value] = [];
			if (result.data) {
				var datas = result.data.rows;
				for(var i =0; i<datas.length; i ++){
					isRomve['event'+dom.value].push(addEventMarkers(dom,datas[i])); //添加事件
				}
			};
		}
	});
}


var addEventLine = function(dom,lnglat,datas){
	var cacheName = dom.name+dom.value;
	var polylines = isRomve["polylines"];
	if(polylines)
		map.getOverlayLayer().clear(isRomve["polylines"]);
		
	var lineList= [];
	var shape = datas.shape;
	if(shape.indexOf("#")>0){
		var xys = shape.split("#");
		for(var i =0;i<xys.length;i++){
			var lnglatarr = [];
			var xy = xys[i].split(";");
			for(var j = 0;j<xy.length;j++){
				var tmp = xy[j].split(",");
				lnglatarr.push(new IMAP.LngLat(tmp[0],tmp[1]));
			}
			var plo = new IMAP.PolylineOptions();
			plo.strokeColor = "red";
			plo.strokeOpacity = "1";
			plo.strokeWeight = "3";
			plo.strokeStyle = IMAP.Constants. OVERLAY_LINE_SOLID;//实线
			plo.editabled = false;
			var polyline = new IMAP.Polyline(lnglatarr, plo);
			map.getOverlayLayer().addOverlay(polyline, false);
			lineList.push(polyline);
		}
	}else if(shape.indexOf(";")>0){
		var xy = shape.split(";");
		var lnglatarr = [];
		for(var j = 0;j<xy.length;j++){
			var tmp = xy[j].split(",");
			lnglatarr.push(new IMAP.LngLat(tmp[0],tmp[1]));
		}
		var plo = new IMAP.PolylineOptions();
		plo.strokeColor = "red";
		plo.strokeOpacity = "1";
		plo.strokeWeight = "3";
		plo.strokeStyle = IMAP.Constants. OVERLAY_LINE_SOLID;//实线
		plo.editabled = false;
		var polyline = new IMAP.Polyline(lnglatarr, plo);
		map.getOverlayLayer().addOverlay(polyline, false);
		lineList.push(polyline);
	}else{
		var xy = shape.split(";");
		var lnglatarr = [];
		for(var j = 0;j<xy.length;j++){
			var tmp = xy[j].split(",");
			lnglatarr.push(new IMAP.LngLat(tmp[0],tmp[1]));
		}
		var plo = new IMAP.PolylineOptions();
		plo.strokeColor = "red";
		plo.strokeOpacity = "1";
		plo.strokeWeight = "3";
		plo.strokeStyle = IMAP.Constants. OVERLAY_LINE_SOLID;//实线
		plo.editabled = false;
		var polyline = new IMAP.Polyline(lnglatarr, plo);
		map.getOverlayLayer().addOverlay(polyline, false);
		lineList.push(polyline);
	}
	isRomve["polylines"] = lineList;
	//map.setFitView(lineList);
	addEventInfoWindow(lnglat,datas.eventType,datas.eventOverview,datas.startTime+"到"+datas.endTime);
}


var addEventMarkers = function (dom,datas){
	if(datas.shape){
		var opts=new IMAP.MarkerOptions();
		opts.anchor = IMAP.Constants.BOTTOM_CENTER;
		opts.icon=new IMAP.Icon(EVENTBIGTYPEBGIMG[dom.value],{"size":new IMAP.Size(28, 26),"offset":new IMAP.Pixel(1, 0)});
		var lnglat=paseXY(datas.shape);
		marker=new IMAP.Marker(lnglat, opts);
		map.getOverlayLayer().addOverlay(marker, false);
		marker.addEventListener(IMAP.Constants.CLICK, function(event){
			addEventLine(dom,lnglat,datas);
	    }, marker);

	    return marker
	}
}

var addEventInfoWindow = function (xy,type,name,time){
	var content='<div class="infoWindow"><h3>'+
		EVENTSUBTYPE[type]+
		'</h3><img class="infoWindowClose" src="../../img/event_infowindow_colse.png"  onclick="closeInfoWindow()"><div class="infocontent"><p><span>详情：</span>'+
		name+
		'</p><p><span>时间：</span>'+
		time+
		'</p></div><div style="height: 0px;width: 100%; position:absolute; bottom:0px; text-align: center; z-index: 2;"><img src="../../img/event_bottom.png"></div>'+
   		'</div>';
	
	infowindow = new IMAP.InfoWindow(content,{
		size : new IMAP.Size(322,103),
		position:xy,
		autoPan : false,
		offset : new IMAP.Pixel(210, -60),
		anchor:IMAP.Constants.TOP_CENTER,
		type:IMAP.Constants.OVERLAY_INFOWINDOW_CUSTOM
	});
	map.getOverlayLayer().addOverlay(infowindow);
	// infowindow.autoPan(true);
	// map.setFitView(infowindow);
	// map.setCenter(xy,15);
	// map.panBy(0,-100);
}

var paseXY = function (shape){
	var xy = shape;
	var x = "";
	var y = "";
	if(xy.indexOf("#") > 0){
		var tmp = xy.split("#")[0].split(";")[0].split(",");
		x = tmp[0];
		y = tmp[1];
	} else if(xy.indexOf(";") > 0){
		var tmp = xy.split(";")[0].split(",");
		x = tmp[0];
		y = tmp[1];
	}else{
		var tmp = xy.split(",");
		x = tmp[0];
		y = tmp[1];
	}
	return new IMAP.LngLat(x, y);
}

//复选框事件
function selectRoadType(dom){
	if(dom.checked)
		addRoadLevel(dom);
	else{
		removeOnMap(dom);
	}
}


///////////////////////////////////////////////////
///////////查询道路等级开始

function addRoadLevel(dom){
	var roadType = "";
	var linkType = "";
	if(dom.name == "roadlinkd")
		linkType = dom.value;
	else
		roadType = dom.value;

//显示信息窗口
var showDetailsInfo = function(item){
	var content='<div class="infoWindow" >'
		+'<span class="infoWindowClose" onclick="closeInfoWindow()"></span>'
		+'<div style="overflow: hidden;position:relative" class="infoWindowInner"><div><div class="infocontent"><p><span>名称：</span>'+item.roadName+
		'</p><p><span>最大速度：</span>'+item.maxSpeed+'km/h'+
		'</p></div></div></div>';
		
	content = content.replace(/undefined/g, "")
	content = content.replace(/null/g, "")
	
	addInfoWindow_my(item.xys,content,350,50);

}
	
	var addRoadLine = function (datas,dom,lineList){
    		var lnglatarr = [];
    		var temp = datas.list[j].xys.split(";");
        	for(i = 0; i<temp.length; i++){
        		var a = temp[i].split(",");
        		lnglatarr.push(new IMAP.LngLat(a[0],a[1]));
        	}
        	var plo = new IMAP.PolylineOptions();
        	
        	if(dom.name == "roadlinkd")
				plo.strokeColor = ROADLINKEDCOLORS[dom.value];
			else
				plo.strokeColor = ROADCOLORS[dom.value];
			
			plo.strokeOpacity = "1";
			
			plo.strokeWeight = "3";
			
			plo.strokeStyle = IMAP.Constants. OVERLAY_LINE_SOLID;//实线
			
			plo.editabled = false;
			
			var polyline = new IMAP.Polyline(lnglatarr, plo);
			var kk = datas.list[j];
			map.getOverlayLayer().addOverlay(polyline, false);
			polyline.addEventListener(IMAP.Constants.CLICK, function(event){
				showDetailsInfo(kk)
	        }, polyline);
			lineList.push(polyline);
		return polyline
	};
	
	var cacheName = dom.name+dom.value;
	var proccessData = function(result){
		if(result.data == "" || result.data == "null" || result.data == null){
    		alert("没有数据");
    		return;
    	}
    	var datas = result.data.rows[0] || {};
		var lineList= [];

		for(j = 0; j< datas.list.length; j++){

			lineList.push(addRoadLine(datas,dom,lineList));
		}
		var cacheName = dom.name+dom.value;
		isRomve[cacheName] = lineList;
		// map.setFitView(lineList);
	};
	var params = {
		sid:50017,
    	roadType:roadType,
    	linkType:linkType,
    	serviceKey:SERVICE_KEY
	};
	
	$.getJSON(SERVICE_URL,params,function(result){
		proccessData(result);
	});
}

//////////////////////////////////查询道路等级结束/////////////////////

function  closeInfoWindow (){
	if (map && infowindow) {
		map.getOverlayLayer().removeOverlay(infowindow);
	}
}

 //关闭自定义信息窗口
function toggleCloseCustomInfowindow() {
	closeInfoWindow();
}



//////////////地图缩放///////////
function mapEnlarge(){
	var zm = map.getZoom();
	if(zm >= 18)
		return;
		
	map.setCenter(map.getCenter(),zm+1);
}

function mapReduce(){
	var zm = map.getZoom();
	if(zm <= 3)
		return;
	map.setCenter(map.getCenter(),map.getZoom()-1);
}
//////////////地图缩放结束



///////////////////日期工具类///////////

//保留n位小数
function NumberToFix(num,n){
	return num.toFixed(n);
}

//获取当前时间
function getNowDate(){
	var date = new Date();
	var year = date.getFullYear();
	var month = date.getMonth() + 1;
	var day = date.getDate();
	return year + '-' + (month < 10 ? "0" + month : month)
	 + '-' + (day < 10 ? "0" + day : day) ;
}

//获取当前时间 y-m-d
function getDateYMD() {
	var myDate = new Date();
	var m = myDate.getMonth() < 9 ? '0' + (myDate.getMonth() + 1) : myDate.getMonth() + 1;
	var d = myDate.getDate() < 9 ? '0' + (myDate.getDate()) : myDate.getDate();
	return myDate.getFullYear() + '-' + m + '-' + d;
}

//获取某5位小时分钟
function pubTimeFormat(time){
	return time.substr(11,5);
}

//获取当前时间
function getNowMonth(){
	var date = new Date();
	var year = date.getFullYear();
	var month = date.getMonth() + 1;
	return year + '-' + (month < 10 ? "0" + month : month);
}
//获取当前时间
function getNowTime(){
	var date = new Date();
	var hour = date.getHours();
	var min = date.getMinutes();
	var second = date.getSeconds();
	return  hour + ':' + (min < 10 ? "0" + min : min)+':'+(second < 10 ? "0" + second : second);
}

//获取当前时间
function getBeforeMonth(){
	var date = new Date();
	var year = date.getFullYear();
	var month = date.getMonth();
	return year + '-' + (month < 10 ? "0" + month : month);
}

//获取指定日期的前num天
function getBeforeOfDate(num){
	var dd = new Date(); 
	dd.setDate(dd.getDate()+num);//获取AddDayCount天后的日期 
	var y = dd.getFullYear(); 
	var m = dd.getMonth()+1;//获取当前月份的日期 
	var d = dd.getDate(); 
	return y+"-"+m+"-"+d;

}


//获取上一周的日期
function getBeforeWeek(){
	var date = new Date(now.getTime() - 7 * 24 * 3600 * 1000);
	var year = date.getFullYear();
	var month = date.getMonth() + 1;
	var day = date.getDate();
	return year + '-' + (month < 10 ? "0" + month : month)
	 + '-' + (day < 10 ? "0" + day : day);
}

//查找val在数组的位置
function getIndexInArr(arr,val){
	for (var i = 0; i < arr.length; i++) {
		if (arr[i] == val) return i;
	}
	return -1;
}

//在数组中删除val
function removeValFromArr(arr,val){
	var index = getIndexInArr(arr,val);
	if (index > -1) {
		arr.splice(index, 1);
	}
}

//随机生成颜色值
var getRandomColor = function(){
  return '#'+('00000'+(Math.random()*0x1000000<<0).toString(16)).substr(-6); 
}

//--------------------------警力--------------------------

var policeMarkerCluster = []
//开始执行搜索
function addPolice(dom){

	if (!dom.checked) {
		policeMarkerCluster.clearMarkers()
		return
	};


	$.ajax({
		type: "GET",
		url: SERVICE_URL,
		data: {
			sid : 120008,
			serviceKey : SERVICE_KEY,
			pageNo : 0,
			pageSize : 800
		},
		dataType: "json",
		success: function(res) {

			//隐藏loading,请地图
			$('#J_modelBox').hide();

			//循环画点并做点聚合 
			forAddPoliceMarkerCluster(res.data.rows);

		}
	});
}

function forAddPoliceMarkerCluster(items){
	var markerArr = addPoliceMarker(items)
	policeMarkersCluster(markerArr)
}
function addPoliceMarker(data) {

	var tempArr = [];
	for (var x = 0; x < data.length; x++) {
		var temp = addEventMarker_my(data[x].xy,'../../img/jl.png',showPoloceDetailsInfo,data[x]);
		tempArr.push(temp);
	}
	return tempArr
	
}

//显示警员信息窗口
function showPoloceDetailsInfo(item){

	var content='<div class="infoWindow" >'
		+'<span class="infoWindowClose" onclick="closeInfoWindow()"></span>'
		+'<div style="overflow: hidden;position:relative" class="infoWindowInner"><div><div class="infocontent"><p><span>警员姓名：</span>'+item.policeName+
		'</p><p><span>警员编号：</span>'+item.policeNum+
		'</p><p><span>PDA编号：</span>'+item.pdaNum+
		'</p><p><span>警员岗位：</span>'+item.position+
		'</p><p><span>所属中队：</span>'+item.midTeam+
		'</p><p><span>所属大队：</span>'+item.bigTeam+
		'</p><p><span>电话：</span>'+item.phone+
		'</p></div></div></div>';
		

	police_addinfoWindow(content,item.xy)

}

//添加警员信息窗口
function police_addinfoWindow(content,xy){

	content = content.replace(/undefined/g, "")
	content = content.replace(/null/g, "")

	addInfoWindow_my(xy,content,360,60);
	$('.infoWindowInner').perfectScrollbar();
}

//警察点聚合
function policeMarkersCluster(markers){

		//markers：点集合（数组类型,必填）
		// var dataCluster = {}
		map.plugin(['IMAP.DataCluster'], function(){
			policeMarkerCluster = new IMAP.DataCluster(map, markers, {
				maxZoom: 0,
				gridSize: 100,
				zoomOnClick:true,
				minimumClusterSize: 10
			});
		});
	}



//-----------------停车场------------------------------

var parkMarkerListClusterArr = []
var parkDatasRowItem = null
function showParks(dom){
	if (!dom.checked) {
		parkMarkerListClusterArr.clearMarkers()
		return
	};
	var tempMarkerList = []
	$.ajax({
		type: "GET",
		url: SERVICE_URL,
		// url:'http://192.168.11.85:8080/traffic-portal/gate',
		data: {
			sid: 70001,
			serviceKey: SERVICE_KEY
		},
		dataType: "json",
		success: function(res) {
			res = {"sid":70001,"status":{"code":0,"msg":"success"},"data":{"rows":[
{"parkID":"420100_145145432913236","parkName":"武大科技园创业楼停车场","surplusCount":65,"xy":"114.415765,30.454873","xys":"114.415765,30.454873","3":{"parkCount":150},"5":{"parkCount":3350}},
{"parkID":"420100_145145447165437","parkName":"吉庆街停车场","surplusCount":0,"xy":"114.298618,30.583006","xys":"114.298618,30.583006","3":{"parkCount":250},"5":{"parkCount":30050}}]}}




			for(var x = 0;x < res.data.rows.length;x++){
				var item = res.data.rows[x];
				if (item.xy) {
					var temp = addEventMarker_my(item.xy,"../../img/parkIcon.png",parkInfoWindow,item);
					tempMarkerList.push(temp);
				};
			}
			parkMarkerListCluster(tempMarkerList)
		}
	});
}

function togParkInfoWindow(num){
	closeInfoWindow()
	parkInfoWindow(parkDatasRowItem,num)
}

//点击添加信息窗口
function parkInfoWindow(datas,thenum){
		parkDatasRowItem = datas
		var item

		if (thenum) {
			thenum = thenum.toString()
			item = datas[thenum]
		}else{
			debugger
			item = datas["3"]
		};

		var name = item.parkName ? item.parkName : '暂无';
		var address = item.parkAddress ? item.parkAddress : '暂无';
		var tel = item.parkTel ? item.parkTel : '暂无';
		var content = []
		content.push('<div class="infoWindow" >')
		content.push('<span class="infoWindowClose" onclick="closeInfoWindow()"></span>')
		// content.push('<h3>停车场名：'+name+'</h3>')
		content.push('<h3>'+address+' '+tel+' ('+item.parkCount+')</h3>')
		content.push('<div class="infocontent">')
		content.push('<p class="tog"><span onclick="togParkInfoWindow(3)">3</span><span onclick="togParkInfoWindow(5)">32</span></p>')
		content.push('<p><span>地址：</span>'+address+'</p>')
		content.push('<p><span>电话：</span>'+tel+'</p>')
		content.push('<p><span>车位总数：</span>'+item.parkCount+'</p>')
		content.push('<p><span>空余车位：</span>'+item.surplusCount+'</p>')
		content.push('<div class="modifyParkInfo"></div><div style="position:absolute;width:30px;height:50px;top:50%;left:-15px;margin-top:-25px;z-index:2;"><img src="../../img/leftArrow.png"></div></div>')


		
		// setMapCenter_my(item.xy);
		addInfoWindow_my(datas.xy,content.join(''),380,70);
	}


	//停车场点聚合
	function parkMarkerListCluster(markers){
		//创建聚合管理对象 并将各参数设置到其中
		map.plugin(['IMAP.DataCluster'], function(){
			parkMarkerListClusterArr = new IMAP.DataCluster(map, markers, {
				maxZoom: 0,
				gridSize: 100,
				zoomOnClick:true,
				minimumClusterSize: 5
			});
		});
	}

//-------------------物流运输-------------------

var transportMarkerList = [[],[],[]];
var dataClusterList = [{},{},{}];

var transportIconList = ["../../img/transport_0.png","../../img/transport_1.png","../../img/transport_2.png"];
function selectTransport(dom){
	if (!dom.checked) {
		//map.getOverlayLayer().clear(transportMarkerList[dom.value]);
		dataClusterList[dom.value].clearMarkers()
		return
	};
	$.ajax({
		type: "GET",
		url: SERVICE_URL,
		data: {
			sid: 110001,
			serviceKey: SERVICE_KEY,
			type : dom.value
		},
		dataType: "json",
		success: function(res) {
			var item = null;
			for(var x = 0;x < res.data.rows.length;x++){
				item = res.data.rows[x];
				if (item.xy) {
					var temp = addEventMarker_my(item.xy,transportIconList[item.type],transportInfoWindow,item);
					transportMarkerList[item.type].push(temp);
				};
			}
			markersCluster(transportMarkerList[item.type],item.type)
		}
	});
}

//点击添加信息窗口
function transportInfoWindow(item){
		// var name = item.parkName ? item.parkName : '暂无';
		// var address = item.parkAddress ? item.parkAddress : '暂无';
		// var tel = item.parkTel ? item.parkTel : '暂无';
		
		var content = '<div class="infoWindow" >'+
			'<span class="infoWindowClose" onclick="closeInfoWindow()"></span>'+
			'<h3>物流类型：'+item.typeCH+'</h3>'+
			'<div class="infocontent"><p><span>地址：</span>'+item.address+
			'</p>';
			if (item.name) {
				content += '<p><span>公司名称：</span>'+item.name+'</p>'
			};
			content += '<div class="modifyParkInfo"></div><div style="position:absolute;width:30px;height:50px;top:50%;left:-15px;margin-top:-25px;z-index:2;"><img src="../../img/leftArrow.png"></div></div>';;
		// setMapCenter_my(item.xy);
		addInfoWindow_my(item.xy,content,380,70);
	}

//-------------------地图操作-------------------

function addEventMarker_my(xy,iconSrc,callback,obj){

	//xy：点坐标  iconSrc：点图标地址  callback：回调函数  obj：回调函数需要的参数对象

	var opts=new IMAP.MarkerOptions();
	opts.anchor = IMAP.Constants.BOTTOM_CENTER;
	opts.icon=new IMAP.Icon(iconSrc,{"size":new IMAP.Size(45, 45),"offset":new IMAP.Pixel(1, 0)});

	var lnglat=paseXY(xy);
	if(lnglat){
		marker=new IMAP.Marker(lnglat, opts);
		map.getOverlayLayer().addOverlay(marker, false);
	}
	if (callback) {
		marker.addEventListener(IMAP.Constants.CLICK, function(event){
        	callback(obj)
        }, marker);
	};
	if (marker) {
		return marker;
	};

}


//企业运输点聚合
	function markersCluster(markers,num){
		//创建聚合管理对象 并将各参数设置到其中
		map.plugin(['IMAP.DataCluster'], function(){
			dataClusterList[num] = new IMAP.DataCluster(map, markers, {
				maxZoom: 0,
				gridSize: 100,
				zoomOnClick:true,
				minimumClusterSize: 5
			});
		});
	}

	//摄像头点聚合
	function markersClusterCamera(markers){
		//创建聚合管理对象 并将各参数设置到其中
		map.plugin(['IMAP.DataCluster'], function(){
			dataCluster_camera = new IMAP.DataCluster(map, markers, {
				maxZoom: 0,
				gridSize: 100,
				zoomOnClick:true,
				minimumClusterSize: 5
			});
		});
	}



	//添加窗口
	function addInfoWindow_my (xy,content,offsetX,offsetY){

		//xy：窗口坐标  content：窗口内容html字符串
		var x = offsetX ? offsetX : 210;
		var y = offsetY ? offsetY : -60;
		var lnglat=this.paseXY(xy);
		infowindow = new IMAP.InfoWindow(content,{
			size : new IMAP.Size(322,103),
			position:lnglat,
			autoPan : false,
			offset : new IMAP.Pixel(x, y),
			anchor:IMAP.Constants.TOP_CENTER,
			type:IMAP.Constants.OVERLAY_INFOWINDOW_CUSTOM
		});
		map.getOverlayLayer().addOverlay(infowindow);
		infowindow.autoPan(true);
	};

	//设置某点为地图中心
	function setMapCenter_my (xy,move111){
		var tmp = paseXY(xy);
		map.setCenter(tmp,15);
		if (move) {
			map.panBy(0,move);
		};
	};
