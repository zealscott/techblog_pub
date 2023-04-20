+++
title = "公交可视化系统"
slug = "bus visualization"
tags = ["nodejs"]
date = "2020-05-30T21:31:03+08:00"
description = "使用 JavaScript 组建上海市公交系统可视化系统"

+++

项目展示地址：[zealscott.com/soda](https://zealscott.com/soda)

## 数据来源

* 上海市一卡通数据
  * 来源：上海市 2018 SODA 比赛
  * 每条线的公交车线路为一个 CSV 文件，每天更新
* 上海市公交车站点信息
  * 来源：百度地图
  * 从百度地图爬取上海市所有的公交车站点经纬度，并使用坐标转换工具转换为标准的 WJS 84 坐标

## 爬虫

* 我们需要上海市所有的公交车站点 GPS 信息，用于展现公交车的行驶路径
* 百度地图没有提供批量的 API 进行查询，因此我们先要伪装一个百度地图容器，然后主动调用公交站站点 API，并进行循环，写入到本地磁盘中

```js
function InitMap() {

            map = new BMap.Map("container");

            var point = new BMap.Point(121.56, 31.25);//上海的经纬度点

            map.centerAndZoom(point, 12);

            map.enableScrollWheelZoom();


            var busLineSearchOptions = {

                renderOptions: {

                    map: map,

                    panel: document.getElementById("result"),

                    autoViewport: true,

                },

                onGetBusListComplete: function (rs) {

                    if(rs.getNumBusList() == 2){

                        // alert("onGetBusListComplete");

                        console.log("本次检索的关键字：" + rs.keyword);

                        console.log("本次检索所在的城市：" + rs.city);

                        console.log("百度地图的url：" + rs.moreResultsUrl);

                        // for (var i = 0; i < rs.getNumBusList(); i++) {

                            // 上行
                            // var busListItem = rs.getBusListItem(0);
                            // 下行
                            var busListItem = rs.getBusListItem(1);

                            console.log(busListItem);

                            // console.log(busLineSearch.getBusLine(busListItem).name);
                            // if(busLineSearch.getBusLine(busListItem).name == busLineName)
                                busLineSearch.getBusLine(busListItem);

                            // if(rs.getNumBusList() == 2 && i == 0)
                            //     console.log("上行");
                            //
                            // if(rs.getNumBusList() == 2 && i == 1)
                            //     console.log("下行");
                        // }
                    }
                },

                onGetBusLineComplete: function (busLine) {

                    //alert("onGetBusLineComplete");

                    // console.log("线路名称:" + busLine.name);

                    // console.log("首班车时间:" + busLine.startTime);
                    //
                    // console.log("末班车时间:" + busLine.endTime);
                    //
                    // console.log("公交线路所属公司:" + busLine.company);

                    // console.log("本公交线路的站点个数:" + busLine.getNumBusStations());

                    // fs.appendFile(fileName, "本公交线路的站点个数:" + busLine.getNumBusStations());


                    for (var i = 0; i < busLine.getNumBusStations(); i++) {

                        var busStation = busLine.getBusStation(i);

                        // console.log("第" + (i + 1) + "个站点名称:" + busStation.name);

                        // console.log("第" + (i + 1) + "个站点的坐标:" + busStation.position);

                        console.log(busLine.name + "," + busStation.name + "," + busStation.position);

                        // document.write("'name:'" + busStation.name + "," + busStation.position);
                        // f1.writeLine("'name:'" + busStation.name + "," + busStation.position);
                    }

                },

            };

            var busLineSearch = new BMap.BusLineSearch(map, busLineSearchOptions);

            // $.getScript('StationLinesJS.js', function (){

            var Names = ["01路","04路"];

                var busLineName;
                for(i=0; i<Names.length; i++){
                    busLineName = Names[i];
                    // alert(Names[i]);
                    busLineSearch.getBusList(busLineName);
                }
            // })

        }

```

## 可视化

* 本次实验主要用到了以下几个可视化工具
* [Echart](https://echarts.apache.org/en/index.html)
  * 在百度地图底图的基础上，使用 Echart 绘制出每辆公交车的行驶轨迹，将行驶轨迹固定在道路路网上，使用不同的颜色进行区分，并控制行驶速度

![](/images/misc/bus1.png)

* [D3](https://d3js.org/)
  * 使用 D3 绘制公交车每个站点分时段的折线图，并随着鼠标点击的站点进行变化
    * ![](/images/misc/bus2.png)
  * 使用 D3 绘制公交车所有站点的分时段流量图，可以选择时间进行展示和对比
    * ![](/images/misc/bus3.png)

* Wordcloud
  * 根据站点的流量绘制了词云，点击每个站点名字，可以对应到地图上的坐标点
    * ![](/images/misc/bus4.png)







