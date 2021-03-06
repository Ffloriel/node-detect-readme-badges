(mt) Stats
==========

(mt) Stats is a specialised proxy server that gets your server's statistics from [MediaTemple's API servers](http://mediatemple.net/api/). It was designed to be used with the [(mt) Stats Viewer](https://github.com/meloncholy/mt-stats-viewer) front end, but may even be more generally useful.

If you want to read more about how it works and some potential problems, there's a surprisingly (to me, at least) long post [here](http://meloncholy.com/blog/using-d3-for-realtime-webserver-stats).


Setting up (mt) Stats
---------------------

- [Get an API key](https://ac.mediatemple.net/api) for your MediaTemple server. You'll also need your service ID later, which you can get by visiting `https://api.mediatemple.net/api/v1/services/ids.json?apikey=XXXXX` (I only have one service ID as I have one server, but apparently you could see more.)

- Rename the config file `node_modules/mt-stats/config/mt-stats-sample.json` to `mt-stats.json` and change the service ID and API key to match your server and key.


Using (mt) Stats with your app
------------------------------

Here's a quick example of using (mt) Stats with Express.

```javascript
var app = require("express").createServer();
var mtStats = require("mt-stats");

app.get("/api/:range?", mtStats);

app.listen(3000);
```

(mt) Stats will respond differently depending on the range passed to it as part of the path.

- **No range** - Server stats for current time
- **Range is one of** `5min`, `15min`, `30min`, `1hour`, `1day`, `1week`, `1month`, `3month`, `1year` - Server stats for that past period will be returned, using the MediaTemple API server's default resolution for that period
- **Range is [0-9]+** - Server stats for the past _number_ of seconds will be returned, using the resolution in the settings file (probably; see below)
- **Range is [0-9]+-[0-9]+** - Server stats covering the range from the first number (time since Linux epoch in seconds) to the second number will be returned, using the resolution in the settings file


Settings
--------

Please rename `mt-stats-sample.json` to `mt-stats.json`.

- **serviceId** - Your server's service ID
- **apiKey** - Your MediaTemple API key
- **mode** - Doesn't do anything here, but its presence gives me a warm, comforting glow
- **rootPath** - The root URL path for all API calls
- **interval** - Server polling interval for client. MediaTemple's stats update every 15s
- **ranges** - For each `range`, the `resolution` at which to request data from the API (e.g. every 15 seconds) and the maximum span (`step`) to request in one go (to stop the API server objecting). Range is the maximum timespan at which to use that resolution and step
- **metrics** - Metrics supplied by the API. `apiKey` is the key name in JSON objects and `niceName` is the name to use on the graphs
- **definedRanges** - MediaTemple also supports some default intervals that can be requested with these URLs, e.g. [this URL](http://bits.meloncholy.com/mt-stats/api/5min) will serve up the last 5 minutes' data
- **currentUrl** - API server URL from which to get the current stats. `%SERVICEID` and `%APIKEY` are replaced with your service ID and API key
- **historyUrl** - API server URL to request stats going back for the past X seconds, e.g. [this URL](http://bits.meloncholy.com/mt-stats/api/300) will also give the past 5 minutes' data
- **rangeUrl** - API server URL to get stats covering a specified time range

```javascript
{
	"serviceId": 000000,
	"apiKey": "XXXXXXXX",
	"mode": "production",
	"rootPath": "/api/",
	"interval": 15000,
	"ranges": [
		{ "range": 3600, "resolution": 15, "step": 3600 },
		{ "range": 43200, "resolution": 120, "step": 28800 },
		{ "range": 86400, "resolution": 240, "step": 57600 },
		{ "range": 604800, "resolution": 1800, "step": 432000 }
	],
	"metrics": [
		{ "apiKey": "cpu", "niceName": "CPU %" },
		{ "apiKey": "memory", "niceName": "Memory %" },
		{ "apiKey": "load1Min", "niceName": "Load 1 min" },
		{ "apiKey": "load5Min", "niceName": "Load 5 min" },
		{ "apiKey": "load15Min", "niceName": "Load 15 min" },
		{ "apiKey": "processes", "niceName": "Processes" },
		{ "apiKey": "diskSpace", "niceName": "Disk space" },
		{ "apiKey": "kbytesIn", "niceName": "kb in / sec" },
		{ "apiKey": "kbytesOut", "niceName": "kb out / sec" },
		{ "apiKey": "packetsIn", "niceName": "Packets in / sec" },
		{ "apiKey": "packetsOut", "niceName": "Packets out / sec" }
	],
	"definedRanges": ["5min", "15min", "30min", "1hour", "1day", "1week", "1month", "3month", "1year"],
	"currentUrl": "https://api.mediatemple.net/api/v1/stats/%SERVICEID.json?apikey=%APIKEY",
	"historyUrl": "https://api.mediatemple.net/api/v1/stats/%SERVICEID/%RANGE.json?apikey=%APIKEY",
	"rangeUrl": "https://api.mediatemple.net/api/v1/stats/%SERVICEID.json?start=%START&end=%END&&resolution=%RESOLUTION&apikey=%APIKEY"
}
```

More on stats resolution
------------------------

MediaTemple may or may not respect the stats resolution you request, e.g. there's a minimum resolution of 15s and a request for 2 hour intervals will be returned at a resolution of 60 minutes.

The API server will also only serve up a fairly small amount of data at a given resolution - smaller than I'd like, so the server divides up the client's range into several requests and combines them before returning. The interval at which to split a single request into multiple API queries is the `step` if this is less than the `range` for that range. So if you wanted a week's worth of data at 15 second intervals, you could add `{"range": 604800, "resolution": 15, "step": 3600 }` (but please don't as you'll hit the API server 168 times!). All numbers are seconds.

The metrics bit of the config file is currently set up to return up to a week's worth of data at once; it will return more, but at a resolution that would hammer the MediaTemple server rather, so please add some more ranges if you want to do that.


Dependencies
------------

- [Konphyg](https://github.com/pgte/konphyg)


Legal fun
---------

Copyright &copy; 2012 Andrew Weeks http://meloncholy.com

(mt) Stats is licensed under the [MIT licence](http://meloncholy.com/licence).


Me
--

I have a [website](http://meloncholy.com) and a [Twitter](https://twitter.com/meloncholy). Please come and say hi if you'd like or if something's not working; be lovely to hear from you.
"
"(mt) Stats\r\n==========\r\n\r\n(mt) Stats is a specialised proxy server that gets your server's statistics from [MediaTemple's API servers](http://mediatemple.net/api/). It was designed to be used with the [(mt) Stats Viewer](https://github.com/meloncholy/mt-stats-viewer) front end, but may even be more generally useful. \r\n\r\nIf you want to read more about how it works and some potential problems, there's a surprisingly (to me, at least) long post [here](http://meloncholy.com/blog/using-d3-for-realtime-webserver-stats).\r\n\r\n\r\nSetting up (mt) Stats\r\n---------------------\r\n\r\n- [Get an API key](https://ac.mediatemple.net/api) for your MediaTemple server. You'll also need your service ID later, which you can get by visiting `https://api.mediatemple.net/api/v1/services/ids.json?apikey=XXXXX` (I only have one service ID as I have one server, but apparently you could see more.)\r\n\r\n- Rename the config file `node_modules/mt-stats/config/mt-stats-sample.json` to `mt-stats.json` and change the service ID and API key to match your server and key.\r\n\r\n\r\nUsing (mt) Stats with your app\r\n------------------------------\r\n\r\nHere's a quick example of using (mt) Stats with Express. \r\n\r\n```javascript\r\nvar app = require(\"express\").createServer();\r\nvar mtStats = require(\"mt-stats\");\r\n\r\napp.get(\"/api/:range?\", mtStats);\r\n\r\napp.listen(3000);\r\n```\r\n\r\n(mt) Stats will respond differently depending on the range passed to it as part of the path.\r\n\r\n- **No range** - Server stats for current time\r\n- **Range is one of** `5min`, `15min`, `30min`, `1hour`, `1day`, `1week`, `1month`, `3month`, `1year` - Server stats for that past period will be returned, using the MediaTemple API server's default resolution for that period\r\n- **Range is [0-9]+** - Server stats for the past _number_ of seconds will be returned, using the resolution in the settings file (probably; see below)\r\n- **Range is [0-9]+-[0-9]+** - Server stats covering the range from the first number (time since Linux epoch in seconds) to the second number will be returned, using the resolution in the settings file\r\n\r\n\r\nSettings\r\n--------\r\n\r\nPlease rename `mt-stats-sample.json` to `mt-stats.json`.\r\n\r\n- **serviceId** - Your server's service ID\r\n- **apiKey** - Your MediaTemple API key\r\n- **mode** - Doesn't do anything here, but its presence gives me a warm, comforting glow\r\n- **rootPath** - The root URL path for all API calls\r\n- **interval** - Server polling interval for client. MediaTemple's stats update every 15s\r\n- **ranges** - For each `range`, the `resolution` at which to request data from the API (e.g. every 15 seconds) and the maximum span (`step`) to request in one go (to stop the API server objecting). Range is the maximum timespan at which to use that resolution and step\r\n- **metrics** - Metrics supplied by the API. `apiKey` is the key name in JSON objects and `niceName` is the name to use on the graphs\r\n- **definedRanges** - MediaTemple also supports some default intervals that can be requested with these URLs, e.g. [this URL](http://bits.meloncholy.com/mt-stats/api/5min) will serve up the last 5 minutes' data\r\n- **currentUrl** - API server URL from which to get the current stats. `%SERVICEID` and `%APIKEY` are replaced with your service ID and API key\r\n- **historyUrl** - API server URL to request stats going back for the past X seconds, e.g. [this URL](http://bits.meloncholy.com/mt-stats/api/300) will also give the past 5 minutes' data\r\n- **rangeUrl** - API server URL to get stats covering a specified time range\r\n\r\n```javascript\r\n{\r\n\t\"serviceId\": 000000,\r\n\t\"apiKey\": \"XXXXXXXX\",\r\n\t\"mode\": \"production\",\r\n\t\"rootPath\": \"/api/\",\r\n\t\"interval\": 15000,\r\n\t\"ranges\": [\r\n\t\t{ \"range\": 3600, \"resolution\": 15, \"step\": 3600 },\r\n\t\t{ \"range\": 43200, \"resolution\": 120, \"step\": 28800 },\r\n\t\t{ \"range\": 86400, \"resolution\": 240, \"step\": 57600 },\r\n\t\t{ \"range\": 604800, \"resolution\": 1800, \"step\": 432000 }\r\n\t],\r\n\t\"metrics\": [\r\n\t\t{ \"apiKey\": \"cpu\", \"niceName\": \"CPU %\" },\r\n\t\t{ \"apiKey\": \"memory\", \"niceName\": \"Memory %\" },\r\n\t\t{ \"apiKey\": \"load1Min\", \"niceName\": \"Load 1 min\" },\r\n\t\t{ \"apiKey\": \"load5Min\", \"niceName\": \"Load 5 min\" },\r\n\t\t{ \"apiKey\": \"load15Min\", \"niceName\": \"Load 15 min\" },\r\n\t\t{ \"apiKey\": \"processes\", \"niceName\": \"Processes\" },\r\n\t\t{ \"apiKey\": \"diskSpace\", \"niceName\": \"Disk space\" },\r\n\t\t{ \"apiKey\": \"kbytesIn\", \"niceName\": \"kb in / sec\" },\r\n\t\t{ \"apiKey\": \"kbytesOut\", \"niceName\": \"kb out / sec\" },\r\n\t\t{ \"apiKey\": \"packetsIn\", \"niceName\": \"Packets in / sec\" },\r\n\t\t{ \"apiKey\": \"packetsOut\", \"niceName\": \"Packets out / sec\" }\r\n\t],\r\n\t\"definedRanges\": [\"5min\", \"15min\", \"30min\", \"1hour\", \"1day\", \"1week\", \"1month\", \"3month\", \"1year\"],\r\n\t\"currentUrl\": \"https://api.mediatemple.net/api/v1/stats/%SERVICEID.json?apikey=%APIKEY\",\r\n\t\"historyUrl\": \"https://api.mediatemple.net/api/v1/stats/%SERVICEID/%RANGE.json?apikey=%APIKEY\",\r\n\t\"rangeUrl\": \"https://api.mediatemple.net/api/v1/stats/%SERVICEID.json?start=%START&end=%END&&resolution=%RESOLUTION&apikey=%APIKEY\"\r\n}\r\n```\r\n\r\nMore on stats resolution\r\n------------------------\r\n\r\nMediaTemple may or may not respect the stats resolution you request, e.g. there's a minimum resolution of 15s and a request for 2 hour intervals will be returned at a resolution of 60 minutes. \r\n\r\nThe API server will also only serve up a fairly small amount of data at a given resolution - smaller than I'd like, so the server divides up the client's range into several requests and combines them before returning. The interval at which to split a single request into multiple API queries is the `step` if this is less than the `range` for that range. So if you wanted a week's worth of data at 15 second intervals, you could add `{\"range\": 604800, \"resolution\": 15, \"step\": 3600 }` (but please don't as you'll hit the API server 168 times!). All numbers are seconds. \r\n\r\nThe metrics bit of the config file is currently set up to return up to a week's worth of data at once; it will return more, but at a resolution that would hammer the MediaTemple server rather, so please add some more ranges if you want to do that. \r\n\r\n\r\nDependencies\r\n------------\r\n\r\n- [Konphyg](https://github.com/pgte/konphyg)\r\n\r\n\r\nLegal fun\r\n---------\r\n\r\nCopyright &copy; 2012 Andrew Weeks http://meloncholy.com\r\n\r\n(mt) Stats is licensed under the [MIT licence](http://meloncholy.com/licence).\r\n\r\n\r\nMe\r\n--\r\n\r\nI have a [website](http://meloncholy.com) and a [Twitter](https://twitter.com/meloncholy). Please come and say hi if you'd like or if something's not working; be lovely to hear from you. \r\n"
"(mt) Stats
==========

(mt) Stats is a specialised proxy server that gets your server's statistics from [MediaTemple's API servers](http://mediatemple.net/api/). It was designed to be used with the [(mt) Stats Viewer](https://github.com/meloncholy/mt-stats-viewer) front end, but may even be more generally useful.

If you want to read more about how it works and some potential problems, there's a surprisingly (to me, at least) long post [here](http://meloncholy.com/blog/using-d3-for-realtime-webserver-stats).


Setting up (mt) Stats
---------------------

- [Get an API key](https://ac.mediatemple.net/api) for your MediaTemple server. You'll also need your service ID later, which you can get by visiting `https://api.mediatemple.net/api/v1/services/ids.json?apikey=XXXXX` (I only have one service ID as I have one server, but apparently you could see more.)

- Rename the config file `node_modules/mt-stats/config/mt-stats-sample.json` to `mt-stats.json` and change the service ID and API key to match your server and key.


Using (mt) Stats with your app
------------------------------

Here's a quick example of using (mt) Stats with Express.

```javascript
var app = require("express").createServer();
var mtStats = require("mt-stats");

app.get("/api/:range?", mtStats);

app.listen(3000);
```

(mt) Stats will respond differently depending on the range passed to it as part of the path.

- **No range** - Server stats for current time
- **Range is one of** `5min`, `15min`, `30min`, `1hour`, `1day`, `1week`, `1month`, `3month`, `1year` - Server stats for that past period will be returned, using the MediaTemple API server's default resolution for that period
- **Range is [0-9]+** - Server stats for the past _number_ of seconds will be returned, using the resolution in the settings file (probably; see below)
- **Range is [0-9]+-[0-9]+** - Server stats covering the range from the first number (time since Linux epoch in seconds) to the second number will be returned, using the resolution in the settings file


Settings
--------

Please rename `mt-stats-sample.json` to `mt-stats.json`.

- **serviceId** - Your server's service ID
- **apiKey** - Your MediaTemple API key
- **mode** - Doesn't do anything here, but its presence gives me a warm, comforting glow
- **rootPath** - The root URL path for all API calls
- **interval** - Server polling interval for client. MediaTemple's stats update every 15s
- **ranges** - For each `range`, the `resolution` at which to request data from the API (e.g. every 15 seconds) and the maximum span (`step`) to request in one go (to stop the API server objecting). Range is the maximum timespan at which to use that resolution and step
- **metrics** - Metrics supplied by the API. `apiKey` is the key name in JSON objects and `niceName` is the name to use on the graphs
- **definedRanges** - MediaTemple also supports some default intervals that can be requested with these URLs, e.g. [this URL](http://bits.meloncholy.com/mt-stats/api/5min) will serve up the last 5 minutes' data
- **currentUrl** - API server URL from which to get the current stats. `%SERVICEID` and `%APIKEY` are replaced with your service ID and API key
- **historyUrl** - API server URL to request stats going back for the past X seconds, e.g. [this URL](http://bits.meloncholy.com/mt-stats/api/300) will also give the past 5 minutes' data
- **rangeUrl** - API server URL to get stats covering a specified time range

```javascript
{
	"serviceId": 000000,
	"apiKey": "XXXXXXXX",
	"mode": "production",
	"rootPath": "/api/",
	"interval": 15000,
	"ranges": [
		{ "range": 3600, "resolution": 15, "step": 3600 },
		{ "range": 43200, "resolution": 120, "step": 28800 },
		{ "range": 86400, "resolution": 240, "step": 57600 },
		{ "range": 604800, "resolution": 1800, "step": 432000 }
	],
	"metrics": [
		{ "apiKey": "cpu", "niceName": "CPU %" },
		{ "apiKey": "memory", "niceName": "Memory %" },
		{ "apiKey": "load1Min", "niceName": "Load 1 min" },
		{ "apiKey": "load5Min", "niceName": "Load 5 min" },
		{ "apiKey": "load15Min", "niceName": "Load 15 min" },
		{ "apiKey": "processes", "niceName": "Processes" },
		{ "apiKey": "diskSpace", "niceName": "Disk space" },
		{ "apiKey": "kbytesIn", "niceName": "kb in / sec" },
		{ "apiKey": "kbytesOut", "niceName": "kb out / sec" },
		{ "apiKey": "packetsIn", "niceName": "Packets in / sec" },
		{ "apiKey": "packetsOut", "niceName": "Packets out / sec" }
	],
	"definedRanges": ["5min", "15min", "30min", "1hour", "1day", "1week", "1month", "3month", "1year"],
	"currentUrl": "https://api.mediatemple.net/api/v1/stats/%SERVICEID.json?apikey=%APIKEY",
	"historyUrl": "https://api.mediatemple.net/api/v1/stats/%SERVICEID/%RANGE.json?apikey=%APIKEY",
	"rangeUrl": "https://api.mediatemple.net/api/v1/stats/%SERVICEID.json?start=%START&end=%END&&resolution=%RESOLUTION&apikey=%APIKEY"
}
```

More on stats resolution
------------------------

MediaTemple may or may not respect the stats resolution you request, e.g. there's a minimum resolution of 15s and a request for 2 hour intervals will be returned at a resolution of 60 minutes.

The API server will also only serve up a fairly small amount of data at a given resolution - smaller than I'd like, so the server divides up the client's range into several requests and combines them before returning. The interval at which to split a single request into multiple API queries is the `step` if this is less than the `range` for that range. So if you wanted a week's worth of data at 15 second intervals, you could add `{"range": 604800, "resolution": 15, "step": 3600 }` (but please don't as you'll hit the API server 168 times!). All numbers are seconds.

The metrics bit of the config file is currently set up to return up to a week's worth of data at once; it will return more, but at a resolution that would hammer the MediaTemple server rather, so please add some more ranges if you want to do that.


Dependencies
------------

- [Konphyg](https://github.com/pgte/konphyg)


Legal fun
---------

Copyright &copy; 2012 Andrew Weeks http://meloncholy.com

(mt) Stats is licensed under the [MIT licence](http://meloncholy.com/licence).


Me
--

I have a [website](http://meloncholy.com) and a [Twitter](https://twitter.com/meloncholy). Please come and say hi if you'd like or if something's not working; be lovely to hear from you. 