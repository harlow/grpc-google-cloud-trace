# Google Cloud Trace intercept for gRPC 

Pass google trace context in remote procedure calls. This allows parent-child tracing across multiple services.

## Client

Use the `intercept.ClientTrace` to add google trace context to outgoing RPC calls
made by the gRPC client.

```go
import(
	"cloud.google.com/go/trace"
	"github.com/harlow/grpc-google-cloud-trace/intercept"
)   

func main() {
	// ...
	
	traceClient, err := trace.NewClient(ctx, projectID)
	if err != nil {
		log.Fatalf("trace client error: %v", err)
	}
	
	conn, err := grpc.Dial(
		address, 
		grpc.WithInsecure(),
		grpc.WithUnaryInterceptor(intercept.ClientTrace(traceClient)),
	)
	if err != nil {
		log.Fatalf("grpc dial error: %v", err)
	}
	defer conn.Close()
	
	// ...
}
```

## Server

Use the `intercept.ServerTrace` to parse the google cloud context from the request
metadata. The interceptor will set up a new child span of the requesting party.

```go
import(
	"cloud.google.com/go/trace"
	"github.com/harlow/grpc-google-cloud-trace/intercept"
)   

func main() {
	// ...
	
	traceClient, err := trace.NewClient(ctx, projectID)
	if err != nil {
		log.Fatalf("trace client error: %v", err)
	}
	
	grpcServer := grpc.NewServer(
	  grpc.UnaryInterceptor(intercept.ServerTrace(traceClient)),
  	)
 	pb.RegisterRouteGuideServer(grpcServer, newServer())
	grpcServer.Serve(lis)
}
```

Create new child spans from the request context:

```go
func getNearbyPoints(ctx context.Context, lat, lon float64) []geo.Point {
	span := trace.FromContext(ctx).NewChild("getNearbyPoints")
	defer span.Finish()
	
	// ...
}
```

## Credits

This codebase was heavily inspired by the following issues and repositories:

* [Support client side interceptor](https://github.com/grpc/grpc-go/pull/867)
* [OpenTracing support for gRPC in Go](https://github.com/grpc-ecosystem/grpc-opentracing/tree/master/go/otgrpc)
