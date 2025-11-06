1.docker部署命令：

```
docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 14250:14250 \
  -p 9411:9411 \
  jaegertracing/all-in-one:latest
```

ui访问地址：localhost:16686

#### go配置

2.添加 OpenTelemetry 检测

```
go get "go.opentelemetry.io/otel" "go.opentelemetry.io/otel/exporters/stdout/stdoutmetric" "go.opentelemetry.io/otel/exporters/stdout/stdouttrace" "go.opentelemetry.io/otel/exporters/stdout/stdoutlog" "go.opentelemetry.io/otel/sdk/log" "go.opentelemetry.io/otel/log/global" "go.opentelemetry.io/otel/propagation" "go.opentelemetry.io/otel/sdk/metric" "go.opentelemetry.io/otel/sdk/resource" "go.opentelemetry.io/otel/sdk/trace" "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp" "go.opentelemetry.io/contrib/bridges/otelslog"
```

3.初始化jaeger和opentelemetry

```
package initialize

import (
	"fmt"
	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/exporters/jaeger"
	"go.opentelemetry.io/otel/propagation"
	"go.opentelemetry.io/otel/sdk/resource"
	"go.opentelemetry.io/otel/sdk/trace"
	sdktrace "go.opentelemetry.io/otel/sdk/trace"
	semconv "go.opentelemetry.io/otel/semconv/v1.21.0"
)

const (
	host        = "192.168.0.109"
	port        = 14268
	Service     = "goods_srv_http"
	environment = "production"
	id          = 1
)

func InitOTelSdk() (*sdktrace.TracerProvider, error) {
	exporter, err := jaeger.New(jaeger.WithCollectorEndpoint(jaeger.WithEndpoint(fmt.Sprintf("http://%s:%d/api/traces", host, port))))
	if err != nil {
		return nil, err
	}
	tp := sdktrace.NewTracerProvider(
		sdktrace.WithSampler(sdktrace.AlwaysSample()),
		sdktrace.WithBatcher(exporter),
		trace.WithResource(resource.NewWithAttributes(
			semconv.SchemaURL,
			semconv.ServiceName(Service),
			attribute.String("environment", environment),
			attribute.Int64("ID", id),
		)),
	)
	otel.SetTracerProvider(tp)
	otel.SetTextMapPropagator(propagation.NewCompositeTextMapPropagator(propagation.TraceContext{}, propagation.Baggage{}))
	return tp, nil
}


```

4.main.go加入以下代码

```
//初始化jaeger，添加链路追踪
	tp, otelErr := initialize.InitOTelSdk()
	if otelErr != nil {
		zap.S().Errorw("err:%w", otelErr)
	}
	defer func() {
		if otelErr := tp.Shutdown(context.Background()); otelErr != nil {
			zap.S().Infof("Error shutting down tracer provider: %v", otelErr)
		}
	}()
```

5.gin初始化

```
Use(otelgin.Middleware("goods-server"))
```

6.grpc初始化

```
	server := grpc.NewServer(
		//openTelemetry 链路追踪
		grpc.StatsHandler(otelgrpc.NewServerHandler(otelgrpc.WithTracerProvider(tp))),
	)
```

