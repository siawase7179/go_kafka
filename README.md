# Kafak Producer/Consumer
```go

require github.com/gin-gonic/gin v1.9.1

require (
	github.com/bytedance/sonic v1.9.1 // indirect
	github.com/chenzhuoyu/base64x v0.0.0-20221115062448-fe3a3abad311 // indirect
	github.com/confluentinc/confluent-kafka-go/v2 v2.3.0 // indirect
	github.com/gabriel-vasile/mimetype v1.4.2 // indirect
	github.com/gin-contrib/sse v0.1.0 // indirect
	github.com/go-playground/locales v0.14.1 // indirect
	github.com/go-playground/universal-translator v0.18.1 // indirect
	github.com/go-playground/validator/v10 v10.14.0 // indirect
	github.com/goccy/go-json v0.10.2 // indirect
	github.com/json-iterator/go v1.1.12 // indirect
	github.com/klauspost/cpuid/v2 v2.2.4 // indirect
	github.com/leodido/go-urn v1.2.4 // indirect
	github.com/mattn/go-isatty v0.0.19 // indirect
	github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd // indirect
	github.com/modern-go/reflect2 v1.0.2 // indirect
	github.com/pelletier/go-toml/v2 v2.0.8 // indirect
	github.com/twitchyliquid64/golang-asm v0.15.1 // indirect
	github.com/ugorji/go/codec v1.2.11 // indirect
	golang.org/x/arch v0.3.0 // indirect
	golang.org/x/crypto v0.9.0 // indirect
	golang.org/x/net v0.10.0 // indirect
	golang.org/x/sys v0.8.0 // indirect
	golang.org/x/text v0.9.0 // indirect
	google.golang.org/protobuf v1.30.0 // indirect
	gopkg.in/yaml.v3 v3.0.1 // indirect
)

```
[github.com/confluentinc/confluent-kafka-go/v2](https://github.com/confluentinc/confluent-kafka-go/tree/master) 를 사용하였다.

-----
## Producer

POST api/ 

{{value}}

를 통해 Publish를 한다.

```go
r := gin.Default()
r.NoRoute(func(c *gin.Context) {
  c.JSON(404, gin.H{
    "status":  404,
    "code":    "90001",
    "message": "No handler found for " + c.Request.Method + " " + c.Request.RequestURI,
  })
})

r.POST("/", func(c *gin.Context) {
  value, err := io.ReadAll(c.Request.Body)

  if err == nil {
    err := p.Produce(&kafka.Message{
      TopicPartition: kafka.TopicPartition{Topic: &topic, Partition: kafka.PartitionAny},
      Value:          value,
      Headers:        []kafka.Header{{Key: "myTestHeader", Value: []byte("header values are binary")}},
    }, nil)

    if err != nil {
      if err.(kafka.Error).Code() == kafka.ErrQueueFull {
        time.Sleep(time.Second)
      }
      fmt.Printf("Failed to produce message: %v\n", err)
    }

  } else {
    fmt.Printf(err.Error())
  }
})
```

## Consumer
Consumer 예시
```go
for run {
  select {
  case sig := <-sigchan:
    fmt.Printf("Caught signal %v: terminating\n", sig)
    run = false
  default:
    ev := c.Poll(100)
    if ev == nil {
      continue
    }

    switch e := ev.(type) {
    case *kafka.Message:
      fmt.Printf("%% Message on %s:\n%s\n",
        e.TopicPartition, string(e.Value))
      if e.Headers != nil {
        fmt.Printf("%% Headers: %v\n", e.Headers)
      }

    case kafka.Error:
      fmt.Fprintf(os.Stderr, "%% Error: %v: %v\n", e.Code(), e)
      if e.Code() == kafka.ErrAllBrokersDown {
        run = false
      }
    default:

    }
  }
}
```
