<template>
  <div class="hello">
    <button @click="capturePicture">拍照</button>
    <button @click="pay('alipay')">支付</button>
    <button @click="pay2('alipay')">支付2 jquery版本</button>
    <button @click="getLocation()">获取定位</button>
  </div>
</template>

<script>
import axios from "axios";
axios.defaults.headers.post["Content-Type"] =
  "application/x-www-form-urlencoded"; //全局更改
import qs from "qs"; //配合qs模块转化post请求的参数，记得先npm install q
import $ from "jquery";
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data: {
    src: ""
  },
  methods: {
    capturePicture() {
      var cmr = plus.camera.getCamera();
      var res = cmr.supportedImageResolutions[0];
      var fmt = cmr.supportedImageFormats[0];
      console.log("Resolution: " + res + ", Format: " + fmt);
      cmr.captureImage(
        function(path) {
          alert("Capture image success: " + path);
        },
        function(error) {
          alert("Capture image failed: " + error.message);
        },
        { resolution: res, format: fmt }
      );
    },
    pay(id) {
      var channel = null;
      // 获取支付通道
      plus.payment.getChannels(
        function(channels) {
          // 获取你的手机端支付通道
          channel = channels[0];
        },
        function(e) {
          alert("获取支付通道失败：" + e.message);
        }
      );
      var ALIPAYSERVER =
        "http://demo.dcloud.net.cn/helloh5/payment/alipay.php";
      var WXPAYSERVER =
        "http://demo.dcloud.net.cn/helloh5/payment/wxpay.php";
      // 从服务器请求支付订单
      var PAYSERVER = "";
      if (id == "alipay") {
        PAYSERVER = ALIPAYSERVER;
      } else if (id == "wxpay") {
        PAYSERVER = WXPAYSERVER;
      } else {
        plus.nativeUI.alert("不支持此支付通道！", null, "捐赠");
        return;
      }
      // 支付
      axios({
        method: "POST",
        url: PAYSERVER,
        // params: {
        //   total: "0.01" //支付金额
        // },
        data: qs.stringify({
          total: "0.01" //支付金额
        })
      })
        .then(function(response) {
          console.log(response);
          // 支付成功之后，通知app显示成功信息
          plus.payment.request(
            channel,
            response,
            function(result) {
              plus.nativeUI.alert("支付成功！", function() {
                back();
              });
            },
            function(error) {
              plus.nativeUI.alert("支付失败：" + error.code);
            }
          );
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    pay2() {
      var channel = null;
      // 获取支付通道
      plus.payment.getChannels(
        function(channels) {
          // 获取你的手机端支付通道
          channel = channels[0];
        },
        function(e) {
          alert("获取支付通道失败：" + e.message);
        }
      );
      $.ajax({
        type:"POST",
        url:"http://demo.dcloud.net.cn/helloh5/payment/alipay.php",
        data:{
          total: "0.01"
        },
        success(data){
          console.log(data)
          plus.payment.request(
            channel,
            data,
            function(result) {
              plus.nativeUI.alert("支付成功！", function() {
                back();
              });
            },
            function(error) {
              plus.nativeUI.alert("支付失败：" + error.code);
            }
          );
        }
      })
    },
    getLocation() {
      plus.geolocation.getCurrentPosition(
        function(p) {
          alert(
            "Geolocation\nLatitude:" +
              p.coords.latitude +
              "\nLongitude:" +
              p.coords.longitude +
              "\nAltitude:" +
              p.coords.altitude
          );
        },
        function(e) {
          alert("Geolocation error: " + e.message);
        }
      );
    }
  },
  mounted() {}
};
</script>

<style scoped>
</style>
