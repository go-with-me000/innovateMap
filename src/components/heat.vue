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
      heat:'',
      heatData:[],
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
          console.log(result.detail.latLng)
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
            new TMap.LatLng(result.detail.location.lat, result.detail.location.lng)
          );
          that.location.longitude = result.detail.location.lng;
          that.location.latitude = result.detail.location.lat;
          that.city = result.detail.addressComponents.city;
        }
      });
      //初始化热力图并添加至map图层
      this.heat = new TMap.visualization.Heat({
        max: 120, // 热力最强阈值
        min: 0, // 热力最弱阈值
        height: 100, // 峰值高度
        gradientColor: {
          // 渐变颜色
          0.25: "rgb(0,0,255)",
          0.35: "rgb(0,255,0)",
          0.65: "rgb(255,255,0)",
          0.9: "rgb(255,0,0)",
          1.0: "rgb(255,0,0)"
        },
        radius: 30 // 最大辐射半径
      }).addTo(that.map);

      this.heat.setData(this.heatData); //设置数据
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
            this.heatData = res.data.data;
          
           this.heat.setData(this.heatData); //设置数据
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
        method: "GET", 
        params:{
          count:1
        }
      })
        .then(res => {
          if (res.data.code == 200) {
           this.heatData = res.data.data;
           this.heat.setData(this.heatData); //设置数据
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