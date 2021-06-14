# HA-NR-MYCovidStats
Home Assistant &amp; Node Red Implementation of Malaysia Covid Stats

Original implementation from Jimmy93 (FB:A Jim) [repo](https://github.com/jimmy93/Malaysia-Daily-Covid-19-Home-Assistant) which implement Malaysia COVID-19 Statistic for use in Home Assistant[HA] with HA configuration.yaml triggering command line callof python script. 

I reimplement similar functions using Node-Red to callout and retrieve statistic [repo](https://github.com/wnarifin/covid-19-malaysia) which you should read out in length how the data scaraping is done *sigh - malaysia statistic avialbility* 

In addition to sending back for HA for lovelace; also included in node red function for Telegram request and reporting.

## How It Works (Extension to Node-Red)
1.  Node-Red pulls CSV from [@wnarifin github](https://github.com/wnarifin/covid-19-malaysia) and returns payload as CSV array.
2.  In Node-Red Function, Array pop() on function node returns last row; which would correspond to latest data and what likely we want to see anyway.
3.  Send to HA entities defined as Sensors. Defined 4 most useful information while the rest information is nested as attributes under the sensor. 
4.  Telegram node /getcovidstats sets flags to identify at output to only send Telegram update when requested. 

## Pre-requisite Setup
A. Working copy 
1.  Home Assistant with Node-Red. Ada banyak tutorial/videos on this with difficulty level as easy. [This is one example](http://https://www.juanmtech.com/get-started-with-node-red-and-home-assistant/). Test that you have enabled and can load Node-red on side bar.
2.  Telegram bot and chat ids. I followed this [tutorial](https://www.thesmarthomebook.com/2020/10/13/a-guide-to-using-telegram-with-node-red-and-home-assistant/) which is clear and easy to follow. Required from here is your botid and chatid. 
3.  
4.  copy code from [configuration.yaml](configuration.yaml) to our HA.
