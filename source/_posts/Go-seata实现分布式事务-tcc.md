---
title: Go+seata实现分布式事务-tcc
date: 2024-09-29 16:04:28
tags: [分布式事务,Go]
categories: [分布式事务,Go]
---
### 上篇用的ta跟xa。这次试试tcc
#### client
```
package main

import (
	"context"
	"flag"
	"fmt"
	"net/http"
	"time"

	"github.com/parnurzeal/gorequest"

	"github.com/seata/seata-go/pkg/client"
	"github.com/seata/seata-go/pkg/constant"
	"github.com/seata/seata-go/pkg/tm"
	"github.com/seata/seata-go/pkg/util/log"
)

func main() {
	flag.Parse()
	client.InitPath("../../../conf/seatago.yml")
	bgCtx, cancel := context.WithTimeout(context.Background(), time.Minute*10)
	defer cancel()
	serverIpPort := "http://127.0.0.1:8080"
	var jsonData = map[string]interface{}{
		"name": "xxJohn",
		"age":  6000,
	}
	tm.WithGlobalTx(
		bgCtx,
		&tm.GtxConfig{
			Name: "TccSampleLocalGlobalTx",
		},
		func(ctx context.Context) (re error) {
			request := gorequest.New()
			log.Infof("branch transaction begin")
			request.Post(serverIpPort+"/prepare").Send(jsonData).
				Set("Content-Type", "application/json").
				Set(constant.XidKey, tm.GetXID(ctx)).
				End(func(response gorequest.Response, body string, errs []error) {
					if response.StatusCode != http.StatusOK {
						re = fmt.Errorf("update data fail")
					}
				})
			return
		})
}
```
### server
```
package main

import (
	"fmt"
	"net/http"

	"github.com/gin-gonic/gin"
    "github.com/seata/seata-go/pkg/tm"
	"github.com/seata/seata-go/pkg/client"
	ginmiddleware "github.com/seata/seata-go/pkg/integration/gin"
	"github.com/seata/seata-go/pkg/rm/tcc"
	"github.com/seata/seata-go/pkg/util/log"
)

func main() {
	client.InitPath("../../../conf/seatago.yml")

	r := gin.Default()

	r.Use(ginmiddleware.TransactionMiddleware())

	userProviderProxy, err := tcc.NewTCCServiceProxy(&RMService{})
	if err != nil {
		log.Errorf("get userProviderProxy tcc service proxy error, %v", err.Error())
		return
	}
	// 定义 JSON 数据的结构体
	type Person struct {
		Name string `json:"name"`
		Age  int    `json:"age"`
	}

	r.POST("/prepare", func(c *gin.Context) {
		var person Person
		// 绑定请求体到 person 结构体
		if err := c.BindJSON(&person); err != nil {
			fmt.Println("请求json错误", err)
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}
		c.Set("rollbackData", person)
		if _, err := userProviderProxy.Prepare(c, person); err != nil {
			c.JSON(http.StatusBadRequest, "prepare failure")
			return
		}
		c.JSON(http.StatusOK, "prepare ok")
		//c.JSON(http.StatusBadRequest, "prepare error") // 模拟错误
	})

	if err := r.Run(":8080"); err != nil {
		log.Fatalf("start tcc server fatal: %v", err)
	}
}

type RMService struct {
	Param interface{} //参数保存，防止rollback找不到
}

// 预提交事务，把参数保存起来
func (b *RMService) Prepare(ctx context.Context, params interface{}) (bool, error) {
	b.Param = params
	log.Infof("TRMService Prepare, param %v", params)
	return true, nil
}
// 提交
func (b *RMService) Commit(ctx context.Context, businessActionContext *tm.BusinessActionContext) (bool, error) {
	log.Infof("RMService Commit, param %v", businessActionContext)
	return true, nil
}
// 回滚
func (b *RMService) Rollback(ctx context.Context, businessActionContext *tm.BusinessActionContext) (bool, error) {
	log.Infof("RMService Rollback, param %v", b.Param)
	return true, nil
}

func (b *RMService) GetActionName() string {
	return "ginTccRMService"
}
```
