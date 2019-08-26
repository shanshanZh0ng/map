<template>
  <div class="">
    <div class="banner">
      <div class="banner-top">
        <p class="banner-part">设备接入总数：<span v-text="this.equipments">788801</span></p>
        <p class="banner-part">设备上云企业数：<span v-text="this.companies">769</span></p>
      </div> 
      <div class="border-box">
        <div class="border-part">
          <p class="part-icon running"></p>
          <p class="part-percent"><span class="rate" v-text="this.runningRate"></span><span class="mini">%</span></p>
          <p class="part-name">运行率</p>
        </div>
        <div class="border-part">
          <p class="part-icon break"></p>
          <p class="part-percent"><span class="rate" v-text="this.errRate"></span><span class="mini">%</span></p>
          <p class="part-name">故障率</p>
        </div>
        <div class="border-part">
          <p class="part-icon wait"></p>
          <p class="part-percent"><span class="rate" v-text="this.waitRate"></span><span class="mini">%</span></p>
          <p class="part-name">待机率</p>
        </div>
      </div>
    </div>
    <!-- <div class="title">Echarts中国地图三级钻取</div> -->
    <div class="box">
      <button class="backBtn">
        <span @click="backChina()">返回中国 </span>
      </button>
      <div id="mapChart" class="chart"></div>
    </div>
  </div>
</template>

<script>
import cityMap from "@/js/china-main-city-map.js";
import total_data from "@/js/mapData.js";
import echarts from "echarts";
import axios from "axios";
import Vue from "vue";

// axios.defaults.baseURL = process.env.API_ROOT

var mapName = "china";
var isShow = false

var data = total_data;

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
  return res;
};
//中国地图（第一级地图）的ID、Name、Json数据
var chinaId = 100000;
var chinaName = "china";
var chinaJson = null;
var _color = "#abd4fa"

//记录父级ID、Name
var mapStack = [];
var parentId = null;
var parentName = null;

//Echarts地图全局变量，主要是在返回上级地图的方法中会用到
var myChart = null;
export default {
  name: "chinaMap",
  data() {
    return {
      equipments: 0,
      companies: 0,
      runningRate: 0,
      errRate: 0,
      waitRate: 0
    }
  },
  props: {
    msg: String
  },
  mounted: function() {
    this.mapChart("mapChart");
    this.getEquipments();
    this.getCompanies();
    this.getRate();
  },
  methods: {
    //设备接入总数接口
    getEquipments () {
      axios
        .get("./api/1.0/iot/getCount", {}).then(Response => {
          let response = JSON.parse(Response.data);
          this.equipments = response.total.toLocaleString();
        })
    },
    //设备上云企业数
    getCompanies () {
      axios
        .get("./api/1.0/iot/getJoinOrgs", {}).then(Response => {
          var response = JSON.parse(Response.data);
          this.companies = response.data.count.toLocaleString();
        })
    },
    //运行率、故障率、待机率
    getRate () {
      axios
        .post("./api/1.0/iot/getLv", {}).then(Response => {
          // let response = JSON.parse(Response.data);
          let res = Response.data.results;
          this.runningRate = res.yxl;
          this.errRate = res.gzl;
          this.waitRate = res.djl;
          this.runningRate = this.runningRate.replace(/[%]/g,'');
          this.errRate = this.errRate.replace(/[%]/g,'');
          this.waitRate = this.waitRate.replace(/[%]/g,'');
        })
    },
    /**
     * 返回上一级地图
     */
    back() {
      isShow = false;
      if (mapStack.length != 0) {
        //如果有上级目录则执行
        var map = mapStack.pop();
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
          });
      }
    },
    /**
     * 返回中国
     */
    backChina () {
      var list = document.getElementById("mapChart")
      list.childNodes[1].style.display = 'none';
      registerAndsetOption(myChart, chinaId, chinaName, chinaJson, false);
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
        myChart.on("click", function(params) {
          showTip (true)
          if (Boolean(params.value)){
            showTip (true)
          } else {
            showTip (false)
          }        
        })
        
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
function registerAndsetOption(myChart, id, name, mapJson, flag, isShow) {
  showTip (false);
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
      // show: true,
      triggerOn: 'click',
      // trigger: 'item',
      padding: 0,
      enterable: true,
      transitionDuration: 1,
      textStyle: {
        color: '#666',
        decoration: 'none',
      },
      formatter: function(params) { 
        if (Boolean(params.value)){
          var _html = '';
          // _tipHtml 提示框数据
          var _tipHtml = '';
          // if (params.componentSubType !== "map"){
          if (params.data.name === "顺德区" || params.data.name === "三水区") {
            _tipHtml = ''
          } else {
            _tipHtml = '<p style="margin:0;color:#fdd01e">'+params.data.name+'</p>'
          }
          if (params.data.tipData === "undefined"){
            return
          }
          params.data.tipData.forEach((val, k) => {
            _tipHtml += '<p style="margin:0;">'+val.name+":"+val.value.toLocaleString()+'</p>'
          })
          _html = '<div id="next" style="background:#fff;border-radious:4px;padding:4px 10px;font-size:12px;">'+_tipHtml+'</div>'
            setTimeout(function() {
              document.getElementById("next").addEventListener("click", function() {
                var list = document.getElementById("mapChart")
                list.childNodes[1].style.display = 'none';
                var cityId = cityMap[params.name];
                
                // if (cityId) {
                  if (name === "china"){   
                    
                    if (params.name === "北京" || params.name === "上海" || params.name === "天津" || params.name === "重庆") {
                      let obj = {"type" : "DeviceMap", "cityName": params.name};
                      getIosAnd(JSON.stringify(obj));
                    } else {
                      //代表有下级地图
                      axios
                        .get("./static/json/map/" + cityId + ".json", {})
                        .then(response => {      
                          showTip(false);                                     
                          const mapJson = response.data;
                          registerAndsetOption(
                            myChart,
                            cityId,
                            params.name,
                            mapJson,
                            true
                          );
                        });
                    }

                  } else {
                    let obj = {"type" : "DeviceMap","province": name,"cityName": params.name};
                    getIosAnd(JSON.stringify(obj));
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
    // geo: {
    //   show: true,
    //   map: name,
    //   visualMap: false,
    //   roam: false,
    //   zoom: 1.1
    // },
    series: [
      {
        type: "map",
        map: name,
        roam: false,
        zoom: 1.1,
        itemStyle: {
          normal: {
            areaColor: "#abd4fa",
            borderColor: "#fff",
            borderWidth: 1
          },
          emphasis: {
            label: {
              color: "#959fa7",
            },
            areaColor: "#fdd01e", //"#4987f6",
            borderColor: "#1dc199",
            borderWidth: 1
					}
        },
        data: data
      },
      // {
      //   type: name === "china" ? 'effectScatter' : 'scatter',
      //   coordinateSystem: 'geo',
      //   // data: name !== "佛山市" ? (convertData(data.sort(function(a, b) {
      //   //   return b.value - a.value;
      //   // }).slice(0, 10))) : foshan,
      //   data: name !== "佛山市" ? convertData(data) : foshan,
      //   symbolSize: function(val) {
      //     return val[2] / 10;
      //   },
      //   showEffectOn: 'render',
      //   rippleEffect: {
      //     brushType: 'stroke'
      //   },
      //   // hoverAnimation: true,
      //   label: {
      //     normal: {
      //       formatter: '{b}',
      //       position: 'left',
      //       show: false
      //     }
      //   },
      //   itemStyle: {
      //     normal: {
      //       color: '#fdd01e',
      //       shadowBlur: 10,
      //       // shadowColor: 'yellow'
      //     }
      //   },
      //   zlevel: 1
      // }
    ]
  });
  // if (flag) {
  //   // myChart.setOption({tooltip: {show: false}})
  //   //往mapStack里添加parentId，parentName,返回上一级使用
  //   mapStack.push({
  //     mapId: parentId,
  //     mapName: parentName
  //   });
  //   parentId = id;
  //   parentName = name;
  // }
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

function getIosAnd(opts){
  var u = navigator.userAgent;
  var isAndroid = u.indexOf('Android') > -1 || u.indexOf('Adr') > -1; //android终端
  var isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端
  if(isAndroid){
    window.htyw.mapMsgFormJs(opts);
  }else if(isiOS){
    window.webkit.messageHandlers.AppModel.postMessage(opts);
  }
};

function showTip (isShow) {
  myChart.setOption({tooltip: {show: isShow}});
}

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
  box-sizing: border-box;
  padding-top: 5%;
}
.part-percent .rate {
  font-size: 1.2em;
  font-weight: bold;
}
.part-percent span.mini {
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
  height: 60vh;
  top: 32%;
  left: 0;
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
