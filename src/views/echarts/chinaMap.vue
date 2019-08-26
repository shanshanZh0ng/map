<template>
  <div class="">
    <div class="banner">
      <div class="banner-top">
        <p class="banner-part">设备接入总数：<span>797,910</span></p>
        <p class="banner-part">设备上云企业数：<span>769</span></p>
      </div> 
      <div class="border-box">
        <div class="border-part">
          <p class="part-icon running"></p>
          <p class="part-percent">68.31<span>%</span></p>
          <p class="part-name">运行率</p>
        </div>
        <div class="border-part">
          <p class="part-icon break"></p>
          <p class="part-percent">0.12<span>%</span></p>
          <p class="part-name">故障率</p>
        </div>
        <div class="border-part">
          <p class="part-icon wait"></p>
          <p class="part-percent">15.59<span>%</span></p>
          <p class="part-name">待机率</p>
        </div>
      </div>
    </div>
    <!-- <div class="title">Echarts中国地图三级钻取</div> -->
    <div class="box">
      <button class="backBtn">
        <span @click="backChina()">返回中国 </span>
        <!-- <span  @click="back()">返回上级</span> -->
      </button>
      <div id="mapChart" class="chart"></div>
    </div>
  </div>
</template>

<script>
import cityMap from "@/js/china-main-city-map.js";
import echarts from "echarts";
import axios from "axios";
import Vue from "vue";
// import registerAndsetOption from '@/js/echarts-map'

var mapName = "china";
var foshan = [
  {name:"三水区",value:[112.89, 23.18, 47],tipData:[{name:"佛山华数中试车间", value: 8}]},
  {name:"顺德区",value:[113.24, 22.84, 89],tipData:[{name:"佛山华数机器人", value: 62}]}
];

var data = [
  {name:"北京",value:40,tipData:[{name:"企业数", value: 99},{name:"设备数",value: 32}]},
  {name:"海淀区",value: 76,tipData:[{name:"企业数", value: 56},{name:"设备数",value: 34}]},
  {name:"河北",value: 67,tipData:[{name:"企业数", value: 67},{name:"设备数",value: 32}]},
  {name:"石家庄市",value:76,tipData:[{name:"企业数", value: 56},{name:"设备数",value: 34}]},
  {name:"山东",value: 52,tipData:[{name:"企业数", value: 82},{name:"设备数",value: 32}]},
  {name:"济南市",value: 52,tipData:[{name:"企业数", value: 89},{name:"设备数",value: 32}]},
  {name:"青海",value: 83,tipData:[{name:"企业数", value: 82},{name:"设备数",value: 32}]},
  {name:"西藏",value: 83,tipData:[{name:"企业数", value: 82},{name:"设备数",value: 32}]},
  {name:"海南藏族自治州",value: 83,tipData:[{name:"企业数", value: 83},{name:"设备数",value: 32}]},
  {name:"共和县",value: 83,tipData:[{name:"佛山华硕机器人", value: 89}]},
  {name:"湖南",value: 47,tipData:[{name:"企业数", value: 47},{name:"设备数",value: 32}]},
  {name:"邵阳市",value:47,tipData:[{name:"企业数", value: 89},{name:"设备数",value: 32}]},
  {name:"江西",value: 39,tipData:[{name:"企业数", value: 89},{name:"设备数",value: 32}]},
  {name:"广西",value: 47,tipData:[{name:"企业数", value: 89},{name:"设备数",value: 32}]},
  {name:"广东",value: 89,tipData:[{name:"企业数", value: 89},{name:"设备数",value: 32}]},
  {name:"佛山市", value: 67,tipData:[{name:"企业数", value: 89},{name:"设备数",value: 32}]},
  {name:"三水区",value:47,tipData:[{name:"佛山机器人", value: 89}]},
  {name:"顺德区",value: 89,tipData:[{name:"佛山华硕机器人", value: 89}]}
]
var data = [
  {name:"上海",value:30,tipData:[{name:"企业总数", value: 3},{name:"设备总数",value: 28495}]},
  {name:"云南",value:30,tipData:[{name:"企业总数", value: 3},{name:"设备总数",value: 168}]},
  {name:"内蒙古",value:60,tipData:[{name:"企业总数", value: 9},{name:"设备总数",value: 1110}]},
  {name:"北京",value:85,tipData:[{name:"企业总数", value: 35},{name:"设备总数",value: 152211}]},
  {name:"吉林",value:20,tipData:[{name:"企业总数", value: 2},{name:"设备总数",value: 1027}]},
  {name:"四川",value:100,tipData:[{name:"企业总数", value: 84},{name:"设备总数",value: 5672}]},
  {name:"天津",value:60,tipData:[{name:"企业总数", value: 9},{name:"设备总数",value: 33150}]},
  {name:"山东",value:30,tipData:[{name:"企业总数", value: 3},{name:"设备总数",value: 99817}]},
  {name:"广东",value:77,tipData:[{name:"企业总数", value: 37},{name:"设备总数",value: 132375}]},
  {name:"广西",value:20,tipData:[{name:"企业总数", value: 2},{name:"设备总数",value: 2444}]},
  {name:"江苏",value:120,tipData:[{name:"企业总数", value: 484},{name:"设备总数",value: 143973}]},
  {name:"江西",value:60,tipData:[{name:"企业总数", value: 6},{name:"设备总数",value: 9055}]},
  {name:"河北",value:12,tipData:[{name:"企业总数", value: 12},{name:"设备总数",value: 18115}]},
  {name:"河南",value:20,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 38975}]},
  {name:"浙江",value:70,tipData:[{name:"企业总数", value: 7},{name:"设备总数",value: 24441}]},
  {name:"湖北",value:60,tipData:[{name:"企业总数", value: 9},{name:"设备总数",value: 5152}]},
  {name:"湖南",value:30,tipData:[{name:"企业总数", value: 3},{name:"设备总数",value: 1945}]},
  {name:"甘肃",value:30,tipData:[{name:"企业总数", value: 3},{name:"设备总数",value: 3356}]},
  {name:"福建",value:30,tipData:[{name:"企业总数", value: 3},{name:"设备总数",value: 9659}]},
  {name:"西藏",value:20,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 2}]},
  {name:"贵州",value:50,tipData:[{name:"企业总数", value: 16},{name:"设备总数",value: 14601}]},
  {name:"辽宁",value:40,tipData:[{name:"企业总数", value: 15},{name:"设备总数",value: 13932}]},
  {name:"重庆",value:40,tipData:[{name:"企业总数", value: 9},{name:"设备总数",value: 13713}]},
  {name:"陕西",value:45,tipData:[{name:"企业总数", value: 5},{name:"设备总数",value: 800}]},
  {name:"黑龙江",value:30,tipData:[{name:"企业总数", value: 2},{name:"设备总数",value: 737}]},
  // 广东
  {name:"广州市",value:30,tipData:[{name:"企业总数", value: 8},{name:"设备总数",value: 24565}]},
  {name:"韶关市",value:35,tipData:[{name:"企业总数", value: 2},{name:"设备总数",value: 1234}]},
  {name:"深圳市",value:35,tipData:[{name:"企业总数", value: 2},{name:"设备总数",value: 10564}]},
  {name:"珠海市",value:35,tipData:[{name:"企业总数", value: 2},{name:"设备总数",value: 66259}]},
  {name:"佛山市",value:75,tipData:[{name:"企业总数", value: 2},{name:"设备总数",value: 70}]},
  {name:"江门市",value:50,tipData:[{name:"企业总数", value: 2},{name:"设备总数",value: 1183}]},
  {name:"湛江市",value:60,tipData:[{name:"企业总数", value: 4},{name:"设备总数",value: 1194}]},
  {name:"茂名市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 2400}]},
  {name:"肇庆市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 1300}]},
  {name:"惠州市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 1534}]},
  {name:"梅州市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 1328}]},
  {name:"汕尾市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 1347}]},
  {name:"河源市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 2594}]},
  {name:"阳江市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 651}]},
  {name:"清远市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 3427}]},
  {name:"东莞市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 1739}]},
  {name:"中山市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 2908}]},
  {name:"潮州市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 1524}]},
  {name:"揭阳市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 1279}]},
  {name:"云浮市",value:45,tipData:[{name:"企业总数", value: 1},{name:"设备总数",value: 1422}]},
  // 佛山市
  {name:"三水区",value:47,tipData:[{name:"佛山华数中试车间", value: 8}]},
  {name:"顺德区",value:89,tipData:[{name:"佛山华数机器人", value: 62}]}
]
var geoCoordMap = {};

var convertData = function (data) {
  var res = [];
  for (var i = 0; i < data.length; i++) {
    var geoCoord = geoCoordMap[data[i].name];
    if (geoCoord) {
      res.push({
        name: data[i].name,
        value: geoCoord.concat(data[i].value),
        // value: data[i].value,
        tipData: data[i].tipData
      });
    }
  }
  // console.log(res)
  return res;
};

//中国地图（第一级地图）的ID、Name、Json数据
var chinaId = 100000;
var chinaName = "china";
var chinaJson = null;

//记录父级ID、Name
var mapStack = [];
var parentId = null;
var parentName = null;

//Echarts地图全局变量，主要是在返回上级地图的方法中会用到
var myChart = null;
export default {
  name: "chinaMap",
  props: {
    msg: String
  },
  mounted: function() {
    this.mapChart("mapChart");
  },
  methods: {
    /**
     * 返回上一级地图
     */
    back() {
      if (mapStack.length != 0) {
        //如果有上级目录则执行
        var map = mapStack.pop();
        console.log(mapStack,map)
        axios
          .get("./static/json/map/" + map.mapId + ".json", {})
          .then(response => {
            const mapJson = response.data;
            registerAndsetOption(
              myChart,
              map.mapId,
              map.mapName,
              mapJson,
              false
            );

            //返回上一级后，父级的ID、Name随之改变
            // parentId = map.mapId;
            // parentName = map.mapName;
            // registerAndsetOption(myChart, chinaId, chinaName, chinaJson, false);
            // mapStack = [];
            // parentId = chinaId;
            // parentName = chinaName;
          });
      }
    },
    /**
     * 返回中国
     */
    backChina () {
      registerAndsetOption(myChart, chinaId, chinaName, chinaJson, false);
      mapStack = [];
      parentId = chinaId;
      parentName = chinaName;
    },
    /**
     * Echarts地图
     */
    mapChart(divid) {
      axios.get("./static/json/map/" + chinaId + ".json", {}).then(response => {
        const mapJson = response.data;
        chinaJson = mapJson;
        myChart = echarts.init(document.getElementById(divid));
        registerAndsetOption(myChart, chinaId, chinaName, mapJson, false);
      });
    }
  }
};
/**
 *
 * @param {*} myChart
 * @param {*} id        省市县Id
 * @param {*} name      省市县名称
 * @param {*} mapJson   地图Json数据
 * @param {*} flag      是否往mapStack里添加parentId，parentName
 */
function registerAndsetOption(myChart, id, name, mapJson, flag) {
  geoCoordMap = {}
  mapJson.features.forEach((v, i) => {
    // 地区名称
    var name = v.properties.name;
    // 地区经纬度
    geoCoordMap[name] = v.properties.cp;
  });

  echarts.registerMap(name, mapJson);
  myChart.setOption({
    title: {
      text: name === 'china' ? '中国' : name,
      left: 'center',
      textStyle: {
        color: '#666',
        top: 'auto',
        fontSize: 12
      }
    },
    tooltip: {
      triggerOn: 'click',
      padding: 0,
      enterable: true,
      transitionDuration: 1,
      textStyle: {
        color: '#fff',
        decoration: 'none',
      },
      formatter: function(params) {
        var _html = '';
        // _tipHtml 提示框数据
        var _tipHtml = '';
        if (params.componentSubType !== "map"){
          if (params.data.name === "顺德区" || params.data.name === "三水区") {
            _tipHtml = ''
          } else {
            _tipHtml = '<p style="margin:0;">'+params.data.name+'</p>'
          }
          params.data.tipData.forEach((val, k) => {
            _tipHtml += '<p style="margin:0;">'+val.name+":"+val.value+'</p>'
          })
          _html = '<div id="next" style="background:rgba(22,80,158,0.8);border:1px solid rgba(7,166,255,0.7);border-radious:4px;padding:4px 10px;font-size:12px;">'+_tipHtml+'</div>'
            clearTimeout(timer);
            var timer = setTimeout(function() {
              document.getElementById("next").addEventListener("click", function() {
                // console.log(params)
                var cityId = cityMap[params.name];
                if (cityId) {
                  //代表有下级地图
                  axios
                    .get("./static/json/map/" + cityId + ".json", {})
                    .then(response => {
                      const mapJson = response.data;
                      registerAndsetOption(
                        myChart,
                        cityId,
                        params.name,
                        mapJson,
                        true
                      );
                    });
                } else {
                  console.log(params)
                  var type = "map"
                  var mapId = 123
                  getIosAnd(type,mapId);
                  // return 
                }
              })
            })
          return _html;
        }
      }
    },
    visualMap: {
      type: 'piecewise',
      pieces: [
        {
          max: 0,
          label: '无数据',
          color: '#2c9a42'
        },
        {
          min: 1,
          label: '有数据',
          color: '#4987f6'
        }
      ],
      show: false,
      calculable: true,
      seriesIndex:'0'
    },
    geo: {
      show: true,
      map: name,
      visualMap: false,
      roam: false,
      zoom: 1.1
    },
    series: [
      {
        type: "map",
        map: name,
        roam: false,
        zoom: 1.1,
        itemStyle: {
          normal: {
            areaColor: "#abd4fa",//#467bc0
            borderColor: "#fff",
            borderWidth: 1
          },
          emphasis: {
            label: {
              color: "#959fa7",
            },
						// areaColor: '#043c94',
						// borderWidth: 1,
            // borderColor: 'yellow',
            areaColor: "#4987f6",
            borderColor: "#1dc199",
            borderWidth: 1
					}
        },
        data: data
      },
      {
        type: name === "china" ? 'effectScatter' : 'scatter',
        coordinateSystem: 'geo',
        // data: name !== "佛山市" ? (convertData(data.sort(function(a, b) {
        //   return b.value - a.value;
        // }).slice(0, 10))) : foshan,
        data: name !== "佛山市" ? convertData(data) : foshan,
        symbolSize: function(val) {
          return val[2] / 10;
        },
        showEffectOn: 'render',
        rippleEffect: {
          brushType: 'stroke'
        },
        // hoverAnimation: true,
        label: {
          normal: {
            formatter: '{b}',
            position: 'left',
            show: false
          }
        },
        itemStyle: {
          normal: {
            color: 'yellow',
            shadowBlur: 10,
            // shadowColor: 'yellow'
          }
        },
        zlevel: 1
      }
    ]
  });

  if (flag) {
    //往mapStack里添加parentId，parentName,返回上一级使用
    mapStack.push({
      mapId: parentId,
      mapName: parentName
    });
    parentId = id;
    parentName = name;
  }
}

function initMapData(mapJson) {
  var mapData = [];
  for (var i = 0; i < mapJson.features.length; i++) {
    mapData.push({
      name: mapJson.features[i].properties.name
      // id:mapJson.features[i].id
    });
  }
  return mapData;
}

function getIosAnd(type,mapId){
  var u = navigator.userAgent;
  var isAndroid = u.indexOf('Android') > -1 || u.indexOf('Adr') > -1; //android终端
  var isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端
  if(isAndroid){
    window.htyw.actionFromJsWithParam(type, mapId);
  }else if(isiOS){
    window.webkit.messageHandlers.AppModel.postMessage(type, mapId);
  }
};
</script>

<style scoped>
p,div {
  margin: 0;
  padding: 0;
}
.banner {
  position: absolute;
  width: 100%;
  height: 20vh;
  top: 0;
  left: 0;
  background: url(../../assets/banner.png) no-repeat;
  background-size: 100% 100%;
}
.banner-top {
  display: flex;
}
.banner-part {
  font-weight: bold;
  margin-top: 8%;
  height: 20px;
  flex: 1;
  color: #fff;
  text-align: center;
  font-size: 0.9em;
}
.border-box {
  display: flex;
  height: 95%;
  width: 100%;
  position: absolute;
  bottom: -50%;
  left: 0;
  background: url(../../assets/border.png) no-repeat;
  background-size: 100% 100%;
}
.border-part {
  flex: 1;
  text-align: center;
  color: #467bc0;
}
.part-icon {
  margin-top: 8%;
  height: 30%;
}
.running {
  background: url(../../assets/running.png) no-repeat center;
  background-size: 20% auto;
}
.break {
  background: url(../../assets/break.png) no-repeat center;
  background-size: 20% auto;
}
.wait {
  background: url(../../assets/wait.png) no-repeat center;
  background-size: 20% auto;
}
.part-percent {
  height: 30%;
  font-size: 1.2em;
  font-weight: bold;
  box-sizing: border-box;
  padding-top: 5%;
}
.part-percent span {
  font-size: 0.5em;
}
.part-name {
  height: 30%;
  font-size: 0.75em;
}

.title {
  width: 100%;
  height: 10vh;
  text-align: center;
  color: #fff;
  font-size: 2em;
  line-height: 10vh;
}
.box {
  position: absolute;
  width: 100%;
  height: 40vh;
  top: 32%;
  left: 0;
  /* background-color: rgba(24, 27, 52, 0.62); */
}
.chart {
  position: relative;
  height: 100%;
  top: 10%;
}
.backBtn {
  position: absolute;
  top: 0;
  background-color: #467bc0;
  border: 0;
  color: #fff;
  height: 27px;
  font-family: Microsoft Yahei;
  font-size: 0.8em;
  cursor: pointer;
}
.myBlog {
  position: absolute;
  top: 2px;
  right: 5%;
  display: block;
  border: 2px solid #262a53;
}
.myBlog a {
  text-decoration: none;
  display: inline-block;
  color: #fff;
  padding: 5px;
  font-size: 1em;
}

.myBlog a img {
  vertical-align: middle;
  height: 20px;
  width: 20px;
  border-radius: 50%;
  margin: -5px 5px auto auto;
}
.bottom {
  position: absolute;
  width: 100%;
  height: 5%;
  line-height: 100%;
  left: 0;
  bottom: 0%;
  text-align: center;
}
</style>
