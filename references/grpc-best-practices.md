# gRPC Java Best Practices Reference

Detailed gRPC Java patterns and common issues with fixes.

## Performance Best Practices

### GRPC-001: Channel/Stub Reuse

```java
// ❌ CRITICAL: Creating new channel per request
public UserServiceClient() {
    this.channel = ManagedChannelBuilder.forAddress("localhost", 9090).build();
    this.stub = UserServiceGrpc.newBlockingStub(channel);
}

public User getUser(Long id) {
    // New channel created every time!
    ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 9090).build();
    return UserServiceGrpc.newBlockingStub(channel).getUser(id);
}

// ✅ CORRECT: Reuse channel and stub
public class UserServiceClient implements Closeable {
    private final ManagedChannel channel;
    private final UserServiceGrpc.UserServiceBlockingStub stub;
    
    public UserServiceClient() {
        this.channel = ManagedChannelBuilder.forAddress("localhost", 9090)
            .usePlaintext() // or useSsl() for production
            .keepAliveTime(30, TimeUnit.SECONDS)
            .build();
        this.stub = UserServiceGrpc.newBlockingStub(channel);
    }
    
    public User getUser(Long id) {
        return stub.withDeadlineAfter(5, TimeUnit.SECONDS).getUser(id);
    }
    
    @Override
    public void close() {
        channel.shutdown();
    }
}
```

### GRPC-010: Configuration-Based Addresses

```java
// ❌ WARNING: Hardcoded address
ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 9090).build();

// ✅ Better: From configuration
@Value("${grpc.server.address:localhost:9090}")
private String serverAddress;

@Bean
public ManagedChannel grpcChannel() {
    return ManagedChannelBuilder.forTarget(serverAddress)
        .usePlaintext()
        .build();
}
```

## Error Handling

### GRPC-004: Handling Status Codes

```java
// ❌ WARNING: Not handling all status codes
public User getUser(Long id) {
    try {
        return blockingStub.getUser(id);
    } catch (StatusRuntimeException e) {
        // Just rethrow - no specific handling
        throw e;
    }
}

// ✅ CORRECT: Handle specific status codes
public User getUser(Long id) {
    try {
        return stub.getUser(id);
    } catch (StatusRuntimeException e) {
        switch (e.getStatus().getCode()) {
            case NOT_FOUND:
                throw new UserNotFoundException("User not found: " + id);
            case DEADLINE_EXCEEDED:
                throw new ServiceTimeoutException("Service call timed out");
            case UNAVAILABLE:
                throw new ServiceUnavailableException("Service unavailable");
            case PERMISSION_DENIED:
                throw new AccessDeniedException("Access denied");
            default:
                throw new RuntimeException("gRPC error: " + e.getMessage());
        }
    }
}
```

### GRPC-005: Proper Error Response

```java
// ❌ WARNING: OK status with error details
public void getUser(GetUserRequest request, StreamObserver<User> responseObserver) {
    try {
        User user = findUser(request.getId());
        responseObserver.onNext(user);
        responseObserver.onCompleted();
    } catch (Exception e) {
        // Bad: Sending error in response body with OK status
        responseObserver.onNext(User.newBuilder()
            .setErrorMessage(e.getMessage())
            .build());
        responseObserver.onCompleted();
    }
}

// ✅ CORRECT: Use proper gRPC status
public void getUser(GetUserRequest request, StreamObserver<User> responseObserver) {
    try {
        User user = findUser(request.getId());
        responseObserver.onNext(user);
        responseObserver.onCompleted();
    } catch (UserNotFoundException e) {
        responseObserver.onError(Status.NOT_FOUND
            .withDescription(e.getMessage())
            .asRuntimeException());
    } catch (IllegalArgumentException e) {
        responseObserver.onError(Status.INVALID_ARGUMENT
            .withDescription(e.getMessage())
            .asRuntimeException());
    } catch (Exception e) {
        responseObserver.onError(Status.INTERNAL
            .withDescription("Internal error: " + e.getMessage())
            .asRuntimeException());
    }
}
```

### GRPC-012: Stream Error Handling

```java
// ❌ ERROR: Not handling stream errors
public StreamObserver<ChatMessage> chat(StreamObserver<ChatMessage> responseObserver) {
    return new StreamObserver<ChatMessage>() {
        @Override
        public void onNext(ChatMessage message) {
            responseObserver.onNext(processMessage(message));
        }
        
        @Override
        public void onCompleted() {
            responseObserver.onCompleted();
        }
        
        // Missing onError handling!
    };
}

// ✅ CORRECT: Handle all stream events
public StreamObserver<ChatMessage> chat(StreamObserver<ChatMessage> responseObserver) {
    return new StreamObserver<ChatMessage>() {
        @Override
        public void onNext(ChatMessage message) {
            try {
                responseObserver.onNext(processMessage(message));
            } catch (Exception e) {
                responseObserver.onError(Status.INTERNAL
                    .withDescription("Processing failed: " + e.getMessage())
                    .asRuntimeException());
            }
        }
        
        @Override
        public void onError(Throwable t) {
            // Handle client-side error
            log.warn("Stream error from client: " + t.getMessage());
        }
        
        @Override
        public void onCompleted() {
            responseObserver.onCompleted();
        }
    };
}
```

## Interceptors

### GRPC-006: Authentication Interceptor

```java
// Server-side auth interceptor
public class AuthInterceptor implements ServerInterceptor {
    @Override
    public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(
            ServerCall<ReqT, RespT> call, Metadata headers, 
            ServerCallHandler<ReqT, RespT> next) {
        
        String token = headers.get(Metadata.Key.of("authorization", Metadata.ASCII_STRING_MARSHALLER));
        
        if (token == null || !isValidToken(token)) {
            call.close(Status.UNAUTHENTICATED.withDescription("Missing or invalid token"),
                new Metadata());
            return new ServerCall.Listener<ReqT>() {};
        }
        
        return next.startCall(call, headers);
    }
}

// Client-side auth interceptor
public class AuthClientInterceptor implements ClientInterceptor {
    @Override
    public <ReqT, RespT> ClientCall<ReqT, RespT> interceptCall(
            MethodDescriptor<ReqT, RespT> method, CallOptions callOptions, Channel next) {
        
        return new ForwardingClientCall.SimpleForwardingClientCall<ReqT, RespT>(
            next.newCall(method, callOptions)) {
            
            @Override
            public void start(Listener<RespT> responseListener, Metadata headers) {
                Metadata.Key<String> key = Metadata.Key.of("authorization", Metadata.ASCII_STRING_MARSHALLER);
                headers.put(key, "Bearer " + getAuthToken());
                super.start(responseListener, headers);
            }
        };
    }
}
```

## Security

### GRPC-SEC-001: TLS Configuration

```java
// ❌ CRITICAL: No TLS
ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 9090)
    .usePlaintext()
    .build();

// ✅ CORRECT: TLS enabled
SslContext sslContext = GrpcSslContexts.forClient()
    .trustManager(new File("certificates/ca.crt"))
    .build();

ManagedChannel channel = NettyChannelBuilder.forAddress("localhost", 9090)
    .sslContext(sslContext)
    .build();

// Server-side TLS
Server server = ServerBuilder.forPort(9090)
    .sslContext(GrpcSslContexts.forServer(
        new File("server.crt"),
        new File("server.key"))
        .build())
    .addService(new UserServiceImpl())
    .build();
```

## Streaming

### GRPC-007: When to Use Streaming

```java
// ❌ WARNING: Unary for large data
public void downloadLargeFile(FileRequest request, StreamObserver<FileChunk> responseObserver) {
    // Sending millions of chunks via unary - inefficient!
    for (ByteBuffer chunk : getFileChunks(request.getFileId())) {
        responseObserver.onNext(FileChunk.newBuilder()
            .setData(ByteString.copyFrom(chunk))
            .build());
    }
    responseObserver.onCompleted();
}

// ✅ CORRECT: Use server streaming
public void downloadLargeFile(FileRequest request, StreamObserver<FileChunk> responseObserver) {
    // Streaming is more efficient for large data
    try (FileInputStream fis = new FileInputStream(request.getFileId())) {
        byte[] buffer = new byte[8192];
        int bytesRead;
        while ((bytesRead = fis.read(buffer)) != -1) {
            responseObserver.onNext(FileChunk.newBuilder()
                .setData(ByteString.copyFrom(buffer, 0, bytesRead))
                .build());
        }
    }
    responseObserver.onCompleted();
}
```

## Deadline & Timeout

### GRPC-009: Setting Deadlines

```java
// ❌ WARNING: No deadline
User user = stub.getUser(id);

// ✅ CORRECT: Always set deadline
User user = stub.withDeadlineAfter(5, TimeUnit.SECONDS)
    .getUser(id);

// ✅ With deadline from context
User user = stub.withDeadlineAfter(
    Context.current().getDeadline(), 
    TimeUnit.MILLISECONDS
).getUser(id);
```

## Resource Management

### GRPC-016: Proper Cleanup

```java
// ❌ CRITICAL: Resource leak
public void processUsers(List<Long> userIds) {
    ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 9090).build();
    UserServiceGrpc.UserServiceBlockingStub stub = UserServiceGrpc.newBlockingStub(channel);
    
    for (Long id : userIds) {
        stub.getUser(id); // Channel never closed!
    }
}

// ✅ CORRECT: Use try-with-resources
public void processUsers(List<Long> userIds) {
    try (ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 9090)
            .usePlaintext()
            .build()) {
        
        UserServiceGrpc.UserServiceBlockingStub stub = 
            UserServiceGrpc.newBlockingStub(channel);
        
        for (Long id : userIds) {
            stub.withDeadlineAfter(5, TimeUnit.SECONDS).getUser(id);
        }
    }
}
```

## Protobuf Best Practices

### GRPC-014: Package Naming

```protobuf
// ❌ WARNING: Missing java_package
package com.example.userservice;

// ✅ CORRECT: Use java_package option
syntax = "proto3";

package com.example.userservice;

option java_package = "com.example.user.service";
option java_outer_classname = "UserServiceProto";
option java_multiple_files = true;

service UserService {
    rpc GetUser (GetUserRequest) returns (User);
    rpc CreateUser (CreateUserRequest) returns (User);
}
```

---

## Quick Checklist

- [ ] Reuse ManagedChannel and stubs
- [ ] Set deadlines for all calls
- [ ] Handle all gRPC status codes
- [ ] Use proper error status (not OK with error body)
- [ ] Enable TLS in production
- [ ] Use interceptors for auth/logging
- [ ] Close resources with try-with-resources
- [ ] Use streaming for large data
- [ ] Configure keepalive
- [ ] Add circuit breaker for resilience
