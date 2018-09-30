---
title: Java根据圆心经纬度、半径获取圆的点集
date: 2018-09-29 16:41:54
tags:
    - Java
---
![homePage](/upload/homePage/20180930150201.jpg)
<!--more-->

## 情景
最近有对接第三方的需求，需要在后台通过圆心点经纬度、半径获取圆的经纬度点集。

## 根据圆心、半径计算圆上点集的算法
首先来分析一下，这个问题可以转换成数学上的平面坐标系计算，已知圆心坐标(x0,y0)，半径r，角度a0，求圆上任一点公式为：

> x1 = x0 + r * cos(a0 * π/180)
> y1 = y0 + r * sin(a0 * π/180)

其中a0 * π/180(角度乘以π/180)，目的是将角度a0转换为弧度。

下面将平面上的圆等分为6份(如下图)，根据公式代入圆心坐标、弧度和半径计算出对应圆弧半径与圆交点的平面坐标。

![获取圆经纬度点集_1](/upload/获取圆经纬度点集/获取圆经纬度点集_1.jpg)

这样就获得了6个平面圆上的坐标点，(x1,y1)、(x2,y2)...(x6,y6)，注意此时的点只是数学意义上的点，接下来需要将平面的点投影到地图上，获取地图经纬度的点集。

## 将平面坐标投影到地图

在介绍如何将平面坐标投影到地图之前，我们首先需要了解一个概念:墨卡托投影。

众所周知，地球是一个不规则的球体，而我们平时看到的地图大多是平面的，那么如何将一个三维的地球画到平面上的呢。

墨卡托投影就是将三维的地球表示在一个二维平面上的方法之一，也是应用得最广泛的方法，我们平时看到的谷歌地图，百度地图，都是使用的墨卡托投影。

墨卡托投影展开方式见下图:

![获取圆经纬度点集_墨卡托投影](/upload/获取圆经纬度点集/获取圆经纬度点集_墨卡托投影_2.gif)

> 什么是墨卡托投影？
> 墨卡托投影（Mercator Projection），又名“等角正轴圆柱投影”，荷兰地图学家墨卡托（Mercator）在1569年拟定，假设地球被围在一个中空的圆柱里，其赤道与圆柱相接触，然后再假想地球中心有一盏灯，把球面上的图形投影到圆柱体上，再把圆柱体展开，这就是一幅标准纬线为零度（即赤道）的“墨卡托投影”绘制出的世界地图。

那么我们如何将已经计算出的平面圆坐标点转换成地图的经纬度点呢？具体实现原理这里就不列举了，有兴趣了解的同学可以查看[经纬度坐标映射到平面直角坐标系](https://blog.csdn.net/ywjatjd/article/details/62896201#%E4%B8%89%E5%9D%90%E6%A0%87%E7%B3%BB%E6%8A%95%E5%BD%B1%E7%9A%84%E5%AE%9E%E7%8E%B0source-code)，该地址上有详细的解释，其中有以C#实现的将XY坐标转经纬度的代码，本文将该部分以Java实现应用到点集计算中，在此感谢大神们的分享。

## 解决方案(代码)

这里通过上述的平面数学公式、XY坐标转经纬度，结合后实现的根据圆心经纬度、半径获取圆点集的最终实现代码如下：

```
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

import com.alibaba.fastjson.JSON;

/**
 * Gis工具类
 * @author hefan
 * @date 2018/9/27 8:55
 */
public class GisUtils {

    /**
     * 反向转换程序中的迭代次数
     */
    private final int iterativeTimes = 10;
    /**
     * 反向转换程序中的迭代初始值
     */
    private final double iterativeValue = 0;
    /**
     * 椭球体长轴,千米
     */
    private double _A = 6378.137;
    /**
     * 椭球体短轴,千米
     */
    private double _B = 6356.752314;
    /**
     * 标准纬度,弧度
     */
    private double _B0 = 0;
    /**
     * 原点经度,弧度
     */
    private double _L0 = 0;
    /**
     * 默认点集点数量
     */
    private static final int _defaultNumPoints = 40;
    /**
     * 地球半径
     */
    //public static final double _R = 6371000.79;

    private GisUtils() {
    }

    /**
     * 获取实例
     * @author hefan
     * @date 2018/9/27 10:05
     */
    public static GisUtils getInstance(){
        return new GisUtils();
    }

    /**
     * 角度到弧度的转换
     * @author hefan
     * @date 2018/9/27 9:07
     */
    public double degreeToRad(double degree) {
        return Math.PI * degree / 180;
    }

    /**
     * 弧度到角度的转换
     * @author hefan
     * @date 2018/9/27 9:07
     */
    public double radToDegree(double rad) {
        return (180 * rad) / Math.PI;
    }

    /**
     * 设定_A与_B
     * @author hefan
     * @date 2018/9/27 9:07
     */
    public void setAB(double a, double b) {
        if (a <= 0 || b <= 0) {
            return;
        }
        _A = a;
        _B = b;
    }

    /**
     * 设定_B0
     * @author hefan
     * @date 2018/9/27 9:27
     */
    public void setLB0(double pmtL0, double pmtB0) {
        double l0 = degreeToRad(pmtL0);
        if (l0 < -Math.PI || l0 > Math.PI) {
            return;
        }
        _L0 = l0;

        double b0 = degreeToRad(pmtB0);
        if (b0 < -Math.PI / 2 || b0 > Math.PI / 2) {
            return;
        }
        _B0 = b0;
    }

    /**
     * 经纬度转XY坐标
     * pmtLB0: 参考点经纬度
     * pmtLB1: 要转换的经纬度
     * 返回值: 直角坐标，单位：公里
     * @author hefan
     * @date 2018/9/27 9:04
     */
    public Point lBToXY(Point pmtLB0, Point pmtLB1) {
        setLB0(pmtLB0.getLng(), pmtLB0.getLat());

        double B = degreeToRad(pmtLB1.getLat());
        double L = degreeToRad(pmtLB1.getLng());

        Point xy = new Point(0,0);
        //xy.setLat(0);
        //xy.setLng(0);

        double f/*扁率*/, e/*第一偏心率*/, e_/*第二偏心率*/, NB0/*卯酉圈曲率半径*/, K, dtemp;
        double E = Math.exp(1);
        if (L < -Math.PI || L > Math.PI || B < -Math.PI / 2 || B > Math.PI / 2) {
            return xy;
        }
        if (_A <= 0 || _B <= 0) {
            return xy;
        }
        f = (_A - _B) / _A;
        dtemp = 1 - (_B / _A) * (_B / _A);
        if (dtemp < 0) {
            return xy;
        }
        e = Math.sqrt(dtemp);
        dtemp = (_A / _B) * (_A / _B) - 1;
        if (dtemp < 0) {
            return xy;
        }
        e_ = Math.sqrt(dtemp);

        NB0 = ((_A * _A) / _B) / Math.sqrt(1 + e_ * e_ * Math.cos(_B0) * Math.cos(_B0));
        K = NB0 * Math.cos(_B0);
        xy.setLng(K * (L - _L0));
        xy.setLat(K * Math.log(Math.tan(Math.PI / 4 + (B) / 2) * Math.pow((1 - e * Math.sin(B)) / (1 + e * Math.sin(B)), e / 2)));
        double y0 = K * Math.log(Math.tan(Math.PI / 4 + (_B0) / 2) * Math.pow((1 - e * Math.sin(_B0)) / (1 + e * Math.sin(_B0)), e / 2));
        xy.setLat(xy.getLat() - y0);

        //正常的Y坐标系（向上）转程序的Y坐标系（向下）
        xy.setLat(-xy.getLat());

        return xy;
    }

    /**
     * XY坐标转经纬度
     * pmtLB0: 参考点经纬度
     * pmtXY: 要转换的XY坐标，单位：公里
     * 返回值: 经纬度
     * @author hefan
     * @date 2018/9/27 9:04
     */
    public Point xYtoLB(Point pmtLB0, Point pmtXY) {
        setLB0(pmtLB0.getLng(), pmtLB0.getLat());

        double X = pmtXY.getLng();
        double Y = -pmtXY.getLat();//程序的Y坐标系（向下）转正常的Y坐标系（向上）

        double B = 0, L = 0;

        Point lb = new Point(0,0);
        //lb.setLat(0);
        //lb.setLng(0);

        double f/*扁率*/, e/*第一偏心率*/, e_/*第二偏心率*/, NB0/*卯酉圈曲率半径*/, K, dtemp;
        double E = Math.exp(1);

        if (_A <= 0 || _B <= 0) {
            return lb;
        }
        f = (_A - _B) / _A;
        dtemp = 1 - (_B / _A) * (_B / _A);
        if (dtemp < 0) {
            return lb;
        }
        e = Math.sqrt(dtemp);
        dtemp = (_A / _B) * (_A / _B) - 1;
        if (dtemp < 0) {
            return lb;
        }
        e_ = Math.sqrt(dtemp);
        NB0 = ((_A * _A) / _B) / Math.sqrt(1 + e_ * e_ * Math.cos(_B0) * Math.cos(_B0));
        K = NB0 * Math.cos(_B0);

        double y0 = K * Math.log(Math.tan(Math.PI / 4 + (_B0) / 2) * Math.pow((1 - e * Math.sin(_B0)) / (1 + e * Math.sin(_B0)), e / 2));
        Y = Y + y0;

        L = X / K + _L0;
        B = iterativeValue;

        for (int i = 0; i < iterativeTimes; i++) {
            B = Math.PI / 2 - 2 * Math.atan(Math.pow(E, (-Y / K)) * Math.pow(E, (e / 2) * Math.log((1 - e * Math.sin(B)) / (1 + e * Math.sin(B)))));
        }

        lb.setLng(radToDegree(L));
        lb.setLat(radToDegree(B));

        return lb;
    }


    /**
     * 获取圆形区域点集（返回区域点集数量默认40个）
     *
     * @author hefan
     * @date 2018/9/27 9:22
     * @param centerPointLat    中心点纬度
     * @param centerPointLng    中心点经度
     * @param radius            圆形区域半径（单位：千米）
     * @return
     */
    public List<double[]> getCircularAreaPoints(double centerPointLng,double centerPointLat,
                                                       double radius){
        return getCircularAreaPoints(new Point(centerPointLng, centerPointLat),
                radius, _defaultNumPoints);
    }

    /**
     * 获取圆形区域点集
     *
     * @author hefan
     * @date 2018/9/27 9:22
     * @param centerPointLng    中心点经度
     * @param centerPointLat    中心点纬度
     * @param radius            圆形区域半径（单位：千米）
     * @param numPoints         返回圆区域点数量
     * @return
     */
    public List<double[]> getCircularAreaPoints(double centerPointLng,double centerPointLat,
                                                       double radius, int numPoints){
        return getCircularAreaPoints(new Point(centerPointLng,centerPointLat),
                radius, numPoints);
    }

    /**
     * 获取圆形区域点集
     *
     * @author hefan
     * @date 2018/9/27 9:13
     * @param centerPoint   圆心点（平面坐标转经纬度参考点）
     * @param radius        圆形区域半径（单位：千米）
     * @param numPoints     返回圆区域点数量
     * @return
     */
    public List<double[]> getCircularAreaPoints(Point centerPoint, double radius, int numPoints) {
        List<double[]> points = new ArrayList<>();
        double phase = 2 * Math.PI / numPoints;
        Point tmpPoint = null;
        for (int i = 0; i < numPoints; i++) {
            /**
             * 计算坐标点
             */
            double dx = (radius * Math.cos(i * phase));
            double dy = (radius * Math.sin(i * phase));

            /**
             * 转换成经纬度
             */
            tmpPoint = xYtoLB(new Point(centerPoint.getLat(),centerPoint.getLng()),
                    new Point(dx, dy));
            points.add(new double[]{roundDown6(tmpPoint.getLng()),
                    roundDown6(tmpPoint.getLat())});
        }
        return points;
    }

    /**
     * 获取圆形区域点集（地图点集）
     *
     * @author hefan
     * @date 2018/9/27 9:13
     * @param centerPoint   圆心点（平面坐标转经纬度参考点）
     * @param radius        圆形区域半径（单位：千米）
     * @param numPoints     返回圆区域点数量
     * @return
     */
    public List<Point> getCircularAreaMapPoints(Point centerPoint, double radius, int numPoints) {
        List<Point> points = new ArrayList<>();
        double phase = 2 * Math.PI / numPoints;
        Point tmpPoint = null;
        for (int i = 0; i < numPoints; i++) {
            /**
             * 计算坐标点
             */
            double dx = (radius * Math.cos(i * phase));
            double dy = (radius * Math.sin(i * phase));

            /**
             * 转换成经纬度
             */
            tmpPoint = xYtoLB(new Point(centerPoint.getLat(),centerPoint.getLng()),
                    new Point(dx, dy));
            points.add(new Point(roundDown6(tmpPoint.getLng()),
                    roundDown6(tmpPoint.getLat())));
        }
        return points;
    }

    /**
     * 获取圆形区域点集（返回区域点集数量默认40个）（地图点集）
     *
     * @author hefan
     * @date 2018/9/27 9:22
     * @param centerPointLat    中心点纬度
     * @param centerPointLng    中心点经度
     * @param radius            圆形区域半径（单位：千米）
     * @return
     */
    public String getCircularAreaMapPoints(double centerPointLng,double centerPointLat,
                                                double radius){
        List<Point> points = getCircularAreaMapPoints(new Point(centerPointLng, centerPointLat),
                radius, _defaultNumPoints);
        return JSON.toJSONString(points);
    }

    /**
     * 保留小数点后6位，不进位
     * @author hefan
     * @date 2018/9/27 10:14
     */
    public double roundDown6(double value) {
        return Double.valueOf(new BigDecimal(value).setScale(6,
                BigDecimal.ROUND_DOWN).doubleValue());
    }

    /**
     * 点
     * @author hefan
     * @date 2018/9/27 9:26
     */
     class Point {
        /**
         * 经度
         */
        private double lng;
        /**
         * 纬度
         */
        private double lat;

        public double getLng() {
            return lng;
        }

        public void setLng(double lng) {
            this.lng = lng;
        }

        public double getLat() {
            return lat;
        }

        public void setLat(double lat) {
            this.lat = lat;
        }

        public Point(double lng, double lat) {
            this.lng = lng;
            this.lat = lat;
        }
    }

    public static void main(String[] args) {
        long startTime = System.currentTimeMillis();//获取开始时间
        String tmp = GisUtils.getInstance().getCircularAreaMapPoints(36.120869,
                120.420729, 1);
        System.out.println(tmp);
        long endTime = System.currentTimeMillis();    //获取结束时间
        System.out.println("程序运行时间：" + (endTime - startTime) + "ms");    //输出程序运行时间
    }

}

```

测试程序执行效率，在生成圆形区域点集点数量默认40的情况下，运行时间小于10ms。

## 总结与反思
除了CUID，你还会些什么？’(°ー°〃)
知其然，更要知其所以然。~(￣0￣)/