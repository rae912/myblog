---
layout: post
title: "给 IP 匹配上国旗 Emoji"
subtitle: "Match IP with country flag"
author: "qingshan"
header-img: "img/post-bg-rwd.jpg"
header-mask: 0.4
tags:
  - Python
  - Emoji
---


最近在进行网络优化，一直在找国内的 IP。每次找 IP 还要去查询 Country Code，非常麻烦。突发奇想何不直接将 Country Code 转成国旗 Emoji。这样就直观多了。

说干就干。

## IP 批量查询 API，转 Country Code

API`https://ip-api.com/docs/api:batch`

```bash
# Example POST

curl http://ip-api.com/batch \
--data '["208.80.152.201", "91.198.174.192"]'

curl http://ip-api.com/batch?fields=isp \
--data '[{"query": "208.80.152.201", "fields": "country"}, "8.8.8.8"]'
```

response
```json
[
    {
        "status": "success",
        "country": "United States",
        "countryCode": "US",
        "region": "IL",
        "regionName": "Illinois",
        "city": "Chicago",
        "zip": "60666",
        "lat": 41.8781,
        "lon": -87.6298,
        "timezone": "America/Chicago",
        "isp": "Wikimedia Foundation Inc.",
        "org": "Wikimedia Foundation Inc",
        "as": "AS14907 Wikimedia Foundation Inc.",
        "query": "208.80.152.201"
    },
    {
        "status": "success",
        "country": "United States",
        "countryCode": "US",
        "region": "VA",
        "regionName": "Virginia",
        "city": "Ashburn",
        "zip": "20149",
        "lat": 39.03,
        "lon": -77.5,
        "timezone": "America/New_York",
        "isp": "Google LLC",
        "org": "Google Public DNS",
        "as": "AS15169 Google LLC",
        "query": "8.8.8.8"
    },
    {
        "status": "success",
        "country": "Canada",
        "countryCode": "CA",
        "region": "QC",
        "regionName": "Quebec",
        "city": "Montreal",
        "zip": "H3G",
        "lat": 45.4995,
        "lon": -73.5848,
        "timezone": "America/Toronto",
        "isp": "Le Groupe Videotron Ltee",
        "org": "Videotron Ltee",
        "as": "AS5769 Videotron Telecom Ltee",
        "query": "24.48.0.1"
    }
]
```

然后配合上一个一个 CountryCode to CountryFlag 的字典，这事就成了。
字典Link: https://github.com/rae912/ClashSubscribeIntegrate/blob/main/country_emoji_data.py


## 完整的项目Link
https://github.com/rae912/ClashSubscribeIntegrate
