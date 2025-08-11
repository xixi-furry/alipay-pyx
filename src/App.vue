<script setup>
import { ref, onMounted, computed, onUnmounted } from "vue";
const now = ref(new Date());
const end = ref(new Date(Date.now() + 3 * 24 * 60 * 60 * 1000)); // 3天后

const timerID = setInterval(() => {
  now.value = new Date();
}, 1000);

const timeDifference = computed(() => Math.max(end.value - now.value, 0));

const days = computed(() => Math.floor(timeDifference.value / (1000 * 60 * 60 * 24)));
const hours = computed(() => {
  const h = Math.floor((timeDifference.value % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  return h.toString().padStart(2, "0"); // 补零到2位
});
const minutes = computed(() => {
  const m = Math.floor((timeDifference.value % (1000 * 60 * 60)) / (1000 * 60));
  return m.toString().padStart(2, "0"); // 补零到2位
});
const seconds = computed(() => {
  const s = Math.floor((timeDifference.value % (1000 * 60)) / 1000);
  return s.toString().padStart(2, "0"); // 补零到2位
});

onUnmounted(() => {
  clearInterval(timerID);
});

// 支付宝用户ID和赏金二维码token - 从环境变量中读取
const alipayUserId = ref((window.ENV_CONFIG && window.ENV_CONFIG.VITE_ALIPAY_USER_ID) || "2088922557117308");
const pathID = ref((window.ENV_CONFIG && window.ENV_CONFIG.VITE_PATH_ID) || "_4btsDsC8UQ0z3FQzrPxjOr");

// 跳转地址
const jumpURL = ref(
  `https://render.alipay.com/p/c/180020570000138521/py-propagate-share.html?appletInfo=kPPFvOxaCL2711fShZtlW101lIpDtOPI&bizCode=SNS_NFC_FRIEND&caprMode=sync&chInfo=&shareId=${alipayUserId.value}&fxzjshareChinfo=ch_share__chsub_CopyLink&apshareid=8708FA59-61D1-461C-B523-4D47480B4377&shareBizType=${pathID.value}&shareTimestamp=1754295378692`
);

// 使用ref创建响应式变量
const amount = ref(0);
const displayAmount = computed(() => {
  return amount.value.toFixed(2);
});

const startAmountAnimation = () => {
  const target = [3, 4, 5][Math.floor(Math.random() * 3)]; // 随机选择 3、4 或 5

  const duration = 1500;
  const steps = 100;
  const increment = target / steps;
  let count = 0;

  const interval = setInterval(() => {
    amount.value = parseFloat((amount.value + increment).toFixed(2));
    count++;
    if (count >= steps) {
      amount.value = target;
      clearInterval(interval);
    }
  }, duration / steps);
};

// 获取设备（操作系统或者指定APP）
const getMobileOS = () => {
  const ua = navigator.userAgent || navigator.vendor || window.opera;
  // 判断是否为抖音（Douyin）
  if (/aweme|douyin/i.test(ua)) {
    return "Douyin";
  }
  // 判断是否为 Android
  if (/android/i.test(ua)) {
    return "Android";
  }
  // 判断是否为 iOS
  if (/iPad|iPhone|iPod/.test(ua) && !window.MSStream) {
    return "iOS";
  }
  // 判断是否为 PC
  if (/Windows NT|Macintosh/.test(ua)) {
    return "PC";
  }
  return "unknown";
};

// 判断是否是微信浏览器
const isWeChat = () => {
  return /MicroMessenger/i.test(navigator.userAgent);
};

// 调试模式的电脑系统检测
const isPC = () => {
  // 使用userAgent替代已弃用的platform
  const ua = navigator.userAgent.toLowerCase();

  // 检查是否为移动设备
  const isMobile = /android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i.test(ua);

  // 额外检查屏幕宽度，通常移动设备屏幕较窄
  const isSmallScreen = window.innerWidth < 768;

  // 如果是移动设备或小屏幕，则不是PC
  return !(isMobile || isSmallScreen);
};

// Android设备跳转
const jumpTo_Android = () => {
  if (isWeChat()) {
    // console.log(window.location.href + "/download?redirect_url=" + window.location.href);
    weui.alert("请在右上角[···]选择在浏览器打开！");
    // 如果是安卓微信内
    // alert("请在右上角[···]选择在浏览器打开！");
  } else {
    // 如果不是微信浏览器
    // 跳转到指定页面
    location.href = jumpURL.value;
  }
};

// iOS设备跳转
const jumpTo_iOS = () => {
  if (confirm("点击右上角[···]在浏览器打开可正常跳转！")) {
    // 跳转入口1
    location.href = jumpURL.value;
  } else {
    // 跳转入口2
    location.href = `https://ace.tb.cn/t?smburl=tbopen://m.taobao.com/tbopen/index.html?h5Url=${jumpURL.value}`;
  }
};

// 不同的设备不同的跳转情况
const jumpTo = () => {
  if (getMobileOS() == "Douyin") {
    // 在抖音打开
    location.href = jumpURL.value;
  } else if (isPC() && getMobileOS() !== "Android") {
    // 在电脑打开（调试模式）
    weui.alert("仅限手机端打开");
  } else if (getMobileOS() == "PC") {
    // 在电脑打开
    weui.alert("仅限手机端打开");
  } else if (getMobileOS() == "Android") {
    // 在Android设备打开
    jumpTo_Android();
  } else if (getMobileOS() == "iOS") {
    // 在iOS设备打开
    jumpTo_iOS();
  } else {
    // 直跳
    location.href = jumpURL.value;
  }
};

// 页面加载完成后执行
onMounted(() => {
  // 如果不是微信内也不是电脑系统，直接跳转
  if (!isWeChat() && !isPC()) {
    location.href = jumpURL.value;
  }

  // 启动金额动画
  startAmountAnimation();
});
</script>

<template>
  <div class="mycontainer">
    <div class="myhead">
      <div class="title">碰一下支付</div>
      <div class="subtitle">天天领红包</div>
    </div>

    <div class="imhHB" @click="jumpTo">
      <div class="tip">
        <div class="tipTitle">碰一下支付红包</div>
      </div>
      <div class="moneyKuang">
        <div class="money">
          <div class="moneynum">{{ displayAmount }}</div>
          <div class="units">元</div>
        </div>
        <div class="propaganda">碰一下支付专享</div>
        <div class="timeTitle">{{ days }}天{{ hours }}:{{ minutes }}:{{ seconds }}后失效</div>
      </div>
      <img class="img" src="./assets/hb.webp" alt="" />
      <div class="imhHBTitle">开心收下</div>
    </div>
    <div class="zwf"></div>
    <div class="zfhl" @click="jumpTo">
      <img class="img" src="./assets/1754020265330.webp" alt="" />
      <div class="zfhlKuang">
        <div class="zfhlKuangTitle">
          你有<span class="zfhlKuangTitleNum">{{ displayAmount }}元</span>红包待使用
        </div>
        <div class="zfhlDoscKuang">
          <div class="zfhlKuangDosc">{{ days }}天{{ hours }}:{{ minutes }}:{{ seconds }}后过期</div>
          <div class="sy">去使用></div>
        </div>
      </div>
    </div>
    <div class="zfbkey">
      <div class="zfbKeyTitle">碰一下支付如何使用</div>
      <div class="zfbKeyContent">1、解锁手机（不用打开支付宝）</div>
      <div class="zfbKeyContent">2、用手机背部碰小蓝环</div>
      <div class="zfbKeyContent">3、点击"打开"或“确认付款”完成支付</div>
    </div>
  </div>
</template>

<style scoped>
@font-face {
  font-family: "DingTalk-JinBuTi";
  src: url("./assets/font/DingTalk-JinBuTi.woff2") format("woff2"),
    url("./assets/font/DingTalk-JinBuTi.woff") format("woff");
  font-weight: normal;
  font-style: normal;
  font-display: swap;
}

.mycontainer {
  background-color: #fff;
  background: linear-gradient(179deg, #b5cfff 0% 0%, #e4e9fc 55%);
  min-height: 100vh;
  width: 100%;
  padding-top: 40px;
  font-family: "DingTalk-JinBuTi", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell,
    "Open Sans", "Helvetica Neue", sans-serif;
  box-sizing: border-box;
  overflow: hidden;
  user-select: none;
}

.myhead {
  text-align: center;
  margin-bottom: 20px;
}

.title {
  font-size: 35px;
  font-weight: bold;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  font-family: "DingTalk-JinBuTi", sans-serif;
  color: #181825;
}

.subtitle {
  width: 100%;
  font-size: 35px;
  font-weight: bold;
  font-family: "DingTalk-JinBuTi", serif;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  /* 设置背景为上下渐变 */
  background-image: linear-gradient(to bottom, #363555, #414068a1);
  /* 将背景应用到文字上 */
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  /* 可选：确保元素能显示背景 */
  display: inline-block;
  /* 可选：增强显示效果 */
  font-weight: bold;
}

.imhHB {
  width: 70%;
  height: 290px;
  position: relative;
  overflow: hidden;
  margin: 0 auto;
}

.tip {
  width: 100%;
  height: 30px;
  line-height: 30px;
  overflow: hidden;
  position: absolute;
  top: 2px;
  left: 0;
}

.tipTitle {
  width: fit-content;
  height: 38px;
  line-height: 38px;
  border-bottom-left-radius: 16px;
  border-bottom-right-radius: 16px;
  text-align: center;
  color: #3b3b5d;
  margin: 0 auto;
  display: flex;
  font-size: 18px;
  align-items: center;
  justify-content: center;
  justify-items: center;
}

.moneyKuang {
  width: fit-content;
  height: 100px;
  position: absolute;
  top: 30px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 5;
  box-sizing: border-box;
}

.money {
  width: fit-content;
  font-size: 56px;
  font-weight: bold;
  color: #ff3b2f;
  padding: 10px 0 10px 0;
  box-sizing: border-box;
  display: flex;
  align-items: end;
}

.moneynum {
  max-width: 160px;
  min-width: 140px;
  width: fit-content;
  text-align: center;
}

.units {
  font-size: 24px;
  height: 37px;
  color: #333333;
}

.propaganda {
  font-size: 16px;
  color: #676767;
  text-align: center;
  margin-bottom: 35px;
}

.timeTitle {
  font-size: 16px;
  color: #ffffffbd;
  text-align: center;
  letter-spacing: 1px;
}

.imhHBTitle {
  position: absolute;
  bottom: 40px;
  left: 0;
  width: 100%;
  height: 30px;
  line-height: 30px;
  z-index: 3;
  color: #181825;
  font-weight: bold;
  text-align: center;
  font-size: 23px;
  display: inline-block;
  /* 必须设置，transform 才能生效 */
  animation: textScale 1.5s infinite ease-in-out;
}

@keyframes textScale {
  0% {
    transform: scale(1);
  }

  50% {
    transform: scale(1.1);
    /* 放大到 1.3 倍 */
  }

  100% {
    transform: scale(1);
  }
}

.img {
  width: 100%;
  height: 100%;
}

.zfhl {
  width: 90%;
  height: 80px;
  margin: 10px auto;
  overflow: hidden;
  border-radius: 10px;
  box-sizing: border-box;
  position: relative;
}

.zfhlKuang {
  position: absolute;
  top: 5px;
  left: 0;
  width: 100%;
  box-sizing: border-box;
  padding: 10px;
  font-size: 16px;
}

.zfhlKuangTitle {
  height: 30px;
  line-height: 30px;
  color: #07070ae3;
}

.zfbkey {
  width: 90%;
  height: fit-content;
  background-color: #fff;
  margin: 0 auto;
  border-radius: 10px;
  box-sizing: border-box;
  padding: 14px 20px;
}

.zfbKeyTitle {
  width: 100%;
  height: 30px;
  line-height: 30px;
  text-align: center;
  font-size: 18px;
  font-weight: bold;
  color: #07070ae3;
}

.zfbKeyContent {
  font-size: 14px;
  margin-top: 10px;
  color: #2f2f2fb8;
}

.zfhlKuangTitleNum {
  color: #ff3b2f;
}
.zfhlDoscKuang {
  display: flex;
  align-items: center;
}
.zfhlKuangDosc {
  width: 130px;
  color: #07070ad1;
  font-size: 14px;
}

.sy {
  color: #2f74ff;
  font-size: 14px;
}
.zwf {
  height: 20px;
}
</style>
