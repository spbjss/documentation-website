---
layout: default
title: WMS map server
parent: OpenSearch Dashboards
nav_order: 5
---

# Configure WMS map server

OpenSearch Dashboards includes default map tiles, but if you need more specialized maps, you can configure OpenSearch Dashboards to use a WMS map server:

1. Open OpenSearch Dashboards at `https://<host>:<port>`. For example, [https://localhost:5601](https://localhost:5601).
1. If necessary, log in.
1. Choose **Management** and **Advanced Settings**.
1. Locate `visualization:tileMap:WMSdefaults`.
1. Change `enabled` to true and add the URL of a valid WMS map server:

   ```json
   {
     "enabled": true,
     "url": "<wms-map-server-url>",
     "options": {
       "format": "image/png",
       "transparent": true
     }
   }
   ```

Map services often have licensing fees or restrictions. You're responsible for all such considerations on any map server that you specify.
{: .note }
