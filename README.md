<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>心理测评+微信支付示范</title>
<style>
  /* 样式同前文，略 */
</style>
</head>
<body>

<h2>心理测评系统示范（含微信支付）</h2>

<div id="sectionUserInfo">
  <label>年龄</label><input type="number" id="inputAge" value="25" min="0" />
  <label>性别</label>
  <select id="selectGender">
    <option value="male">男性</option>
    <option value="female">女性</option>
    <option value="other">其他</option>
  </select>
  <button id="btnStart">开始答题</button>
</div>

<div id="sectionQuiz" style="display:none;">
  <div id="questionText"></div>
  <div id="optionsContainer"></div>
  <button id="btnPrev" disabled>上一题</button>
  <button id="btnNext" disabled>下一题</button>
  <div id="progressText"></div>
</div>

<div id="sectionResult" style="display:none;">
  <div id="totalScoreText"></div>
  <div id="radarChart" style="width:400px;height:400px;margin:0 auto;"></div>
  <h3>基础解析（免费）</h3>
  <pre id="basicAnalysis"></pre>
  <h3>深度解析（付费）</h3>
  <pre id="premiumAnalysis" title="点击购买深度解析" style="background:#fff0f0; color:#888; cursor:pointer;">点击购买深度解析</pre>
  <button id="btnRestart">重新开始</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>
<script>
  // 省略题库、维度定义和基础函数，参考前文完整代码
  // 这里只展示支付相关核心代码和调用示范

  let userHasPaid = false; // 付费状态

  // 微信支付调用函数
  async function callWeChatPay() {
    try {
      // 调用后端创建支付订单接口，获取支付参数
      const res = await fetch('http://localhost:3000/api/createPayment', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          amount: 1, // 1元，单位可根据后端调整
          description: '深度解析购买',
          openid: '用户openid' // 公众号授权获取
        })
      });
      const data = await res.json();
      if (data.code !== 0) {
        alert('支付参数获取失败');
        return;
      }
      const payData = data.data;

      // 调用微信JSAPI支付
      function onBridgeReady(){
        WeixinJSBridge.invoke(
          'getBrandWCPayRequest', {
            "appId": payData.appId,
            "timeStamp": payData.timeStamp,
            "nonceStr": payData.nonceStr,
            "package": payData.package,
            "signType": payData.signType,
            "paySign": payData.paySign
          },
          function(res){
            if(res.err_msg === "get_brand_wcpay_request:ok" ) {
              alert('支付成功');
              userHasPaid = true;
              generatePremiumAnalysis();
            } else {
              alert('支付失败或取消');
            }
          }
        );
      }

      if (typeof WeixinJSBridge === "undefined"){
        if( document.addEventListener ){
          document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
        } else if (document.attachEvent){
          document.attachEvent('WeixinJSBridgeReady', onBridgeReady);
          document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
        }
      } else {
        onBridgeReady();
      }

    } catch(e) {
      alert('支付请求异常');
      console.error(e);
    }
  }

  // 付费按钮点击事件
  document.getElementById('premiumAnalysis').onclick = () => {
    if (userHasPaid) {
      generatePremiumAnalysis();
    } else {
      callWeChatPay();
    }
  };

  // 生成深度解析示范（实际应调用后端AI接口）
  function generatePremiumAnalysis() {
    document.getElementById('premiumAnalysis').textContent = '这是基于AI生成的深度个性解析报告，内容更丰富、更专业。';
  }

  // 其余测评逻辑请参考之前完整代码，包含题目渲染、基础解析、雷达图绘制等
</script>

</body>
</html>
