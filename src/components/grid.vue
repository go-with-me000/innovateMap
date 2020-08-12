<template>
  <div style="margin-top:240px;padding-bottom:80px">
    <div style="width:46%;position:relative;left:27%;top:20px;">
      <div style="display:flex">
        <v-distpicker @selected="sel"></v-distpicker>

        <Input
          v-model="address"
          placeholder="请输入详细地址"
          size="large"
          style="width: 180px;margin-left:5px;"
        />
          <Button size="large" type="primary" ghost style="margin-left:5px;font-size:20px" @click="codeAddress()">搜索</Button>
      
      </div>
    </div>
    <div id="container"></div>
    
  </div>
</template>

<script>
import VDistpicker from "v-distpicker";
import axios from "axios";
export default {
  data() {
    return {
      data: [],
      geocoder: "",
      address: "",
      addressZone: "",
      map: "",
      city: "",
      location: {
        latitude: "",
        longitude: ""
      },
     
      citydata: "",
      hexagon: "",
    };
  },
  components: { VDistpicker },
  methods: {
    sel: function(data) {
      this.citydata = data.province.value + data.city.value + data.area.value;
    },
    initMap() {
      var citylocation,
        marker = null;
      var that = this;
      var center = new TMap.LatLng(39.909897147274364, 116.39756310116866);
      //初始化地图
      this.map = new TMap.Map("container", {
        zoom: 12, //设置地图缩放级别
        pitch: 45, // 设置地图俯仰角
        center: center, //设置地图中心点坐标
        mapStyleId: "style2" //个性化样式
      });

      citylocation = new qq.maps.CityService({
        complete: function(result) {
          console.log(result.detail.latLng);
          // that.map.setCenter(result.detail.latLng);
          that.map.setCenter(
            new TMap.LatLng(result.detail.latLng.lat, result.detail.latLng.lng)
          );
          that.city = result.detail.name;
          that.location.longitude = result.detail.latLng.lng;
          that.location.latitude = result.detail.latLng.lat;
        
        }
      });

      citylocation.searchLocalCity();
      //调用地址解析类

      this.geocoder = new qq.maps.Geocoder({
        complete: function(result) {
          that.map.setCenter(
            new TMap.LatLng(
              result.detail.location.lat,
              result.detail.location.lng
            )
          );
          that.location.latitude = result.detail.location.lat;
          that.city = result.detail.addressComponents.city;
          that.city = result.detail.addressComponents.city;
        }
      });
      //初始化热力图并添加至map图层
      this.hexagon = new TMap.visualization.Hexagon({
        radius: 1000, //六边形中心点到端点的距离（半径）
        extrudable: true, //六边形是否可拔起
        colorList: [
          "rgb(1, 152, 189)",
          "rgb(73, 227, 206)",
          "rgb(216, 254, 181)",
          "rgb(254, 237, 177)",
          "rgb(254, 173, 84)",
          "rgb(209, 55, 78)"
        ], //颜色层级
        heightRange: [1, 5000], // 高度变化区间
        showRange: [1, 100] //蜂窝聚合数据显示区间
      }).addTo(that.map);

      this.hexagon.setData(this.data); //设置数据
      // 数据聚合之后才能够真正获取值域范围
      this.hexagon.setShowRange(this.hexagon.getValueRange());
    },
    loadData() {
      let URL = `http://www.pongshy.com:8081/api/showLogAndLat`;
      axios({
        url: URL,
        method: "POST",
        data: {
          name: this.city
        }
      })
        .then(res => {
          if (res.data.code == 200) {
            this.data = res.data.data;
            this.hexagon.setData(this.data); //设置数据
            // 数据聚合之后才能够真正获取值域范围
            this.hexagon.setShowRange(this.hexagon.getValueRange());
          } else {
            this.$Message.warning("数据加载出错");
          }
        })
        .catch(err => {
          this.$Message.warning("服务器出错");
        });
    },
    loadData2() {
      let URL = `http://www.pongshy.com:8081/api/showAllAbbreviation`;
      axios({
        url: URL,
        method: "GET"
      })
        .then(res => {
          if (res.data.code == 200) {
            this.data = res.data.data;
            this.hexagon.setData(this.data); //设置数据
            // 数据聚合之后才能够真正获取值域范围
            this.hexagon.setShowRange(this.hexagon.getValueRange());
          } else {
            this.$Message.warning("数据加载出错");
          }
        })
        .catch(err => {
          this.$Message.warning("服务器出错");
        });
    },
    codeAddress() {
      this.addressZone =this.citydata+this.address;
      var address = this.addressZone;
      if (address == "全国") {
        this.loadData2();
        return;
      }
      this.geocoder.getLocation(address);
    },

   
  },
  mounted() {
    this.initMap();
  },
  watch: {
    city(val, old) {
      
      this.loadData();
    },
    citydata(val, oldVal) {
      // this.addressZone = val;
      // this.address = "";
      // this.codeAddress();
    },
    address(val, oldVal) {
      // this.addressZone = this.citydata + val;
      // this.codeAddress();
    }
  }
};
</script>

<style scoped>
body {
  height: 50vh;
}
body,
button,
input,
select,
textarea {
  font: 12px/16px Verdana, Helvetica, Arial, sans-serif;
}
#info {
  width: 603px;
  padding-top: 3px;
  overflow: hidden;
}
.btn {
  width: 112px;
}
#container {
  position: relative;
  left: 27%;
  top: 40px;
  min-width: 600px;
  /* min-height:767px; */
  width: 46%;
  height: 80vh;
}
ul li h3 {
  font-size: 24px;
  font-weight: bold;
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  margin-bottom: 10px;
}
ul li #district {
  font-size: 24px;
  margin-bottom: 10px;
}
ul li p {
  font-size: 16px;
  font-weight: bold;
  /* font-family: Verdana, Geneva, Tahoma, sans-serif; */
  margin-bottom: 10px;
}
#location {
  position: relative;
  left: 27%;
  /* padding-top: 80px; */
  min-width: 600px;
  /* min-height:767px; */
  width: 46%;
}
.search {
  position: relative;
  left: 15%;
  width: 70%;
  top: 20px;
}
.medium {
  margin-top: 15%;
  position: relative;
  right: 10px;
}
</style>
<style>
.ivu-tag-size-medium {
  height: 50px;
  width: 150px;
  line-height: 48px;
}
</style>