# Building a gRPC Server and Client in Java: A CarPark Example

## What Is gRPC?

> **Definition:** gRPC is a modern open-source high-performance Remote Procedure Call (RPC) framework developed by Google. It can efficiently connect services in and across data centers with pluggable support for load balancing, tracing, health checking, and authentication.

## Why gRPC Over REST?

### Protobuf Instead of JSON/XML

REST uses JSON for sending and receiving messages. JSON is flexible, text-based, and human-readable, but it is not fast enough or lightweight enough for transmitting data between systems.

> **Key point:** gRPC uses the Protobuf (Protocol Buffers) messaging format, which is faster and more efficient for data transmission.

### First-Class Support for Code Generation

REST needs third-party tools to generate the code for API calls. gRPC comes with an in-built `protoc` compiler, which provides code generation features by default.

### Built on HTTP/2

REST is built on HTTP/1.1, which uses a request-response model of communication. gRPC uses HTTP/2, which supports client-response communication and bidirectional streaming.

### Works Across Languages and Platforms

Since the code can be generated for any language, it is easy to create microservices in any language that can interact with each other, enabling communication across languages.

## The 4 Types of APIs in gRPC

gRPC supports four types of API methods, illustrated in the following service definition:

```protobuf
service GreetService {
  // Unary
  rpc Greet(GreetRequest) returns (GreetResponse) {};

  // Streaming server
  rpc GreetManyTimes(GreetManyTimesRequest) returns (stream GreetManyTimesResponse) {};

  // Streaming client
  rpc LongGreet(stream LongGreetRequest) returns (LongGreetResponse) {};

  // Bidirectional streaming
  rpc GreetAll(stream GreetAllRequest) returns (stream GreetAllResponse) {};
}
```

The four types are:

- **Unary**: A single request returns a single response.
- **Server streaming**: A single request returns a stream of responses.
- **Client streaming**: A stream of requests returns a single response.
- **Bidirectional streaming**: Both client and server send a stream of messages to each other.

## Implementing a gRPC Server and Client in Java: The CarPark Example

With the basics of gRPC covered, this section walks through implementing a gRPC server and client using Java, building a **CarPark** service.

### Steps for the Sample

1. Create a Maven project.
2. Define a service in a `.proto` file.
3. Generate server and client code using the Protocol Buffer compiler.
4. Build the server application and implement the generated service interfaces, spawning the gRPC server.
5. Build the client application, making RPC calls using generated stubs.

> The complete source code for this sample is available on [GitHub](https://github.com/pamodaaw/grpc-sample), with inline comments for further detail.

## Step 1: Create a Maven Project

Create a Maven project (refer to the complete [pom.xml](https://github.com/pamodaaw/grpc-sample/blob/main/pom.xml) of the sample) and add the following dependencies:

```xml
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-netty</artifactId>
    <version>1.36.0</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-protobuf</artifactId>
    <version>1.36.0</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-stub</artifactId>
    <version>1.36.0</version>
</dependency>
```

Add the [protobuf-maven-plugin](https://github.com/pamodaaw/grpc-sample/blob/main/pom.xml#L33), which is responsible for code generation:

```xml
<build>
    <extensions>
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.6.2</version>
        </extension>
    </extensions>
    <plugins>
        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.6.1</version>
            <configuration>
                <protocArtifact>
                    com.google.protobuf:protoc:3.3.0:exe:${os.detected.classifier}
                </protocArtifact>
                <pluginId>grpc-java</pluginId>
                <pluginArtifact>
                    io.grpc:protoc-gen-grpc-java:1.4.0:exe:${os.detected.classifier}
                </pluginArtifact>
                <protoSourceRoot>src/main/resources</protoSourceRoot>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>compile-custom</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

> **Note:** `<protoSourceRoot>` indicates the place where the service definition file is located. This sample uses Java 8.

## Step 2: Define the Service

Defining the service means specifying the methods that can be remotely called, along with their parameters and return types. This is done in the `.proto` file using Protocol Buffers, which also describes the structure of the payload messages.

> The complete proto file for the sample is available [here](https://github.com/pamodaaw/grpc-sample/blob/main/src/main/resources/CarPark.proto).

### Add the Common Configurations

```protobuf
syntax = "proto3";
option java_multiple_files = true;
```

- **syntax**: Indicates the syntax of this file.
- **java_multiple_files**: Instructs the compiler to generate Java code in separate files. Otherwise, all generated code will be in a single Java file.

### Define the Message Formats

```protobuf
message Vehicle {
  string vehicle_number = 1;
  string vehicle_type = 2;
}

message ParkRequest {
  Vehicle vehicle = 1;
}
```

In this example, `vehicle_number` and `vehicle_type` are the attributes of the `Vehicle` message, and the given value is the field order. `ParkRequest` is another message format. All message formats needed for the service can be defined similarly.

### Define the Service and the Methods

```protobuf
service CarParkService {
  rpc parkVehicle(ParkRequest) returns (ParkResponse);
}
```

- **CarParkService**: This is the gRPC service.
- **parkVehicle**: This is a method in the `CarParkService`.
- **ParkRequest**: This is the input parameter for the `parkVehicle` method.
- **ParkResponse**: This is the response of the `parkVehicle` method.

### Define Another Method with Streaming

```protobuf
service CarParkService {
  rpc parkVehicle(ParkRequest) returns (ParkResponse);
  rpc parkVehicleManyTimes(ParkRequest) returns (stream ParkResponseManyTimes);
}
```

In this method, the server will stream multiple responses back to the client.

## Step 3: Generate the Source Code

Run the following command to compile the project and generate the Java code for the defined service:

```bash
mvn clean install
```

The generated Java classes will appear inside `<SAMPLE_HOME>/target/generated-sources/protobuf/java`.

## Step 4: Implement the Server

### 1. Create a Server Package

Create a separate package for the server , for this sample, `org.sample.park.server`.

### 2. Implement the Service

Implement the service by extending the generated service base class. The base class contains all the methods defined in the proto file. The implementation is in the [CarParkServiceImpl](https://github.com/pamodaaw/grpc-sample/blob/main/src/main/java/org/sample/park/server/CarParkServiceImpl.java) class:

```java
public class CarParkServiceImpl extends CarParkServiceGrpc.CarParkServiceImplBase {
```

### 3. Implement the parkVehicle Method

In the service implementation class, implement the `parkVehicle` method by overriding the method of the base class:

```java
public void parkVehicle(ParkRequest request, StreamObserver<ParkResponse> responseObserver) {

    String vehicleNo = request.getVehicle().getVehicleNumber();
    String vehicleType = request.getVehicle().getVehicleType();

    String resultMsg = "The vehicle of number " + vehicleNo + " and type " + vehicleType + " is parked.";
    System.out.println(resultMsg);

    ParkResponse parkResponse = ParkResponse
            .newBuilder()
            .setResult(resultMsg)
            .build();

    responseObserver.onNext(parkResponse);
    responseObserver.onComplete();
}
```

**Explanation:**

- gRPC uses builders for creating objects.
- Inside the `parkVehicle` method, the number and type of the vehicle are printed and then sent back to the client.
- The response is sent by calling the `responseObserver`'s `onNext()` method.
- `onCompleted()` specifies that the RPC is finished , otherwise, the connection will hang, and the client will wait for more information to arrive.

### 4. Implement the parkVehicleManyTimes Method

Implement the `parkVehicleManyTimes` method by overriding the method of the base class, where the server sends several messages to the client:

```java
@Override
public void parkVehicleManyTimes(ParkRequest request, StreamObserver<ParkResponseManyTimes> responseObserver) {

    String vehicleNo = request.getVehicle().getVehicleNumber();
    String vehicleType = request.getVehicle().getVehicleType();

    String response1 = "The vehicle of number " + vehicleNo + " and type " + vehicleType + " entered the park.";
    ParkResponseManyTimes parkResponse1 = ParkResponseManyTimes
            .newBuilder()
            .setResult(response1)
            .build();
    responseObserver.onNext(parkResponse1);

    Thread.sleep(10000);

    // Build and send the second response.
    String response2 = "The ground floor is full.";
    ParkResponseManyTimes parkResponse2 = ParkResponseManyTimes
            .newBuilder()
            .setResult(response2)
            .build();
    responseObserver.onNext(parkResponse2);

    Thread.sleep(10000);

    // Build and send the third response.
    String response3 = "The vehicle of number " + vehicleNo + " and type " + vehicleType + " is parked in the first floor";
    ParkResponseManyTimes parkResponse3 = ParkResponseManyTimes
            .newBuilder()
            .setResult(response3)
            .build();
    responseObserver.onNext(parkResponse3);

    // Complete the communication.
    responseObserver.onCompleted();
}
```

**Explanation:** Here, the server sends 3 responses to the client, waiting 10 seconds before each response.

### 5. Start the Server

Write a separate Java class to start the server, named [CarParkServer](https://github.com/pamodaaw/grpc-sample/blob/main/src/main/java/org/sample/park/server/CarParkServer.java). The service is added and the server is started with a given port, then kept alive for the client to make requests:

```java
public static void main(String[] args) throws IOException, InterruptedException {

    Server server = ServerBuilder.forPort(5003)
            .addService(new CarParkServiceImpl())
            .build();

    server.start();
    server.awaitTermination();
}
```

## Step 5: Implement the Client

### 1. Create a Client Package

Create a separate package for the client , for this sample, `org.sample.park.client`.

### 2. Write a Class to Start the Client

The client class is [CarParkClient](https://github.com/pamodaaw/grpc-sample/blob/main/src/main/java/org/sample/park/client/CarParkClient.java).

### 3. Initiate Communication with the Server

In the main method of the client, initiate communication with the server using a channel:

```java
ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 5003)
        .usePlaintext()
        .build();
```

### 4. Use the Stub to Call Service Methods

Inside the same main method, use the stub of the `CarParkService` to call the methods of the service. The stub is the primary way for clients to interact with the server:

```java
CarParkServiceGrpc.CarParkServiceBlockingStub stub = CarParkServiceGrpc.newBlockingStub(channel);
```

> **Note:** This uses a blocking/synchronous stub, so the RPC call waits for the server to respond, either returning a response or raising an exception. gRPC also provides two other types of stubs that facilitate non-blocking/asynchronous calls.

### 5. Call the parkVehicle Method

In the same main method, define a `ParkRequest` to call the `parkVehicle` method, then call the method and capture the `ParkResponse` returned:

```java
ParkRequest parkRequest = ParkRequest.newBuilder()
        .setVehicle(Vehicle.newBuilder()
                .setVehicleNumber("NA-1324")
                .setVehicleType("BUS")
                .build())
        .build();

ParkResponse parkResponse = stub.parkVehicle(parkRequest);
System.out.println("Response for first call: " + parkResponse.getResult());
```

### 6. Call the parkVehicleManyTimes Method

In the same main method, call the `parkVehicleManyTimes` method and handle the streaming response from the server:

```java
ParkRequest parkRequest2 = ParkRequest.newBuilder()
        .setVehicle(Vehicle.newBuilder()
                .setVehicleNumber("PE-6723")
                .setVehicleType("VAN")
                .build())
        .build();

System.out.println("Response for the second call: ");
stub.parkVehicleManyTimes(parkRequest2).forEachRemaining(parkResponseManyTimes -> {
    System.out.println(parkResponseManyTimes.getResult());
});
```

## Try Out the Service

To test the sample:

1. Run the server.
2. Run the client.

You will then see the corresponding messages printed to the console, showing the unary response for the first call and the streamed responses for the second call.

## Closing Thoughts

Although the walkthrough involves several code snippets, the overall steps for building a gRPC service , defining a `.proto` file, generating code, implementing the server, and implementing the client , are straightforward. Trying out the sample is a good way to see firsthand how easy gRPC is to work with.
