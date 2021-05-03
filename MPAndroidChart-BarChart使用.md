---
title: MPAndroidChart BarChart使用
date: 2021-05-03 14:30:24
tags:  Android
categories: Android
---

![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fpyqkyt4sgj20z40jgwq5.jpg)
继上次的LineChart，本篇文章记录柱状图BarChart的使用，原理与LineChart没什么区别，都是定义左右YAxis和上下的XAxis，然后设置Labels，最后填充数据就ok。先看看效果图

<!--more-->

## 使用

- XML布局

```xml
<com.github.mikephil.charting.charts.BarChart
        android:id="@+id/chart1"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
       />
```

- 定义BarChart管理器

```java
/**
 * @author Administrator
 * @version 1.0
 * @description class description
 * @date 2018/1/2
 */

public class BarChartManager {
  private BarChart mBarChart;
  private YAxis leftAxis;
  private YAxis rightAxis;
  private XAxis xAxis;

  public BarChartManager(BarChart barChart) {
    this.mBarChart = barChart;
    leftAxis = mBarChart.getAxisLeft();
    rightAxis = mBarChart.getAxisRight();
    xAxis = mBarChart.getXAxis();
  }

  /**/
  private void initLineChart() {
    mBarChart.setDrawValueAboveBar(false);
    mBarChart.getDescription().setEnabled(false);
    //背景颜色
    //mBarChart.setBackground(SupportApp.drawable(R.drawable.shape_out_in));
    //网格
    mBarChart.setDrawGridBackground(false);
    //背景阴影
    mBarChart.setDrawBarShadow(false);
    mBarChart.setHighlightFullBarEnabled(false);
    //显示边界
    mBarChart.setDrawBorders(false);
    mBarChart.setPadding(1, 1, 1, 1);
    //设置动画效果
    mBarChart.animateY(1000, Easing.EasingOption.Linear);
    mBarChart.animateX(1000, Easing.EasingOption.Linear);
    mBarChart.setTextAlignment(View.TEXT_ALIGNMENT_CENTER);
    mBarChart.setOnTouchListener(null);
    //折线图例 标签 设置
    Legend legend = mBarChart.getLegend();
    legend.setForm(Legend.LegendForm.CIRCLE);
    legend.setTextSize(11f);
    legend.setTextColor(Color.WHITE);
    //显示位置
    legend.setVerticalAlignment(Legend.LegendVerticalAlignment.TOP);
    legend.setHorizontalAlignment(Legend.LegendHorizontalAlignment.CENTER);
    legend.setOrientation(Legend.LegendOrientation.HORIZONTAL);
    legend.setDrawInside(false);
    legend.setXEntrySpace(32);
    legend.setTextSize(12);
    //XY轴的设置
    //X轴设置显示位置在底部
    rightAxis.setEnabled(false);
    xAxis.setPosition(XAxis.XAxisPosition.BOTTOM);
    xAxis.setTextSize(10);
    xAxis.setTextColor(Color.WHITE);
    xAxis.setAxisMinimum(0f);
    leftAxis.setAxisMinimum(0f);
    leftAxis.setTextSize(12);
    leftAxis.setTextColor(Color.WHITE);
    xAxis.setGranularityEnabled(true);
    xAxis.setGranularity(0.9f);
    xAxis.setCenterAxisLabels(true);
    mBarChart.notifyDataSetChanged();
  }

  /**
   * 展示柱状图(一条)
   */
  public void showBarChart(List<Float> xAxisValues, List<Float> yAxisValues, String label,
      int color) {
    initLineChart();
    ArrayList<BarEntry> entries = new ArrayList<>();
    for (int i = 0; i < yAxisValues.size(); i++) {
      entries.add(new BarEntry(xAxisValues.get(i), yAxisValues.get(i)));
    }
    // 每一个BarDataSet代表一类柱状图
    BarDataSet barDataSet = new BarDataSet(entries, label);
    barDataSet.setColor(color);
    barDataSet.setValueTextSize(9f);
    barDataSet.setFormLineWidth(1f);
    barDataSet.setFormSize(15.f);
    barDataSet.setAxisDependency(YAxis.AxisDependency.LEFT);
    BarData data = new BarData(barDataSet);
    data.setDrawValues(false);
    data.setBarWidth(0.5f);
    data.setValueTextColor(SupportApp.color(R.color.white));
    //设置X轴的刻度数
    xAxis.setLabelCount(xAxisValues.size() - 1, false);
    mBarChart.setData(data);
    mBarChart.invalidate();
  }

  /**
   * 展示柱状图(多条)
   */
  public void showBarChart(List<Float> xAxisValues, List<List<Float>> yAxisValues,
      List<String> labels, List<Integer> colours) {
    initLineChart();
    BarData data = new BarData();
    for (int i = 0; i < yAxisValues.size(); i++) {
      ArrayList<BarEntry> entries = new ArrayList<>();
      for (int j = 0; j < yAxisValues.get(i).size(); j++) {

        entries.add(new BarEntry(xAxisValues.get(j), yAxisValues.get(i).get(j)));
      }
      BarDataSet barDataSet = new BarDataSet(entries, labels.get(i));
      barDataSet.setColor(colours.get(i));
      barDataSet.setValueTextColor(colours.get(i));
      barDataSet.setValueTextSize(10f);
      barDataSet.setAxisDependency(YAxis.AxisDependency.LEFT);
      barDataSet.setDrawValues(false);
      data.addDataSet(barDataSet);
    }
    int amount = yAxisValues.size();
    float groupSpace = 0.5f; //柱状图组之间的间距
    float barSpace = (float) ((1 - 0.12) / amount / 20); // x4 DataSet
    float barWidth = (float) ((1 - 0.12) / amount / 10 * 4); // x4 DataSet

    // (0.2 + 0.02) * 4 + 0.08 = 1.00 -> interval per "group"
    xAxis.setLabelCount(xAxisValues.size() - 1, false);
    List<String> values = Lists.newArrayList();
    values.add("11-01");
    values.add("11-02");
    values.add("11-03");
    values.add("11-04");
    values.add("11-05");
    values.add("11-06");
    values.add("11-07");
    xAxis.setPosition(XAxis.XAxisPosition.BOTTOM);
    xAxis.setValueFormatter(new YAxisValueFormatter(values));
    data.setBarWidth(barWidth);
    data.groupBars(0, groupSpace, barSpace);
    mBarChart.notifyDataSetChanged();
    mBarChart.setData(data);
    mBarChart.invalidate();
  }

  /**
   * 展示柱状图(多条)
   */
  public void showBarChart(List<Float> xAxisValues, List<List<Float>> yAxisValues,
      List<String> labels, List<Integer> colours, List<String> values) {
    initLineChart();
    BarData data = new BarData();
    for (int i = 0; i < yAxisValues.size(); i++) {
      ArrayList<BarEntry> entries = new ArrayList<>();
      for (int j = 0; j < yAxisValues.get(i).size(); j++) {
        entries.add(new BarEntry(xAxisValues.get(j), yAxisValues.get(i).get(j)));
      }
      BarDataSet barDataSet = new BarDataSet(entries, labels.get(i));
      barDataSet.setColor(colours.get(i));
      barDataSet.setValueTextColor(colours.get(i));
      barDataSet.setValueTextSize(10f);
      barDataSet.setAxisDependency(YAxis.AxisDependency.LEFT);
      barDataSet.setDrawValues(false);
      data.addDataSet(barDataSet);
    }
    int amount = yAxisValues.size();
    float groupSpace = 0.5f; //柱状图组之间的间距
    float barSpace = (float) ((1 - 0.12) / amount / 20); // x4 DataSet
    float barWidth = (float) ((1 - 0.12) / amount / 10 * 4); // x4 DataSet
    xAxis.setLabelCount(xAxisValues.size() - 1, false);
    xAxis.setPosition(XAxis.XAxisPosition.BOTTOM);
    xAxis.setValueFormatter(new YAxisValueFormatter(values));
    data.setBarWidth(barWidth);
    data.groupBars(0, groupSpace, barSpace);
    mBarChart.notifyDataSetChanged();
    mBarChart.setData(data);
  }

  /**
   * 设置Y轴值
   */
  public void setYAxis(float max, float min, int labelCount) {
    if (max < min) {
      return;
    }
    leftAxis.setAxisMaximum(max);
    leftAxis.setAxisMinimum(min);
    leftAxis.setLabelCount(labelCount, false);

    rightAxis.setAxisMaximum(max);
    rightAxis.setAxisMinimum(min);
    rightAxis.setLabelCount(labelCount, false);
    mBarChart.invalidate();
  }

  /**
   * 设置X轴的值
   */
  public void setXAxis(float max, float min, int labelCount) {
    xAxis.setAxisMaximum(max);
    xAxis.setAxisMinimum(min);
    xAxis.setLabelCount(labelCount, false);

    mBarChart.invalidate();
  }

  /**
   * 设置高限制线
   */
  public void setHightLimitLine(float high, String name, int color) {
    if (name == null) {
      name = "高限制线";
    }
    LimitLine hightLimit = new LimitLine(high, name);
    hightLimit.setLineWidth(4f);
    hightLimit.setTextSize(10f);
    hightLimit.setLineColor(color);
    hightLimit.setTextColor(color);
    leftAxis.addLimitLine(hightLimit);
    mBarChart.invalidate();
  }

  /**
   * 设置低限制线
   */
  public void setLowLimitLine(int low, String name) {
    if (name == null) {
      name = "低限制线";
    }
    LimitLine hightLimit = new LimitLine(low, name);
    hightLimit.setLineWidth(4f);
    hightLimit.setTextSize(10f);
    leftAxis.addLimitLine(hightLimit);
    mBarChart.invalidate();
  }

  /**
   * 设置描述信息
   */
  public void setDescription(String str) {
    Description description = new Description();
    description.setText(str);
    mBarChart.setDescription(description);
    mBarChart.invalidate();
  }

  /**
   * 展示柱状图(多条)
   */
  public void showBarChartPlan(List<Float> xAxisValues, List<List<Float>> yAxisValues,
      List<String> labels, List<Integer> colours, List<String> values) {
    initLineChart();
    BarData data = new BarData();
    for (int i = 0; i < yAxisValues.size(); i++) {
      ArrayList<BarEntry> entries = new ArrayList<>();
      for (int j = 0; j < yAxisValues.get(i).size(); j++) {
        entries.add(new BarEntry(xAxisValues.get(j), yAxisValues.get(i).get(j)));
      }
      BarDataSet barDataSet = new BarDataSet(entries, labels.get(i));
      barDataSet.setColor(colours.get(i));
      barDataSet.setValueTextColor(colours.get(i));
      barDataSet.setValueTextSize(10f);
      barDataSet.setAxisDependency(YAxis.AxisDependency.LEFT);
      barDataSet.setDrawValues(false);
      data.addDataSet(barDataSet);
    }
    int amount = yAxisValues.size();
    float groupSpace = 0.5f; //柱状图组之间的间距
    float barSpace = (float) ((1 - 0.12) / amount / 20); // x4 DataSet
    float barWidth = (float) ((1 - 0.12) / amount / 10 * 4); // x4 DataSet
    xAxis.setLabelCount(xAxisValues.size() - 1, false);
    xAxis.setPosition(XAxis.XAxisPosition.BOTTOM);
    xAxis.setValueFormatter(new YAxisValueFormatter(values));
    xAxis.setXOffset(barSpace);
    data.setBarWidth(barWidth);
    data.groupBars(0, groupSpace, barSpace);
    mBarChart.setDrawValueAboveBar(true);
    mBarChart.notifyDataSetChanged();
    mBarChart.setData(data);
  }
}
```

- 填充数据



```java
public void init(BarChart chart) {
    mManager = new BarChartManager(chart);
    for (int i = 0; i < 7; i++) {
      xValues.add((float) i);
    }
    //设置y轴的数据()
    List<List<Float>> yValues = new ArrayList<>();
    for (int i = 0; i < 4; i++) {
      List<Float> yValue = new ArrayList<>();
      for (int j = 0; j < 7; j++) {
        yValue.add((float) (Math.random() * 8000));
      }
      yValues.add(yValue);
    }
    //颜色集合
    colours.add(SupportApp.color(R.color.color_bar_one));
    colours.add(SupportApp.color(R.color.color_bar_three));
    colours.add(SupportApp.color(R.color.color_bar_four));
    colours.add(SupportApp.color(R.color.color_bar_two));
    //线的名字集合
    List<String> names = new ArrayList<>();
    names.add("米");
    names.add("面");
    names.add("粮");
    names.add("油");

    //创建多条折线的图表
    mManager.showBarChart(xValues, yValues, names, colours);
  }
```

今天的文章基本都是源码，有使用的MPAndroidChart BarChart的可以参考。