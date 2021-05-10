---
layout: post
title: "Android 判断当前是否在模拟器中运行"
date: 2014-07-13 09:28
author: ray
comments: true
categories: [Blog]
tags: [Android]
---
<pre>//是否是模拟器。如果返回TRUE，则当前是模拟器，不是返回FALSE
    public static boolean isEmulator(Context context){
        try{
            TelephonyManager tm = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
            String imei = tm.getDeviceId();
            if (imei != null &amp;&amp; imei.equals("000000000000000")){
                return true;
            }
            return  (Build.MODEL.equals("sdk")) || (Build.MODEL.equals("google_sdk"));
        }catch (Exception ioe) { 

        }
        return false;
    }</pre>
