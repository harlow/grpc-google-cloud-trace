# gRPC interceptor for Stackdriver Trace (Google Cloud) 

Pass google trace context in remote procedure calls. This allows parent-child tracing across multiple services.

## Client

Use the `interceptor.Client` to add google trace context to outgoing RPC calls
made by the gRPC client.

```go
import "github.com/harlow/grpc-google-cloud-trace/interceptor"

func main() {
	conn, err := grpc.Dial(
		address, 
		grpc.WithInsecure(),
		grpc.WithUnaryInterceptor(interceptor.Client(traceClient)),
	)
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	
	// ...
}
```

## Server

Use the `interceptor.Server` to parse the google cloud context from the request
metadata. The interceptor will set up a new child span of the requesting party.

```go
import "github.com/harlow/grpc-google-cloud-trace/interceptor"

func main() {
	// ...
	
	grpcServer := grpc.NewServer(
	  grpc.UnaryInterceptor(interceptor.Server(traceClient)),
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