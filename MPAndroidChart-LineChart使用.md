---
title: MPAndroidChart LineChart使用
date: 2021-05-03 14:30:24
tags:  Android
categories: Android
---

MPAndroidChart是开源的一个Android图标库，效果酷炫，内部包括了LineChart(折线图),BarChart(饼状图),CombinedChart(组合图),PieChart(饼状图),ScatterChart(散点图),BubbleChart(气泡图),Stacked BarChart(堆积条形图)，CandleStickChart（蜡烛图）,CubicLineChart(立方折线图),RadarChart(雷达图),RealtimeChart(实时折线图)，SinusBarChart(正弦柱状图)等等。

<!--more-->

## 集成

- 添加到项目的`build.gradle`

```groovy
allprojects {
	repositories {
		maven { url "https://jitpack.io" }
	}
}
```

- 添加到模块的`build.gradle`

```groovy
dependencies {
	implementation 'com.github.PhilJay:MPAndroidChart:v3.0.3'
}
```



## 使用

- 在布局文件xml中添加

```xml
 <com.github.mikephil.charting.charts.LineChart
          android:id="@+id/wh_detail_chart"
          android:layout_width="match_parent"
          android:layout_height="match_parent"
          android:layout_below="@+id/wh_detail_title"
          android:layout_marginBottom="@dimen/distance_8"
          />
```

- 配置

```java
 /**
   * 配置图表显示样式
   * @param size x轴label个数
   * @param wareHouseInfos 图表数据
   */
  public void chart(int size, List<WareHouseInfo> wareHouseInfos) {
    //设置des
    mBinding.whDetailChart.getDescription().setEnabled(false);
    // 绘制网格线
    mBinding.whDetailChart.setDrawGridBackground(false);
    // 绘制边框
    mBinding.whDetailChart.setDrawBorders(false);
    // 设置legend,后面描述
    mBinding.whDetailChart.getLegend().setEnabled(false);
    // 当没有图标数据的时候显示
    mBinding.whDetailChart.setNoDataText("库区储备明细图");
    // 设置x轴样式
    XAxis xAxis = mBinding.whDetailChart.getXAxis();
    xAxis.setPosition(XAxis.XAxisPosition.BOTTOM);
    xAxis.setDrawGridLines(false);
    xAxis.setDrawAxisLine(false);
    xAxis.setAxisMinimum(1f);
    xAxis.setLabelCount(size, true);
    xAxis.setAxisMaximum(size);
    xAxis.setAxisLineColor(R.color.colorAccent);
    List<String> value = Lists.newArrayList();
    for (int i = 0; i <= size; i++) {
      value.add(i + "仓");
    }
    //设置x轴label
    xAxis.setValueFormatter(new YAxisValueFormatter(value));
    // 设置y轴样式
    YAxis leftAxis = mBinding.whDetailChart.getAxisLeft();
    leftAxis.setLabelCount(10, true);
    leftAxis.setAxisMinimum(0f);
    YAxis rightAxis = mBinding.whDetailChart.getAxisRight();
    rightAxis.setEnabled(false);
    for (int i = 0; i < 100; i++) {
      if (i % 5 == 0) {
        mChartData = generateDataLine(wareHouseInfos);
        mBinding.whDetailChart.setData(mChartData);
      }
    }
    // 以动画的方式绘制
    mBinding.whDetailChart.animateX(750);
  }
```

- 绘制线条

```java
 /**
   * 绘制线条
   *
   * @param wareHouseInfos 图表数据
   */
  private LineData generateDataLine(Integer count, List<WareHouseInfo> wareHouseInfos) {
    // lineChart Entry列表
    List<Entry> e1 = Lists.newArrayList();
    for (int i = 0; i < count; i++) {
      // 将数据填充到每个y值
      e1.add(new Entry(i, wareHouseInfos.get(i).getAuthorizedCapacity() / 1000));
    }
    LineDataSet d1 = new LineDataSet(e1, "存储明细");
    // 绘制线宽
    d1.setLineWidth(2.5f);
    // 绘制entry圆形
    d1.setCircleRadius(4.5f);
    // 配置是否在Entry上画值
    d1.setDrawValues(true);
    // 设置线的颜色
    d1.setColors(new int[] {
        R.color.colorAccent, R.color.colorAccent, R.color.colorAccent, R.color.colorAccent
    }, getContext());
    d1.setCircleColors(new int[] {
        R.color.colorAccent, R.color.colorAccent, R.color.colorAccent, R.color.colorAccent
    }, getContext());
    // 设置line模式
    d1.setMode(LineDataSet.Mode.HORIZONTAL_BEZIER);
    ArrayList<ILineDataSet> sets = new ArrayList<>();
    sets.add(d1);
    return new LineData(sets);
  }
```

## 显示样式

![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fpvup4bgv6j20e60crwf6.jpg)

打完收工，下次写`BarChart`。