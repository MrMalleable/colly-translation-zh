## Redis 后端

```go
package main

import (
	"log"

	"github.com/gocolly/colly"
	"github.com/gocolly/colly/queue"
	"github.com/gocolly/redisstorage"
)

func main() {
	urls := []string{
		"http://httpbin.org/",
		"http://httpbin.org/ip",
		"http://httpbin.org/cookies/set?a=b&c=d",
		"http://httpbin.org/cookies",
	}

	c := colly.NewCollector()

	// 使用Redis来存储收集器在运行过程中需要记录的数据
	storage := &redisstorage.Storage{
		Address:  "127.0.0.1:6379",
		Password: "",
		DB:       0,
		Prefix:   "httpbin_test",
	}

	// 为收集器设置后端存储
	err := c.SetStorage(storage)
	if err != nil {
		panic(err)
	}

	// 删除之前的数据
	if err := storage.Clear(); err != nil {
		log.Fatal(err)
	}

	// 使用defer函数在本函数执行结束之后关闭客户端
	defer storage.Client.Close()

	// 创建一个新的请求队列，数据使用Redis存储
	q, _ := queue.New(2, storage)

	c.OnResponse(func(r *colly.Response) {
		log.Println("Cookies:", c.Cookies(r.Request.URL.String()))
	})

	// 将链接加入到队列
	for _, u := range urls {
		q.AddURL(u)
	}
	// 消费需要访问的链接
	q.Run(c)
}
```

