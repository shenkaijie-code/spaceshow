<template>
  <div class="home">
    <el-row>
      <el-col :span="15">
        <!--        <div id="map" ref="map" ></div>-->
        <div id="map" class="map__x"></div>
      </el-col>
      <el-col :span="9" class="attribute">
        <el-row>
          <div>
            <button @click="drawPoint">画点</button>
            <button @click="drawLine">画线</button>
            <button @click="drawPolygon">画面</button>
            <button @click="drawCircle">画圆</button>
            <button @click="removeLastPoint">撤回</button>
            <button @click="clearDraw">清空</button>
            <button @click="exitDraw">退出</button>
            <!--            <button @click="tes">测试</button>-->
          </div>
        </el-row>
        <el-row type="flex" justify="center">
          <!--          <el-col :span="2">坐标</el-col>-->
          <el-col :span="24">
            <el-text class="mx-1" type="danger">
              坐标:
              <div id='points'></div>
              wkt:
              <div id='wkt'></div>
            </el-text>
          </el-col>
        </el-row>
        <el-row>
          <el-form :model="ruleForm" status-icon ref="formRef" label-width="5px">
            <el-row :gutter="24">
              <el-col :span="2">
                <el-form-item>
                  <el-button type="primary" @click="addFruitConfig">新增行</el-button>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="24" v-for="(item, index) in ruleForm.fruitConfig" :key="index">
              <el-col :span="3">
                <el-button type="success" @click.prevent="upSid(item)">{{ item.sid }}</el-button>
                <!--                <el-form-item label="坐标系" prop="'sid' + index">-->
                <!--                  <el-input type="text" v-model="item.sid" autocomplete="off" maxlength="2">-->
                <!--                  </el-input>-->
                <!--                </el-form-item>-->
              </el-col>
              <el-col :span=4>
                <el-form-item label="id" prop="'linename' + index">
                  <el-input type="text" v-model="item.linename" autocomplete="off" maxlength="2">
                  </el-input>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="wkt" prop="'coordinate' + index" :error="errMsg">
                  <el-input type="text" v-model="item.coordinate" autocomplete="off" minlength="5" @blur="checkWkt">
                  </el-input>
                  <!--                  <label style="color: red">{{ errMsg }}</label>-->
                </el-form-item>
              </el-col>
              <el-col :span="3">
                <el-button type="danger" @click.prevent="removeFruitConfig(item)">删除</el-button>
              </el-col>
              <el-col :span="2">
                <el-button type="success" @click.prevent="to57(item)">转57</el-button>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="4">
                <el-form-item>
                  <el-button :disabled="subHid" type="primary" @click="submitForm">确 定</el-button>
                </el-form-item>
              </el-col>
              <el-col :span="10">
                <el-button :style="{ display: hid}" type="primary" @click="clearLayer">清空</el-button>
              </el-col>
            </el-row>
          </el-form>
        </el-row>
      </el-col>
    </el-row>
  </div>
</template>
<script setup>
import {onMounted, reactive, ref, toRefs} from 'vue' // vue相关方法
import {Map, View} from 'ol' // 地图实例方法、视图方法
import Tile from 'ol/layer/Tile' // 瓦片渲染方法
import TileLayer from 'ol/layer/Tile'
import 'ol/ol.css'
import VectorSource from "ol/source/Vector"; // 地图样式
import VectorLayer from "ol/layer/Vector";
import {GeoJSON, WKT} from "ol/format";
import {XYZ} from "ol/source";
import {Draw} from "ol/interaction";
import * as control from "ol/control";
import {ScaleLine} from "ol/control";
import * as coordinate from "ol/coordinate";
// 参数声明
const formRef = ref(null);
const state = reactive({
  ruleForm: {
    fruitConfig: [{
      linename: '',
      coordinate: '',
      sid: '84',
      errMsg: ''
    }]
  },
})

let features = [];
let wkts = []
let layer = reactive(null)
const addFruitConfig = () => { // 新增
  state.ruleForm.fruitConfig.push({
    linename: '',
    coordinate: '',
    sid: '84',
    errMsg: ''
  })
  subHid.value = true
  hid.value = '';
}

let hid = ref()
let subHid = ref(true)
const removeFruitConfig = (item) => {
  let message = state.ruleForm.fruitConfig;
  const index = message.indexOf(item)
  if (index !== -1) {
    // wkts = wkts.filter(x => x.linename !== message[index].linename)
    let id = message[index].linename;
    if (layer) {
      if (id) {
        // 有值时删除
        let f = layer.getSource().getFeatureById(id);
        layer.getSource().removeFeature(f);
      } else {
        // 如果没有id,按下标删除
        let newVar = layer.getSource().getFeatures();
        // console.log(JSON.stringify(newVar[index] )+"--------"+index)
        layer.getSource().removeFeature(newVar[index])
      }
    }
    // 删除这个值
    message.splice(index, 1);

    if (message.length === 0) {
      hid.value = 'none';
    }
  }
}

const upSid = (item) => {
  console.log(item)
  item.sid = item.sid === '57' ? '84' : '57'
}
const to57 = (item) => {
  let message = state.ruleForm.fruitConfig;
  const index = message.indexOf(item)
  if (index !== -1) {
    let coordinate = message[index].coordinate;
    let s = new WKT().readFeature(coordinate);

    let s1 = new WKT().writeFeature(s, {
      dataProjection: 'EPSG:3857',//目标坐标系
      featureProjection: 'EPSG:4326'  //当前坐标系
    });
    document.getElementById('points').innerHTML = s.getGeometry().getCoordinates()
    document.getElementById('wkt').innerHTML = s1;

  }
}


const clearLayer = () => { // 删除
  let features1 = layer.getSource().getFeatures();
  for (const feature of features1) {
    // layer.getSource().removeFeature(feature)
    layer.getSource().clear()
  }
  // console.log(features1.length)
  // map.removeLayer(layer)

}

let errMsg = ref('')
const checkWkt = (event) => {
  errMsg.value = ''
  let value = event.target.value;
  console.log(value)
  try {
    new WKT().readFeature(value)
    // 校验通过后可以提交
    subHid.value = false
  } catch (e) {
    errMsg.value = '请输入正确的wkt格式';
  }


}

const submitForm = () => { // 点击确定按钮，输出行内数据
  let fruitConfig = state.ruleForm.fruitConfig;
  // console.log("线名：" + fruitConfig[0].linename);
  // console.log("wkt坐标：" + fruitConfig[0].coordinate);
  let featus = [];
  let i = 0
  for (const x of fruitConfig) {
    // console.log(x.coordinate.toString()+"---"+JSON.stringify(x.coordinate));
    // wkts.push({'linename': x.linename, 'coordinate': x.coordinate});
    let readFeature;
    console.log(x.sid + 'sid---')
    if (x.sid === '57') {
      readFeature = new WKT().readFeature(x.coordinate, {
        dataProjection: 'EPSG:3857',
        featureProjection: 'EPSG:4326'
      });
      // console.log(JSON.stringify(readFeature)+'----')
    } else {
      readFeature = new WKT().readFeature(x.coordinate)
    }


    if (x.linename) {
      readFeature.setId(x.linename);
    } else {
      readFeature.setId('w' + i);
      i++
    }
    featus.push(readFeature)
  }

  map.removeLayer(layer)
  // addWkt();
  layer = new VectorLayer({
    source: new VectorSource({
      features: featus
    })
  });
  map.addLayer(layer)

}


// 数据解构
const {
  ruleForm,
  rules
} = {
  ...toRefs(state)
};


// 一个是属性,另一个是context上下文对象,包括(emit事件函数,slots插槽,attrs值对象)
let map = ref(null) // 存放地图实例


let geofiles = []
// geofiles.push(sichuan)
// geofiles.push(sichuan1)


addWkt();
// addGeo();


// features.push(getFeatureByWKT(wkt))
function addGeo() {
  for (let i = 0; i < geofiles.length; i++) {
    let readFeature = new GeoJSON().readFeature(geofiles[i]);
    readFeature.setId('g' + i)
    features.push(readFeature);
  }
}

function addWkt() {
  for (let i = 0; i < wkts.length; i++) {
    let wkt = wkts[i];
    let readFeature = new WKT().readFeature(wkt.coordinate);
    if (wkt.linename) {
      readFeature.setId(wkt.linename);
    } else {
      readFeature.setId('w' + i);
    }
    features.push(readFeature);
  }
}

let source = new VectorSource()
let layers = [
  //图层
  // new Tile({   // 使用瓦片渲染方法
  //   // 空白底图,生产环境不可使用
  //   source: new OSM(),
  // }),
  new Tile({
    source: new XYZ({url: 'http://t4.tianditu.com/DataServer?T=vec_w&tk=049df85c66a8dedc9a2d3dda08793769&x={x}&y={y}&l={z}'}),
    // source: new OSM(),
  }),
  new TileLayer({
    source: new XYZ({url: 'http://t3.tianditu.com/DataServer?T=cva_w&x={x}&y={y}&l={z}&tk=049df85c66a8dedc9a2d3dda08793769'}),
  }),
  new VectorLayer({
    source: new VectorSource({
      features: features
    })
  }),
  new VectorLayer({
    source: source
  })
];


// 实例化鼠标位置控件
let mousePositionControl = new control.MousePosition({
  coordinateFormat: coordinate.createStringXY(6), //坐标格式
  projection: 'EPSG:4326', //地图投影坐标系
});

// 添加比例尺
let scaleline = new ScaleLine({
  units: "metric"
})

function initMap() {
  // 地图实例
  map = new Map({
    target: 'map',                         // 对应页面里 id 为 map 的元素
    layers: layers,  // 图层
    view: new View({                       // 地图视图
      projection: "EPSG:4326",             // 坐标系，有EPSG:4326和EPSG:3857
      center: [114.064839, 22.548857],     // 深圳坐标
      minZoom: 1,                          // 地图缩放最小级别
      zoom: 6                             // 地图缩放级别（打开页面时默认级别）
    }),
    controls: control.defaults().extend([scaleline, mousePositionControl])
  })
  return map
}

map = initMap()

onMounted(() => {
  initMap()
})

//保存这个控件状态,保证不会改变成别的对象
let draw = ref(null)


function addInteraction(type) {
  // 添加交互绘制控件
  draw = new Draw({
    source: source,
    type: type
  });
  map.addInteraction(draw);
  // 监听点
  draw.on('drawend', (event) => {
    //  结束时的线
    let feature = event.feature;
    let geometry = feature.getGeometry();
    let type = geometry.getType()
    // console.log(type)
    let a
    if (type === 'Circle') {
      a = '半径:' + geometry.getRadius() + '圆心:' + geometry.getCenter()
      // console.log(geometry.getRadius() + '===' + geometry.getCenter());
      document.getElementById('wkt').innerHTML = a;
    } else {
      a = JSON.stringify(geometry.getCoordinates())
      document.getElementById('wkt').innerHTML = new WKT().writeFeature(feature, {
        dataProjection: 'EPSG:4326',//目标坐标系
        featureProjection: 'EPSG:4326'  //当前坐标系
      });
    }
    document.getElementById('points').innerHTML = a
    return a;
  })
}

const drawPoint = () => {
  //退出其他绘制
  exitDraw();
  // 绘制点
  addInteraction("Point");
}

function drawLine() {
  //退出其他绘制
  exitDraw();
  // 绘制线
  addInteraction("LineString");
}

function drawPolygon() {
  //退出其他绘制
  exitDraw();
  // 绘制面
  addInteraction("Polygon");
}

function drawCircle() {
  //退出其他绘制
  exitDraw();
  // 绘制圆
  addInteraction("Circle");
}

function clearDraw() {
  document.getElementById('wkt').innerHTML = '';
  document.getElementById('points').innerHTML = ''
  // 清空绘制
  source.clear();
}

function exitDraw() {
  // 退出绘制模式
  if (draw) {
    // 移除交互
    map.removeInteraction(draw);
    source.clear();
    draw = null;
  }
}

// 撤回操作
function removeLastPoint() {
  draw.removeLastPoint()
}


</script>

<style scoped>
.home {
  width: 1024px;
}

.map__x {
  width: 600px;
  height: 530px;
  border: 1px solid #eee;
}


</style>


