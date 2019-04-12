---
layout: post
title: "从前端获取内网IP地址"
subtitle: "How to get internal IP from frontend"
author: "qingshan"
header-img: "img/post-bg-js-version.jpg"
header-mask: 0.4
tags:
  - 工作
---

今天工作中遇到一个要求，就是要获取用户在交互界面上的访问行为。为了收集并分析以用于产品改进。虽说其实界面有登录逻辑的，登录之后就可以将IP与用户名进行绑定。但是这种方式有两个问题：

1. 不登陆的用户无法识别
2. 在离线情况下，行为无法被收集

因此，经过调研，决定直接通过前端想办法获取IP进行绑定。

因为产品使用场景是内网，所以通过公共的外网服务接口返回IP这种方法行不通，因为肯定都是识别的是同一个外网IP。那么有什么办法能够获取内网地址呢？

还真是有办法，虽然比较曲折一点。那就是通过WebRTC。基本原理就是建立一个WebRTC服务，然后去连接它。通过这个服务获取IP。因为是内网访问，自然得到的就是内网IP。

Vue中的代码如下，可以设置成全局方法，到处调用：
```javascript
export function _getInterIp() {
    var RTCPeerConnection = window.RTCPeerConnection || window.webkitRTCPeerConnection || window.mozRTCPeerConnection;
    if (RTCPeerConnection) (function () {
        var rtc = new RTCPeerConnection({iceServers:[]});
        if (1 || window.mozRTCPeerConnection) {
            rtc.createDataChannel('', {reliable:false});
        };

        rtc.onicecandidate = function (evt) {
            if (evt.candidate) grepSDP("a="+evt.candidate.candidate);
        };
        rtc.createOffer(function (offerDesc) {
            grepSDP(offerDesc.sdp);
            rtc.setLocalDescription(offerDesc);
        }, function (e) { console.warn("offer failed", e); });


        var addrs = Object.create(null);
        addrs["0.0.0.0"] = false;
        function updateDisplay(newAddr) {
            if (newAddr in addrs) return;
            else addrs[newAddr] = true;
            var displayAddrs = Object.keys(addrs).filter(function (k) { return addrs[k]; });
            for(var i = 0; i < displayAddrs.length; i++){
                if(displayAddrs[i].length > 16){
                    displayAddrs.splice(i, 1);
                    i--;
                }
            }
            console.log(displayAddrs[0]);      //打印出内网ip
            return displayAddrs[0]
        }

        function grepSDP(sdp) {
            var hosts = [];
            sdp.split('\r\n').forEach(function (line, index, arr) {
                if (~line.indexOf("a=candidate")) {
                    var parts = line.split(' '),
                        addr = parts[4],
                        type = parts[7];
                    if (type === 'host') updateDisplay(addr);
                } else if (~line.indexOf("c=")) {
                    var parts = line.split(' '),
                        addr = parts[2];
                    updateDisplay(addr);
                }
            });
        }
    })();
    else{
        console.log("请使用主流浏览器：chrome,firefox,opera,safari");
    }
}
```
