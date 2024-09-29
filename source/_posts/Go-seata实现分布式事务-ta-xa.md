---
title: Go+seata实现分布式事务-ta-xa
date: 2024-09-29 16:05:40
tags: [分布式事务,Go]
categories: [分布式事务,Go]
---
### docker安装seata
```
version: '3'
services:
  seata-server:
    image: seataio/seata-server:latest
    ports:
      - "8091:8091"
      - "7091:7091"
    environment:
      - SEATA_PORT=8091
      - STORE_MODE=file

  mysql:
    image: mysql:8.0.32
    container_name: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=12345678
    command: --default-authentication-plugin=mysql_native_password --default-time-zone='+08:00'
    volumes:
      - ./mysql:/docker-entrypoint-initdb.d
      - ./mysql/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf
    ports:
      - "3306:3306"
```
### mysql数据库文件和mysql配置(可选不一定非要docker，只需要大于8.0就行)
```
CREATE database if NOT EXISTS seata_client default character set utf8mb4 collate utf8mb4_unicode_ci;
USE seata_client;

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

CREATE TABLE IF NOT EXISTS  order_tbl (
  id int(11) NOT NULL AUTO_INCREMENT,
  user_id varchar(255) DEFAULT NULL,
  commodity_code varchar(255) DEFAULT NULL,
  count int(11) DEFAULT '0',
  money int(11) DEFAULT '0',
  descs varchar(255) DEFAULT '',
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT INTO seata_client.order_tbl (id, user_id, commodity_code, count, money, descs) VALUES (1, 'NO-100001', 'C100000', 100, 10, 'init desc');

DROP TABLE IF EXISTS undo_log;

CREATE TABLE undo_log (
                            id bigint NOT NULL AUTO_INCREMENT,
                            branch_id bigint NOT NULL,
                            xid varchar(100) NOT NULL,
                            context varchar(128) NOT NULL,
                            rollback_info longblob NOT NULL,
                            log_status int NOT NULL,
                            log_created datetime NOT NULL,
                            log_modified datetime NOT NULL,
                            ext varchar(100) DEFAULT NULL,
                            PRIMARY KEY (id),
                            KEY idx_unionkey (xid,branch_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

```
### 本次主要测试at跟xa模式，at模式跟xa模式差距不大,at是seata特有的模式，需要本地一个undo_log记录数据
#### 连接seata的配置文件
```
# time 时间单位对应的是 time.Duration(1)
seata:
  enabled: true
  # application id
  application-id: applicationName
  # service group
  tx-service-group: default_tx_group
  access-key: aliyunAccessKey
  secret-key: aliyunSecretKey
  enable-auto-data-source-proxy: true
  data-source-proxy-mode: AT
  client:
    rm:
      # Maximum cache length of asynchronous queue
      async-commit-buffer-limit: 10000
      # The maximum number of retries when report reports the status
      report-retry-count: 5
      # The interval for regularly checking the metadata of the db（AT）
      table-meta-check-enable: false
      # Whether to report the status if the transaction is successfully executed（AT）
      report-success-enable: false
      # Whether to allow regular check of db metadata（AT）
      saga-branch-register-enable: false
      saga-json-parser: fastjson
      saga-retry-persist-mode-update: false
      saga-compensate-persist-mode-update: false
      #Ordered.HIGHEST_PRECEDENCE + 1000  #
      tcc-action-interceptor-order: -2147482648
      # Parse SQL parser selection
      sql-parser-type: druid
      lock:
        retry-interval: 30
        retry-times: 10
        retry-policy-branch-rollback-on-conflict: true
    tm:
      commit-retry-count: 5
      rollback-retry-count: 5
      default-global-transaction-timeout: 60s
      degrade-check: false
      degrade-check-period: 2000
      degrade-check-allow-times: 10s
      interceptor-order: -2147482648
    undo:
      # Judge whether the before image and after image are the same，If it is the same, undo will not be recorded
      data-validation: false
      # Serialization method
      log-serialization: json
      # undo log table name
      log-table: undo_log
      # Only store modified fields
      only-care-update-columns: true
      compress:
        # Compression type. Allowed Options: None, Gzip, Zip, Sevenz, Bzip2, Lz4, Zstd, Deflate
        type: None
        #  Compression threshold Unit: k
        threshold: 64k
    load-balance:
      type: RandomLoadBalance
      virtual-nodes: 10
  service:
    vgroup-mapping:
      # Prefix for Print Log
      default_tx_group: default
    grouplist:
      default: 127.0.0.1:8091
    enable-degrade: false
    # close the transaction
    disable-global-transaction: false
  transport:
    shutdown:
      wait: 3s
    # Netty related configurations
    # type
    type: TCP
    server: NIO
    heartbeat: true
    # Encoding and decoding mode
    serialization: seata
    # Message compression mode
    compressor: none
    # Allow batch sending of requests (TM)
    enable-tm-client-batch-send-request: false
    # Allow batch sending of requests (RM)
    enable-rm-client-batch-send-request: true
    # RM send request timeout
    rpc-rm-request-timeout: 30s
    # TM send request timeout
    rpc-tm-request-timeout: 30s
  # Configuration Center
  config:
    type: file
    file:
      name: config.conf
    nacos:
      namespace: ""
      server-addr: 127.0.0.1:8848
      group: SEATA_GROUP
      username: ""
      password: ""
      ##if use MSE Nacos with auth, mutex with username/password attribute
      #access-key: ""
      #secret-key: ""
      data-id: seata.properties
  # Registration Center
  registry:
    type: file
    file:
      name: registry.conf
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      group: "SEATA_GROUP"
      namespace: ""
      username: ""
      password: ""
      ##if use MSE Nacos with auth, mutex with username/password attribute  #
      #access-key: ""  #
      #secret-key: ""  #
  log:
    exception-rate: 100
  tcc:
    fence:
      # Anti suspension table name
      log-table-name: tcc_fence_log_test
      clean-period: 60s
  # getty configuration
  getty:
    reconnect-interval: 0
    # temporary not supported connection-num
    connection-num: 1
    session:
      compress-encoding: false
      tcp-no-delay: true
      tcp-keep-alive: true
      keep-alive-period: 120s
      tcp-r-buf-size: 262144
      tcp-w-buf-size: 65536
      tcp-read-timeout: 1s
      tcp-write-timeout: 5s
      wait-timeout: 1s
      max-msg-len: 16498688
      session-name: client_test
      cron-period: 1s
```
### utils.go
```
package util

import (
	"database/sql"
	"os"

	sql2 "github.com/seata/seata-go/pkg/datasource/sql"
)

func GetAtMySqlDb() *sql.DB {
	dsn := "root:root@tcp(192.168.252.1:3306)/seata_client?multiStatements=true&interpolateParams=true"
	dbAt, err := sql.Open(sql2.SeataATMySQLDriver, dsn)
	if err != nil {
		panic("init seata at mysql driver error")
	}
	return dbAt
}

func GetXAMySqlDb() *sql.DB {
	dsn := "root:root@tcp(192.168.252.1:3306)/seata_client?multiStatements=true&interpolateParams=true"
	dbAt, err := sql.Open(sql2.SeataXAMySQLDriver, dsn)
	if err != nil {
		panic("init seata at mysql driver error")
	}
	return dbAt
}
```
### 客户端，模拟用户请求
```
package main

import (
	"context"
	"flag"
	"fmt"
	"github.com/parnurzeal/gorequest"
	"github.com/seata/seata-go/pkg/constant"
	"github.com/seata/seata-go/pkg/tm"
	"github.com/seata/seata-go/pkg/util/log"
	"net/http"
	"time"

	"github.com/seata/seata-go/pkg/client"
)

var serverIpPort = "http://127.0.0.1:8080"

func main() {
	flag.Parse()
	client.InitPath("../../../conf/seatago.yml")

	bgCtx, cancel := context.WithTimeout(context.Background(), time.Minute*10)
	defer cancel()
	sampleUpdate(bgCtx)

}
func updateData(ctx context.Context) (re error) {
	request := gorequest.New()
	log.Infof("branch transaction begin")
	request.Post(serverIpPort+"/updateDataSuccess").
		Set(constant.XidKey, tm.GetXID(ctx)).
		End(func(response gorequest.Response, body string, errs []error) {
			if response.StatusCode != http.StatusOK {
				re = fmt.Errorf("update data fail")
			}
		})
	return
}

func sampleUpdate(ctx context.Context) {
	if err := tm.WithGlobalTx(ctx, &tm.GtxConfig{
		Name:    "ATSampleLocalGlobalTx_Update",
		Timeout: time.Second * 30,
	}, updateData); err != nil {
		log.Info(err)
	}
}
```
### 服务端，模拟api网关，往不同的微服务发请求
```
package main

import (
	"context"
	"database/sql"
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/seata/seata-go-samples/util"
	"github.com/seata/seata-go/pkg/client"
	ginmiddleware "github.com/seata/seata-go/pkg/integration/gin"
	"github.com/seata/seata-go/pkg/util/log"
	"net/http"
)

var db *sql.DB

func main() {
	client.InitPath("../../../conf/seatago.yml")
    //TODO 这里是不同模式
    // db = util.GetXAMySqlDb() //xa
	db = util.GetAtMySqlDb()   //at
	r := gin.Default()
	r.Use(ginmiddleware.TransactionMiddleware())
	r.POST("/updateDataSuccess", updateDataSuccessHandler)
	if err := r.Run(":8080"); err != nil {
		log.Fatalf("start tcc server fatal: %v", err)
	}
}

func updateDataSuccessHandler(c *gin.Context) {
	log.Infof("get tm updateData")
	if err := updateDataSuccess(c); err != nil {
		c.JSON(http.StatusBadRequest, "updateData failure")
		return
	}
	if err := updateDataSuccess2(c); err != nil {
		c.JSON(http.StatusBadRequest, "updateData2 failure")
		return
	}
	//c.JSON(http.StatusOK, "updateData ok") //成功
	c.JSON(http.StatusBadRequest, "updateData failure") //TODO 测试fail,回滚
}

func updateDataSuccess(ctx context.Context) error {
	sql := "update order_tbl set count=? where id=?"
	ret, err := db.ExecContext(ctx, sql, 10, 1)
	if err != nil {
		fmt.Printf("update failed, err:%v\n", err)
		return nil
	}

	rows, err := ret.RowsAffected()
	if err != nil {
		fmt.Printf("update failed, err:%v\n", err)
		return nil
	}
	fmt.Printf("更新成功 success： %d.\n", rows)
	return nil
}

func updateDataSuccess2(ctx context.Context) error {
	sql := "update order_tbl set count=? where id=?"
	ret, err := db.ExecContext(ctx, sql, 101, 1)
	if err != nil {
		fmt.Printf("update failed, err:%v\n", err)
		return nil
	}

	rows, err := ret.RowsAffected()
	if err != nil {
		fmt.Printf("update failed, err:%v\n", err)
		return nil
	}
	fmt.Printf("更新成功 success： %d.\n", rows)
	return nil
}
```
### 把原生sql连接改成gorm连接
```
package main

import (
	"context"
	"database/sql"
	"time"

	"github.com/seata/seata-go/pkg/client"
	sql2 "github.com/seata/seata-go/pkg/datasource/sql"
	"github.com/seata/seata-go/pkg/tm"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

type OrderTblModel struct {
	Id            int64  `gorm:"column:id" json:"id"`
	UserId        string `gorm:"column:user_id" json:"user_id"`
	CommodityCode string `gorm:"commodity_code" json:"commodity_code"`
	Count         int64  `gorm:"count" json:"count"`
	Money         int64  `gorm:"money" json:"money"`
	Descs         string `gorm:"descs" json:"descs"`
}

func main() {
	initConfig()
	// insert
	tm.WithGlobalTx(context.Background(), &tm.GtxConfig{
		Name:    "ATSampleLocalGlobalTx",
		Timeout: time.Second * 30,
	}, insertData)
	<-make(chan struct{})
}

func initConfig() {
	// init seata client config
	client.InitPath("../../conf/seatago.yml")
	// init db object
	initDB()
}

var gormDB *gorm.DB

func initDB() {
	sqlDB, err := sql.Open(sql2.SeataATMySQLDriver, "root:12345678@tcp(127.0.0.1:3306)/seata_client?multiStatements=true&interpolateParams=true")
	if err != nil {
		panic("init service error")
	}

	gormDB, err = gorm.Open(mysql.New(mysql.Config{
		Conn: sqlDB,
	}), &gorm.Config{})
}

// insertData insert one data
func insertData(ctx context.Context) error {
	data := OrderTblModel{
		UserId:        "NO-100003",
		CommodityCode: "C100001",
		Count:         101,
		Money:         11,
		Descs:         "insert desc",
	}

	return gormDB.WithContext(ctx).Table("order_tbl").Create(&data).Error
}

// deleteData delete one data
func deleteData(ctx context.Context) error {
	return gormDB.WithContext(ctx).Where("id = ?", "1").Delete(&OrderTblModel{}).Error
}

// updateDate update one data
func updateData(ctx context.Context) error {
	return gormDB.WithContext(ctx).Model(&OrderTblModel{}).Where("id = ?", "1").Update("commodity_code", "C100002").Error
}

```
### 最后，如果报错 first phase error: undo log parser type jackson not found 那么请修改
#### 这句是写死的，没有走配置
![](1.png)

