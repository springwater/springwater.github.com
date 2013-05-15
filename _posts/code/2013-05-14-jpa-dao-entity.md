---
layout: post
title: jpa dao"
tagline: ""
description: ""
category : code
tags: [java, jpa, entity]
---
{% include JB/setup %}

###id转换name
{% highlight java %}
public String getGridJson(String con, Collection<?> paras, String start, String limit, String sortdir,
		HashMap<String, HashMap<String, String>> abmap, String cloud) {
	try {
		ObjectMapper objectMapper = new ObjectMapper();
		HashMap<String, Object> map = new HashMap<String, Object>();
		if (StringUtils.isNotBlank(start) && StringUtils.isNotBlank(limit)) {
			int first = Integer.parseInt(start);
			int max = Integer.parseInt(limit);
			if (first >= 0 && max > 0) {
				map.put("total", queryTotal(con, paras, cloud));
			}
		}
		List<M> list = query(con, paras, start, limit, sortdir, cloud);
		List<HashMap<String, Object>> data = new ArrayList<>();
		if (abmap != null && abmap.size() > 0) {
			HashMap<String, Object> e;
			HashMap<String, String> ab;
			String fname, key, id, value;
			for (M m : list) {
				e = new HashMap<>();
				Field[] declaredFields = getEntityClass().getDeclaredFields();
				for (Field field : declaredFields) {
					fname = field.getName();
					if (!"serialVersionUID".equals(fname)) {
						field.setAccessible(true);
						e.put(fname, field.get(m));
					}
				}
				for (Entry<String, HashMap<String, String>> entry : abmap.entrySet()) {
					key = entry.getKey();
					ab = entry.getValue();
					id = e.get(key).toString();
					value = ab.get(id);
					if (value != null) {
						e.put(key, ab.get(id));
					}
				}
				data.add(e);
			}
		}
		map.put("data", data);
		return objectMapper.writeValueAsString(map);
	} catch (Exception e) {
		e.printStackTrace();
	}
	return "{}";
}
{% endhighlight %}