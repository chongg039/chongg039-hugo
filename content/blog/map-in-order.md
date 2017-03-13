+++
date = "2017-03-13T18:51:20+08:00"
draft = false
title = "解决 Go 中遍历 map 的随机化问题"
Tags = ["Go"]
+++

在 air-server 中有一个函数，支持解析 URL 的 query 并返回字段中城市的数据，如：

```
GET /aqi/cities?1=成都&2=北京&3=杭州&4=西安
```

客户端希望的返回值应该是按照输入顺序，即 1，2，3，4 返回的，但实际多次测试后得到的结果总会是不一样的顺序。想了想应该是 query 解析的结果，即`r.URL.Query()`为 map 的原因。

查询后得知，Go 对 map 的实现是在底层的 hashmap，因此对 key-value 的插入和对 map 的遍历而言两者的 key 访问顺序不同。实践后也证明在使用 range 遍历 map 时得到的 key 的顺序被随机化了，Go 
官方也提到过不希望开发者依赖 range 遍历的 key 次序。

```
When iterating over a map with a range loop, the iteration order is not specified and is not guaranteed to be the same from one iteration to the next. Since Go 1 the runtime randomizes map iteration order, as programmers relied on the stable iteration order of the previous implementation.
```

但我们的业务逻辑往往不可避免的使用顺序的 key-value，怎么解决这个问题呢。
有作者给出了一个方法，即单独维护一个有序的 key 表，根据对应的 key 遍历原始的 map。下面是

```go
func ReturnDataOfCities(w http.ResponseWriter, r *http.Request) {
	q := r.URL.Query()

	var locations []string

	// 由于遍历 map 时 key 的随机化问题，维护一个有序的 keys 数组保证
	// 每次的顺序都是固定的
	sorted_keys := make([]int, 0)
	for k, _ := range q {
	   // 我们希望的 key 为自然数，并按大小排序
		i, err := strconv.Atoi(k)
		if err != nil {
			panic(err)
		}
		sorted_keys = append(sorted_keys, i)
	}

	// Sort 'int' key in decreasing order
	sort.Ints(sorted_keys)

	// If you want key in increasing order
	// for i, j := 0, len(sorted_keys)-1; i < j; i, j = i+1, j-1 {
	// 	sorted_keys[i], sorted_keys[j] = sorted_keys[j], sorted_keys[i]
	// }

	for _, v := range sorted_keys {
	   // 将 'int' 类型的 key 转换成 'string'
		s := strconv.Itoa(v)
		locations = append(locations, q[s][0])
	}
	
	result := CompareDataOfCities(locations)

	w.Header().Set("Content-Type", "application/json; charset=UTF-8")
	w.WriteHeader(http.StatusOK)
	if err := json.NewEncoder(w).Encode(result); err != nil {
		panic(err)
	}
}
```

至此达到了我们的要求。

参考文章：
[Go-maps-in-action](https://blog.golang.org/go-maps-in-action)

[遍历map时的key随机化问题及解决方法](http://blog.csdn.net/slvher/article/details/44779081)


