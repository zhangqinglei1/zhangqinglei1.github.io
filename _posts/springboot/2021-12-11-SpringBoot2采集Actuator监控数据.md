---

layout: post
title:  "SpringBoot2采集Actuator监控数据"
date:   2021-12-11 20:14:54
categories: SpringBoot
tags: SpringBoot
excerpt: SpringBoot2采集Actuator监控数据
mathjax: true

---

* content
{:toc}
采集代码，自己写的，可完善

## 采集Metrics

### bean对象MetricsInfoBean

```

import java.util.Date;

public class MetricsInfoBean {

	private String memoryMax;
	
	private String memoryUsed;
	
	private String cpuUsage;
	
	/**
	 * 系统运行时间，单位分钟，保留二位小数
	 */
	private long uptime;
	
	private Date startTime;

	public Date getStartTime() {
		return startTime;
	}

	public void setStartTime(Date startTime) {
		this.startTime = startTime;
	}

	public String getMemoryMax() {
		return memoryMax;
	}

	public void setMemoryMax(String memoryMax) {
		this.memoryMax = memoryMax;
	}

	public String getMemoryUsed() {
		return memoryUsed;
	}

	public void setMemoryUsed(String memoryUsed) {
		this.memoryUsed = memoryUsed;
	}

	public String getCpuUsage() {
		return cpuUsage;
	}

	public void setCpuUsage(String cpuUsage) {
		this.cpuUsage = cpuUsage;
	}
	
	public long getUptime() {
		return uptime;
	}

	public void setUptime(long uptime) {
		this.uptime = uptime;
	}

	@Override
	public String toString() {
		return "MetricsInfoBean [memoryMax=" + memoryMax + ", memoryUsed=" + memoryUsed + ", cpuUsage=" + cpuUsage
				+ ", uptime=" + uptime + ", startTime=" + startTime + "]";
	}
	
}
```

### 工具处理类

```
public class MetricsUtil {

	/**
	 * 从B到GB，保留二位小数
	 * @param value
	 * @return
	 */
	public static String toTransGBformB(Double value) {
		BigDecimal bd = new BigDecimal(value);
		BigDecimal bdSub = new BigDecimal(1024*1024*1024);
		String memoryMax = bd.divide(bdSub,2,1).toPlainString();
		return memoryMax;
	}
	
	public static String toTransCpu(Double value) {
		BigDecimal bd = new BigDecimal(value);
		return bd.setScale(4,RoundingMode.HALF_EVEN).toPlainString();
	}
}
```

### 采集代码

```
import java.math.BigDecimal;
import java.util.Date;
import java.util.List;
import java.util.Map;

import org.apache.commons.collections.CollectionUtils;
import org.springframework.web.client.RestTemplate;

import com.epoch.actuator.bean.MetricsInfoBean;
import com.epoch.actuator.util.MetricsUtil;

public class MetricsService {
	
	private static String url = "http://192.168.56.101:6869/actuator";

	public MetricsInfoBean findMetricsInfo() {
		RestTemplate rest = new RestTemplate();
		String baseUrl = url+"/metrics";
		String[] names = {"jvm.memory.max","jvm.memory.used","process.cpu.usage","process.uptime","process.start.time"};
		MetricsInfoBean bean = new MetricsInfoBean();
		for (String name : names) {
			Map<String,Object> maps = rest.getForObject(baseUrl+"/"+name, Map.class);
			List<Map<String,Object>> measurements = (List<Map<String, Object>>) maps.get("measurements");
			Double value = (Double) measurements.get(0).get("value");
			if(CollectionUtils.isNotEmpty(measurements)) {
				switch(name){
					case "jvm.memory.max":
						bean.setMemoryMax(MetricsUtil.toTransGBformB(value));
					case "jvm.memory.used":
						bean.setMemoryUsed(MetricsUtil.toTransGBformB(value));
					case "process.cpu.usage":
						bean.setCpuUsage(MetricsUtil.toTransCpu(value));
					case "process.uptime":
						bean.setUptime(new BigDecimal(value).longValue());
					case "process.start.time":
						bean.setStartTime(new Date(new BigDecimal(value).longValue()));
				}
			}
			
		}
		return bean;
	}

}
```

