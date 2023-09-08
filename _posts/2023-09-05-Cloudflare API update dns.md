---
title: Cloudflare API update dns
date: 2023-09-05 11:11:00 -0700
categories: [Cloudflare]
Pin:
---

#### Backup

Get dns list, please check zonesID at domain dashboard

```shell
curl -X GET "https://api.cloudflare.com/client/v4/zones/zonesID/dns_records" \
     -H "Content-Type:application/json" \
     -H "X-Auth-Key:API_KEY" \
     -H "X-Auth-Email:yourname@gmail.com"
```

Update dns record

```shell
curl --request PUT \
  --url https://api.cloudflare.com/client/v4/zones/zonesID/dns_records/domainID \
  --header 'Content-Type: application/json' \
   -H "X-Auth-Key:API_KEY" \
   -H "X-Auth-Email:yourname@gmail.com" \
  --data '{
  "content": "2600:fe80:0000:000",
  "name": "test.test.com",
  "proxied": false,
  "type": "AAAA",
  "ttl": 1
}'
```

