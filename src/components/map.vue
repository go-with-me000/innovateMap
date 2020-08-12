<template>
  <div style="margin-top:240px;">
    <div style="width:46%;position:relative;left:27%;top:20px;">
      <div style="display:flex">
        <v-distpicker
          @selected="sel"
          :province="select.province"
          :city="select.city"
          :area="select.area"
        ></v-distpicker>

        <Input
          v-model="address"
          placeholder="请输入详细地址"
          size="large"
          style="width: 210px;margin-left:5px;"
        />
        <Button
          size="large"
          type="primary"
          ghost
          style="margin-left:5px;font-size:20px"
          @click="codeAddress()"
        >搜索</Button>
      </div>
    </div>
    <div id="container"></div>
    <div style="width:46%;position:relative;left:27%;top:70px">
      <Card :bordered="false">
        <div style="height:330px">
          <p style="text-align:center;margin-bottom:20px;font-size:30px">距离您不同距离的小区疫情数量</p>
          <Row type="flex" justify="center" class="code-row-bg" style>
            <Col span="3" offset="2">
              <div style=";position:relative;margin-top:0px;">
                <p
                  style="position:relative;font-size:34px;font-weight:bold;color:rgb(77, 121, 243);"
                >{{areaNum.three}}</p>
                <p style="position:relative;font-size:14px;font-weight:normal;color:#000;">3公里</p>
              </div>
            </Col>
            <Divider type="vertical" style="width:1px;height:80px" />

            <Col span="3" offset="2">
              <div style=";position:relative;margin-top:0px;">
                <p
                  style="position:relative;font-size:34px;font-weight:bold;color:rgb(77, 121, 243);"
                >{{areaNum.five}}</p>
                <p style="position:relative;font-size:14px;font-weight:normal;color:#000;">5公里</p>
              </div>
            </Col>
            <Divider type="vertical" style="width:1px;height:80px" />
            <Col span="3" offset="2">
              <div style=";position:relative;margin-top:0px;">
                <p
                  style="position:relative;font-size:34px;font-weight:bold;color:rgb(77, 121, 243);"
                >{{areaNum.ten}}</p>
                <p style="position:relative;font-size:14px;font-weight:normal;color:#000;">10公里</p>
              </div>
            </Col>
            <Divider type="vertical" style="width:1px;height:80px" />
            <Col span="3" offset="2">
              <div style=";position:relative;margin-top:0px;">
                <p
                  style="position:relative;font-size:34px;font-weight:bold;color:rgb(77, 121, 243);"
                >{{areaNum.twenty}}</p>
                <p style="position:relative;font-size:14px;font-weight:normal;color:#000;">20公里</p>
              </div>
            </Col>
          </Row>
          <p style="text-align:center;margin-top:20px;font-size:30px">您的综合风险评分为</p>
          <p style="text-align:center;margin-top:10px;font-size:70px;color:rgb(77, 121, 243);">{{grade}}</p>
        </div>
      </Card>
    </div>
    <div id="location">
      <ul style="position:relative;padding-top:50px">
        <li
          v-for="(item,index) in polydatas"
          :key="index"
          style="list-style-type:none;padding:0px;margin-bottom:10px;"
          v-if="item.provinceName!=''"
        >
          <Card :bordered="false">
            <Row type="flex" class="code-row-bg">
              <Col span="2">
                <img
                  src="http://114.55.93.118/group1/M00/00/00/rBA7015h5piACWgmAAAUq_qJBW8349.png"
                  width="80%"
                />
              </Col>
              <Col span="13">
                <h3>{{item.community}}</h3>
                <div style="display:flex;" id="district">
                  <Tag color="blue" size="large" style="font-size:18px;">{{item.provinceName}}</Tag>
                  <Tag color="green" size="large" style="font-size:18px;">{{item.district}}</Tag>
                </div>
                <p>{{item.provinceName}}{{item.district}}{{item.community}}</p>
              </Col>
              <Col span="4">
                <div v-if="index==0" style="top:27%;position:absolute;left:15px;">
                  <span style="font-size:18px;color:red;font-weight:bold">离你最近</span>
                </div>
              </Col>
              <Col span="5">
                <div style="font-size:25px;padding-top:30px;">
                  <span>{{item.distance}}</span>
                  <span>&nbsp;公里</span>
                </div>
              </Col>
            </Row>
          </Card>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import VDistpicker from "v-distpicker";
import axios from "axios";
export default {
  data() {
    return {
      check: false,
      data: [],
      geocoder: "",
      address: "",
      addressZone: "",
      map: "",
      city: "",
      areaNum: {
        three: 0,
        five: 0,
        ten: 0,
        twenty: 0
      },
      grade:0,
      select: {
        province: "",
        city: "",
        area: ""
      },
      location: {
        latitude: "",
        longitude: ""
      },
      polydatas: [
        {
          provinceName: "",
          district: "",
          community: "",
          distance: -1
        },
        {
          provinceName: "",
          district: "",
          community: "",
          distance: -1
        },
        {
          provinceName: "",
          district: "",
          community: "",
          distance: -1
        },
        {
          provinceName: "",
          district: "",
          community: "",
          distance: -1
        },
        {
          provinceName: "",
          district: "",
          community: "",
          distance: -1
        }
      ],
      citydata: "",
      markersArray: [],
      groundcity: "",
      first: true,
      latLng: "",
      
    };
  },
  components: { VDistpicker },
  methods: {
    sel: function(data) {
      this.citydata = data.province.value + data.city.value + data.area.value;
      this.groundcity = data.city.value;
    },
    init() {
      var that = this;
      var citylocation,
        marker = null;
      var center = new qq.maps.LatLng(32.916527, 116.397128);
      this.map = new qq.maps.Map(document.getElementById("container"), {
        center: center,
        zoom: 13,
        mapStyleId: "style2"
      });
      //获取城市列表接口设置中心点
      citylocation = new qq.maps.CityService({
        complete: function(result) {
          that.map.setCenter(result.detail.latLng);
          that.city = result.detail.name;

          that.location.longitude = result.detail.latLng.lng;
          that.location.latitude = result.detail.latLng.lat;
          that.latLng = new qq.maps.LatLng(
            that.location.latitude,
            that.location.longitude
          );
          setTimeout(function() {
            var marker = new qq.maps.Marker({
              position: new qq.maps.LatLng(
                that.location.latitude,
                that.location.longitude
              ),
              animation: qq.maps.MarkerAnimation.DROP,
              map: that.map
            });
            that.geocoder.getAddress(that.latLng);

            //marker.setAnimation(qq.maps.Animation.DROP);
          }, 1000);
        }
      });

      citylocation.searchLocalCity();
      //调用地址解析类
      var center2 = new qq.maps.LatLng(
        that.location.latitude,
        that.location.longitude
      );
      this.geocoder = new qq.maps.Geocoder({
        complete: function(result) {
          that.map.setCenter(result.detail.location);
          that.city = result.detail.addressComponents.city;
          that.location.longitude = result.detail.location.lng;
          that.location.latitude = result.detail.location.lat;

          if (that.first) {
            that.select.province = result.detail.addressComponents.province;
            that.select.city = result.detail.addressComponents.city;
            that.select.area = result.detail.addressComponents.district;
            that.first = false;
          }
          setTimeout(function() {
            var marker = new qq.maps.Marker({
              position: new qq.maps.LatLng(
                that.location.latitude,
                that.location.longitude
              ),
              animation: qq.maps.MarkerAnimation.DROP,
              map: that.map
            });

            //marker.setAnimation(qq.maps.Animation.DROP);
          }, 1000);
        }
      });

      //marker.setAnimation(qq.maps.Animation.DROP);
    },
    loadData() {
      let URL = `http://www.pongshy.com:8081/api/Display`;

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

            this.market();
          } else {
            this.$Message.warning("数据加载出错");
          }
        })
        .catch(err => {
          this.$Message.warning("服务器出错");
        });
    },
    loadData2() {
      let URL = `http://www.pongshy.com:8081/api/show`;

      axios({
        url: URL,
        method: "GET"
      })
        .then(res => {
          if (res.data.code == 200) {
            this.data = res.data.data;
            this.market();
          } else {
            this.$Message.warning("数据加载出错");
          }
        })
        .catch(err => {
          this.$Message.warning("服务器出错");
        });
    },
    codeAddress() {
      this.addressZone = this.citydata + this.address;

      var address = this.addressZone;
      if (address == "全国") {
        this.loadData2();
        return;
      }
      this.areaNum.three=this.areaNum.five=this.areaNum.ten=this.areaNum.tewnty=0;

      for (var i = 0; i < this.polydatas.length; i++) {
        this.polydatas[i].provinceName = "";
        this.polydatas[i].district = "";
        this.polydatas[i].community = "";
        this.polydatas[i].distance = -1;
      }
      this.check = true;

      this.geocoder.getLocation(address);
    },

    market() {
      var that = this;

      let length = this.data.length < 5 ? this.data.length : 5;
      var latlngs = [];
      for (let i = 0; i < this.data.length; i++) {
        var marker = new qq.maps.Marker({
          position: new qq.maps.LatLng(
            this.data[i].latitude,
            this.data[i].longitude
          ),
          // icon: icon,
          map: that.map
        });

        var anchor = new qq.maps.Point(0, 0),
          size = new qq.maps.Size(100, 100),
          origin = new qq.maps.Point(0, 0),
          markerIcon = new qq.maps.MarkerImage(
            "http://114.55.93.118/group1/M00/00/00/rBA7015h5piACWgmAAAUq_qJBW8349.png",
            size,
            origin,
            anchor
          );
        marker.setIcon(markerIcon);
        this.markersArray.push(marker);
        var infoWin = new qq.maps.InfoWindow({
          map: that.map
        });
        latlngs[i] = new qq.maps.LatLng(
          this.data[i].latitude,
          this.data[i].longitude
        );
        qq.maps.event.addListener(marker, "click", function() {
          infoWin.open();
          infoWin.setContent(
            '<div style="text-align:center;white-space:' +
              'nowrap;">' +
              that.data[i].community +
              "</div>"
          );
          infoWin.setPosition(latlngs[i]);
        });

        var start = new qq.maps.LatLng(
          that.location.latitude,
          that.location.longitude
        );
        var end = new qq.maps.LatLng(
          that.data[i].latitude,
          that.data[i].longitude
        );
        var distance =
          Math.round(
            qq.maps.geometry.spherical.computeDistanceBetween(start, end) * 10
          ) / 10;

        this.computeNum(distance);
        if (
          distance < this.polydatas[0].distance ||
          this.polydatas[0].distance == -1
        ) {
          this.polydatas[0].distance = distance;
          this.polydatas[0].provinceName = this.data[i].provincename;
          this.polydatas[0].district = this.data[i].districtname;
          this.polydatas[0].community = this.data[i].community;
          let j = 1;
          while (
            j < length &&
            (distance < this.polydatas[j].distance ||
              this.polydatas[j].distance == -1)
          ) {
            let temp = this.polydatas[j];
            this.polydatas[j] = this.polydatas[j - 1];
            this.polydatas[j - 1] = temp;
            j++;
          }
        }
      }
      this.polydatas.reverse();
      for (let i = 0; i < length; i++) {
        this.polydatas[i].distance = (
          this.polydatas[i].distance / 1000
        ).toFixed(2);
      }
      this.computeFinalNum();
    },
    computeFinalNum(){
      let num = this.areaNum.three*0.55+this.areaNum.five*0.35+this.areaNum.ten*0.13+this.areaNum.twenty*0.02;
      this.grade=num;
      this.grade = this.grade.toFixed(2);
    },
    computeNum(distance) {
      distance = (distance / 1000);
      if (distance <= 3) {
        
        this.areaNum.three++;
      } else if (distance <= 5) {
        this.areaNum.five++;
      } else if (distance <= 10) {
        this.areaNum.ten++;
      } else if (distance <= 20) {
        this.areaNum.twenty++;
      } else {
      }
    }
  },
  mounted() {
    this.init();
    // this.addressZone =
    //   this.select.province + this.select.city + this.select.area;
  },
  watch: {
    city(val, old) {
      this.check = false;
      for (let i in this.markersArray) {
        this.markersArray[i].setMap(null);
      }
      this.markersArray.length = 0;

      this.loadData();
    },

    addressZone(val, oldVal) {
      var that = this;
      setTimeout(function() {
        if (that.check) {
          for (var i = 0; i < that.polydatas.length; i++) {
            that.polydatas[i].provinceName = "";
            that.polydatas[i].district = "";
            that.polydatas[i].community = "";
            that.polydatas[i].distance = -1;
          }

          that.loadData();
          that.check = false;
        }
      }, 500);
    },
    groundcity(val, oldVal) {}
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
  top: 30px;
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