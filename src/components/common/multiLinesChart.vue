<template>
    <div class="boarder" >
        <div class="chart" :id="id" ></div>
    </div>
</template>

<script>
    import HighStock from 'highcharts/highstock'
    HighStock.setOptions({global:{useUTC: false}});

    export default {
        name: "areasplineChart",
        props: {'tag':String,'title':String,'lineCount':Number,'linesName':Array[String],'linesShortName':Array[String],
             'linesColor':Array[String],'yShow':Boolean,'unitFormatter':Function},
        data() {
            let formatter = this.unitFormatter || function (x) { return x}
            return {
                id: this.tag || 'lineChart',
                cpuChart:null,
                option: {
                    chart: {
                        type: 'areaspline',
                    },
                    credits: {
                        enabled: false
                    },
                    title: {//指定图表标题
                        text: this.title || null,
                        align: 'left',
                        y:25,
                        margin: -20,
                        style: {
                            color: '#999999',
                            fontSize: '15px',
                        }
                    },
                    // subtitle:{
                    //     align: 'left',
                    //     y:40,
                    //     style: {
                    //         color: '#AAAAAA',
                    //         fontSize: '25px',
                    //         fontWeight: 'bold'
                    //     }
                    // },
                    // colors:['#333333',  '#409EFF',   '#67C23A',   '#E6A23C'],
                    colors: this.linesColor || ['rgba(64,158,255, 1)','rgba(245,108,108, 0.8)', 'rgba(103,194,58, 0.8)', 'rgba(230,162,60, 0.8)', 'rgba(51, 51, 51, 0.4)'],
                    xAxis: {
                        type: 'datetime',
                        dateTimeLabelFormats: {
                            // millisecond: '%H:%M:%S.%L',
                            // second: '%H:%M:%S',
                            second: '%M:%S',
                            // minute: '%H:%M',
                            // hour: '%H:%M',
                            // day: '%Y-%m-%d',
                            // month: '%Y-%m',
                            // year: '%Y'
                        },
                        title: {
                            text: '',
                            margin: -120,
                        },
                        //x坐标轴的刻度值
                        // categories: [], //与datetime格式互斥
                        labels: {//坐标轴上的刻度值（显示间隔、样式、单位）
                            style: {
                                color: '#999999',
                                fontSize: '0px'
                            },
                            align: 'center'
                        },
                        lineColor: '#dfdfdf',//坐标轴的颜色
                        tickColor: '#dfdfdf',//坐标轴上的刻度线的颜色
                        tickLength: 0,//坐标轴上刻度线的长度
                        gridLineWidth: 1,//网格线宽度。x轴默认为0，y轴默认为1px。
                        gridLineColor: '#f2f2f2',//网格线颜色。默认为：#C0C0C0。
                        gridLineDashStyle: 'Dash',//网格线线条样式。和Css border-style类似，常用的有：Solid、Dot、Dash
                        tickInterval: 30*1000,//刻度间隔
                        tickmarkPlacement: 'between',//刻度线对齐方式，有between和on可选，默认是between
                        crosshair: {//鼠标放上后显示纵轴的数据
                            color: '#999',
                            width: 1
                        }
                    },
                    yAxis: {//图表的纵坐标，多个轴[{轴一},{轴二}]
                        // gridLineWidth: 1,
                        // gridLineColor: '#f2f2f2',
                        tickPositions: [0, 25, 50, 75, 100],//y轴刻度值
                        // gridLineDashStyle: 'Dash',
                        // tickLength: 0,
                        title: {//纵坐标标题
                            text: ' ',
                            margin: 0,
                            style: {
                                color: '#999999',
                                fontSize: '10px'
                            }
                        },
                        labels: {//坐标轴上刻度的样式及单位
                            margin: -10,
                            reserveSpace:false,
                            style: {
                                color: '#CCCCCC',
                                fontSize: this.yShow ? '10px' : '0px',
                                // writingMode:'sideways-lr'
                            },
                            formatter: function () {
                                return formatter(this.value)
                            }

                        },
                        offset: -30,//距离坐标轴的距离
                    },
                    tooltip: {//数据提示框
                        headerFormat: '<small>{point.key}</small><br/>',//标题格式
                        pointFormatter: function () {
                            return '<span style="color:' + this.series.color + '">' + this.series.name +'</span>：' + formatter(this.y) + '<br/>'
                        },
                        xDateFormat: '%H:%M:%S',
                        shared: true,
                        followPointer: true,//跟随鼠标
                        followPointerMove: true,//是否跟随手指移动
                        // footerFormat: 'muzi',//尾部格式化字符串
                        style: {
                            fontSize: '12px',
                            fontFamily: '微软雅黑',
                            fontWeight: 'normal',
                            color: '#666666'
                        }
                    },
                    //标示线总是垂直于它属于的轴。它可单独定义在x轴或y轴，也可以同时定义在x轴和y轴
                    plotOptions: {
                        areaspline: {
                            marker: {
                                enabled: true,
                                symbol: 'circle',
                                radius: 2.5,
                                states: {
                                    hover: {
                                        enabled: true
                                    }
                                }
                            },
                            lineWidth: 1
                        }
                    },
                    legend: {//图例居中显示在图表下方
                        enabled:true,
                        floating:true,
                        align: 'right',
                        verticalAlign: 'top',
                        layout:'vertical',
                        rtl: true,
                        x:21,
                        y:-10,
                        symbolRadius: 4,//图标圆角
                        symbolWidth: 8,//图标宽度
                        symbolHeight: 8,//图标高度
                        itemStyle: {
                            color: '#999999',
                            fontWeight: 'normal',
                            fontSize: '10px'
                        },
                        itemMarginBottom: 0,//图例项底部外边距
                    },
                    series: this.markOptionSeries()
                },
            }
        },
        methods: {
            markDefaultPoint(count = 5){
                let defaultList = [];
                for(let index = 0;index<count;index++){
                    defaultList.push([(new Date()).getTime()+50*index, 0])
                }
                return defaultList
            },
            markOptionSeries(){
                let series = [];
                for(let index = 0;index<this.lineCount;index++){
                    let item = {
                        name: this.linesName[index] || 'line'+index.toString(),
                        data: this.markDefaultPoint(),
                        // tooltip: {
                        //     formatter: function () {
                        //         return this.formatter(this.point.y);
                        //     }
                        // },
                        fillOpacity: index==0 ? 0.5 : 0.1
                    }
                    series.push(item)
                }
                return series
            },
            updateLine(data){
                console.log(this.title, " update:", data)
                //let x = (new Date()).getTime()
                // console.log(x)
                // this.cpuChart.series[0].addPoint([ x, Math.round(Math.random() * 100)], false, true);

                data['data'].forEach((item,index,array) => {
                    this.cpuChart.series[index].addPoint([data['time'], item], false, true);
                    this.cpuChart.legend.allItems[index].legendItem.attr({text:this.linesShortName[index]+':'+ this.formatter(item)})
                })
                this.cpuChart.setTitle (null, {text: data['data'][0].toString()}, false)

                if (data.hasOwnProperty('data')){
                    if (data['data'].length > 0)
                    this.cpuChart.redraw();
                }

            }
        },
        mounted: function () {
            this.cpuChart = HighStock.chart(this.id, this["option"]);
        },
        components: {
            HighStock
        }
    }
</script>

<style scoped>
    .boarder {
        float: left;
        box-sizing:border-box;
        background-color: #fff;
        border-radius: 8px;
        width: 100%;
        height: 100%;
    }

    .chart {
        /*width: 400px;*/
        /*height: 200px*/
        border-radius: 8px;
        width: 100%;
        height: 100%;
    }
</style>