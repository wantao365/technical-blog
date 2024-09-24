> 在使用之前，你需要下载arcgis的离线安装包，然后最好下载一个nginx，用来管理arcgis api和地图碎片管理。同时你应该下载离线地图下载器，将地图碎片下载到本地，放入/arcgis/map文件夹，注意路径1,2,3.....
>
> ```
> npm i esri-loader --save-dev
> ```

# nginx 配置

```shell

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       9400;
        server_name  localhost;
        location / {
            root  D:/Temporary/arcgis;
            index api/4.9/dojo/dojo.js index.html index.htm;
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
            add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';

            if ($request_method = 'OPTIONS') {
                return 204;
            }
        }
    }
    
    server {
        listen       9097;
        server_name  localhost;
        charset utf-8;
        root  /home/data/nginx/html/webpages;
        index index.html index.htm;
        location /platform-boot/ {
            proxy_pass http://127.0.0.1:9402;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
        location /arcgis/ {
            rewrite ^.+fe/?(.*)$ /$1 break;
            proxy_pass http://127.0.0.1:9400/;
            index  index.html index.htm;
        }
    }
    
    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

# 前端demo

基于element-ui......

```html
<template>
  <el-tabs type="border-card" v-model="activeName" @tab-click="handleClick">
    <el-tab-pane label="index" name="first">index</el-tab-pane>
    <el-tab-pane label="map" name="second">
      <div class="map-container">
        <div class="container">
          <div id="arcgis-map-id" :style="'width: 100%;height:'+height+'px'" />
          <div id="line-form-id" :style="'background-color: #fff;padding: 10px;overflow: auto;width: 416px;max-height:122px;height:'+(height-100)+'px'">
            <el-form ref="form">
              <el-row>
                <el-form-item>
                  <el-button type="danger" size="mini" round plain @click="GetLineInfo">Get Line Info</el-button>
                </el-form-item>
              </el-row>
            </el-form>
            <el-form :inline="true" class="demo-form-inline">
              <el-form-item>
                <el-button type="danger" size="mini">lng:{{ pointParams.lng }}</el-button>
              </el-form-item>
              <el-form-item>
                <el-button type="danger" size="mini">lng:{{ pointParams.lat }}</el-button>
              </el-form-item>
            </el-form>
          </div>
          <div id="line-button-id" class="esri-widget esri-widget--button esri-interactive">
            <span class="esri-icon-polyline" />
          </div>
        </div>
      </div>
    </el-tab-pane>
  </el-tabs>
</template>
<style>

</style>
<script>
import { loadModules,loadCss } from 'esri-loader';
import green from '@/assets/icons/green.svg';

export default {
  name: '',
  components: {},
  data() {
    return {
      activeName:"first",
      height: document.body.clientHeight - 200,
      arcGisUrlTemplate: 'http://localhost:9097/arcgis/map/{level}/{col}/{row}.png',
      lineParams:{
        name:'line',
        baseLocation: [114.33305100, 30.53750500],
        gisPoints: [],
        lineInfo:[],
      },
      pointParams:{
        lat:'30.582884765822733',
        lng:'114.25288513696107'
      }
    }
  },
  mounted() {
    // this.arcGisUrlTemplate = "http://webrd01.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=8&x={col}&y={row}&z={level}";
  },
  beforeDestroy() {
    if (this.mapView) {
      this.mapView.container = null
    }
  },
  methods: {
    handleClick(data){
      if(data.name == "first"){

      }else{
        this.initArcGisMap();
      }
    },
    initArcGisMap(){
      loadCss('/arcgis/api/4.9/esri/css/main.css')
      const option = { url: '/arcgis/api/4.9/init.js' }
      loadModules(['esri/Map', 'esri/layers/WebTileLayer', 'esri/views/MapView', 'esri/layers/GraphicsLayer', 'esri/widgets/Expand', 'esri/widgets/Fullscreen', 'esri/Graphic', 'esri/widgets/Search', 'esri/views/2d/draw/Draw', 'esri/geometry/geometryEngine'], option)
        .then(([ArcGISMap, WebTileLayer, MapView, GraphicsLayer, Expand, Fullscreen, Graphic, Search, Draw, geometryEngine]) => {
          const _this = this
          const t1 = new WebTileLayer({
            id: 't1',
            urlTemplate: this.arcGisUrlTemplate,
            copyright: 'wan.tao'
          })

          this.graphicsLayer = new GraphicsLayer()
          const arcGISMap = new ArcGISMap({
            layers: [t1, this.graphicsLayer]
          })

          this.mapView = new MapView({
            container: 'arcgis-map-id',
            map: arcGISMap,
            center: _this.lineParams.baseLocation,
            zoom: 12,
            constraints: { // 设置约束条件
              minZoom: 9,
              maxZoom: 18
            }
          })

          //地图顶层表单信息
          const lineFormExpand = new Expand({
            content: document.getElementById('line-form-id'),
            expanded: true
          })

          this.mapView.when(function() {
            //表单放上去
            _this.mapView.ui.add([
              { component:lineFormExpand, position: 'top-right' },
              { component: 'line-button-id', position: 'top-left' }
            ])
          }, function(error) {
            console.log('地图加载失败: ', error)
          })

          //加载原始线
          const gisInfo = _this.lineParams.lineInfo;
          if (gisInfo) {
            // 如果线路图不为空的话，就绘制线路
            _this.drawLine()
          }

          //加载原始点
          _this.refreshGraphic();
          _this.mapView.on('click', function (event) {
            let mapPoint = event.mapPoint;
            _this.pointParams.lng = mapPoint.longitude
            _this.pointParams.lat = mapPoint.latitude
            _this.refreshGraphic();
          })

          const draw = new Draw({
            view: _this.mapView
          })
          // 绘制线路按钮事件
          document.getElementById('line-button-id').onclick = function() {
            _this.mapView.graphics.removeAll()// 清楚当前图层
            // 创建并返回PolyLineDrawAction的实例
            const action = draw.create('polyline')
            lineFormExpand.expanded = false
            _this.draw = action
            // 聚焦视图以激活用于绘制草图的键盘快捷键
            _this.mapView.focus()
            // 倾听多段线绘图事件以立即提供视觉反馈
            // 在视图上绘制线时发送给用户.
            action.on('vertex-add', updateVertices)
            action.on('vertex-remove', updateVertices)
            action.on('cursor-update', updateVertices)
            action.on('redo', updateVertices)
            action.on('undo', updateVertices)
            action.on('draw-complete', updateVertices)
          }
          // 检查最后一个顶点是否使直线与自身相交。
          function updateVertices(event) {
            // 从返回的顶点创建多段线
            const result = createGraphic(event)
            // 如果最后一个顶点使直线与自身相交

            // 防止事件发生
            if (result.selfIntersects) {
              event.preventDefault()
            }
            if(event.type === 'draw-complete'){
              lineFormExpand.expanded = true
              _this.tempLineGis = event.vertices
            }

          }
          // 创建一个显示在视图上绘制的多段线的新图形
          function createGraphic(event) {
            const vertices = event.vertices
            _this.mapView.graphics.removeAll()
            // 表示正在绘制的多段线的图形
            const graphic = new Graphic({
              geometry: {
                type: 'polyline',
                paths: vertices,
                spatialReference: _this.mapView.spatialReference
              },
              symbol: {
                type: 'simple-line',
                color: [4, 90, 141],
                width: 4,
                cap: 'round',
                join: 'round'
              }
            })

            // 检查多段线是否与自身相交。
            const intersectingSegment = getIntersectingSegment(graphic.geometry)

            // 为相交线段添加新图形。
            if (intersectingSegment) {
              _this.mapView.graphics.addMany([graphic, intersectingSegment])
            } else {
              // 如果没有交集，只需添加表示多段线的图形
              _this.mapView.graphics.add(graphic)
            }
            // 返回交叉段
            return {
              selfIntersects: intersectingSegment
            }
          }
          // 检查直线是否与自身相交的函数
          function isSelfIntersecting(polyline) {
            if (polyline.paths[0].length < 3) {
              return false
            }
            const line = polyline.clone()

            // 从正在绘制的多段线中获取最后一段
            const lastSegment = getLastSegment(polyline)
            line.removePoint(0, line.paths[0].length - 1)
            // 如果直线与自身相交，则返回true，否则返回false
            return geometryEngine.crosses(lastSegment, line)
          }
          // 检查直线是否与自身相交。如果是，请更改最后一个段的符号，向用户提供视觉反馈。
          function getIntersectingSegment(polyline) {
            if (isSelfIntersecting(polyline)) {
              return new Graphic({
                geometry: getLastSegment(polyline),
                symbol: {
                  type: 'simple-line',
                  style: 'short-dot',
                  width: 3.5,
                  color: 'yellow'
                }
              })
            }
            return null
          }
          // 获取正在绘制的多段线的最后一段
          function getLastSegment(polyline) {
            const line = polyline.clone()
            const lastXYPoint = line.removePoint(0, line.paths[0].length - 1)
            const existingLineFinalPoint = line.getPoint(0, line.paths[0].length - 1)
            return {
              type: 'polyline',
              spatialReference: _this.mapView.spatialReference,
              hasZ: false,
              paths: [
                [
                  [existingLineFinalPoint.x, existingLineFinalPoint.y],
                  [lastXYPoint.x, lastXYPoint.y]
                ]
              ]
            }
          }
        })
    },
    drawLine() {
      if(this.lineParams.lineInfo.length > 0){
        const _this = this
        this.lineParams.gisPoints = JSON.parse(this.lineParams.lineInfo)
        const polylineGraphic = {
          geometry: {
            type: 'polyline',
            paths: [],
            spatialReference: {
              latestWkid: 3857,
              wkid: 102100,
              wkt: null
            }
          },
          symbol: {
            type: 'simple-line',
            color: [4, 90, 141],
            width: 4,
            cap: 'round',
            join: 'round'
          },
          attributes: {
            title: _this.lineParams.name
          },
          popupTemplate: {
            title: '{title}',
            content: _this.lineParams.name
          }
        }
        _this.lineParams.gisPoints.forEach((a) => {
          polylineGraphic.geometry.paths.push([Number(a.lng), Number(a.lat)])
        })
        console.log(polylineGraphic)
        _this.mapView.graphics.removeAll()
        _this.mapView.graphics.add(polylineGraphic)
      }
    },
    refreshGraphic() {
      this.mapView.graphics.removeAll();
      const lat = this.pointParams.lat;
      const lng = this.pointParams.lng;
      if (lat != undefined && lat != null && lat != '' && lng != undefined && lng != null && lng != '') {
        let symbol = {
          type: 'picture-marker',
          url: green,
          width: '26px',
          height: '30px'
        }
        const graphicPoint = {
          symbol: symbol,
          geometry: {
            type: 'point',
            longitude: lng,
            latitude: lat
          }
        }
        this.mapView.graphics.add(graphicPoint)
      }
    },
    GetLineInfo() {
      if (this.tempLineGis != null) {
        this.lineParams.gisPoints = []
        this.tempLineGis.forEach((g) => {
          this.lineParams.gisPoints.push({
            lng: g[0],
            lat: g[1]
          })
        })
      }
      this.lineParams.lineInfo = JSON.stringify(this.lineParams.gisPoints)
      this.$alert( this.lineParams.lineInfo, 'Line Info', {
        confirmButtonText: '确定',
        callback: action => {
          this.$message({
            type: 'info',
            message: `action: ${ action }`
          });
        }
      });
    },
    close() {
    },
  }
}
</script>
```

