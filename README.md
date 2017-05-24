## RPC Benchmark

Benchmark for grpc-go and thrift-go

**Machines**

Two with same configuration, one for client, one for server
* CPU: 64 cores  Intel(R) Xeon(R) CPU E5-2698 v3 @ 2.30GHz
* Memory: 512G
* OS: Linux version 3.10.0-229.el7.x86_64, CentOS 6.4
* Go: server 1.8, client 1.7
* gRPC: 1.3.0
* Thrift: 0.10.0

**Comparison**

* QPS
* Latency(mean, median, max, P99), min latency is always 0
* CPU
* Memory not listed(Go has a very small footprint of memory usage)
* Long connection(one conn for a client to do all RPCs), short connection(one conn for each RPC)
* Concurrency numbers of clients, for long connection is 100, 1000, 5000 and 10000, for short connection is 10, 50 and 100

**Work Pattern**

Thrift-go and gRPC-go hava a same backend work pattern, one goroutine for each connection.
1. Thrift-go

```go
func (p *TSimpleServer) AcceptLoop() error {
	for {
		client, err := p.serverTransport.Accept()
		if err != nil {
			... ...
			return err
		}
		if client != nil {
			p.Add(1)
			go func() {
				if err := p.processRequests(client); err != nil {
					log.Println("error processing request:", err)
				}
			}()
		}
	}
}

func (p *TSimpleServer) Serve() error {
	err := p.Listen()
	if err != nil {
		return err
	}
	p.AcceptLoop()
	return nil
}
```

2. gRPC-go

```go
// Serve accepts incoming connections on the listener lis, creating a new
// ServerTransport and service goroutine for each. The service goroutines
// read gRPC requests and then call the registered handlers to reply to them.

func (s *Server) Serve(lis net.Listener) error {
	... ...

	for {
		rawConn, err := lis.Accept()
		if err != nil {
			... ...
		}
		tempDelay = 0
		// Start a new goroutine to deal with rawConn
		// so we don't stall this Accept loop goroutine.
		go s.handleRawConn(rawConn)
	}
}
```

**Protocal**

1. Protobuf

```protobuf
syntax = "proto3";

package main;

service Hello {
  // Sends a greeting
  rpc Say (BenchmarkMessage) returns (BenchmarkMessage) {}
}


message BenchmarkMessage {
  string field1 = 1;
  string field9 = 9;
  string field18 = 18;
  bool field80 = 80 ;
  bool field81 = 81 ;
  int32 field2 = 2;
  int32 field3 = 3;
  int32 field280 = 280;
  int32 field6 = 6 ;
  int64 field22 = 22;
  string field4 = 4;
  repeated fixed64 field5 = 5;
  bool field59 = 59 ;
  string field7 = 7;
  int32 field16 = 16;
  int32 field130 = 130 ;
  bool field12 = 12 ;
  bool field17 = 17 ;
  bool field13 = 13 ;
  bool field14 = 14 ;
  int32 field104 = 104 ;
  int32 field100 = 100 ;
  int32 field101 = 101 ;
  string field102 = 102;
  string field103 = 103;
  int32 field29 = 29 ;
  bool field30 = 30 ;
  int32 field60 = 60 ;
  int32 field271 = 271 ;
  int32 field272 = 272 ;
  int32 field150 = 150;
  int32 field23 = 23 ;
  bool field24 = 24 ;
  int32 field25 = 25 ;
  bool field78 = 78;
  int32 field67 = 67 ;
  int32 field68 = 68;
  int32 field128 = 128 ;
  string field129 = 129 ;
  int32 field131 = 131 ;
}
```

2. Thrift
```thrift
namespace go main

struct BenchmarkMessage
{
  1:  string field1,
  2:  i32 field2,
  3:  i32 field3,
  4:  string field4,
  5:  i64 field5,
  6:  i32 field6,
  7:  string field7,
  9:  string field9,
  12:  bool field12,
  13:  bool field13,
  14:  bool field14,
  16:  i32 field16,
  17:  bool field17,
  18:  string field18,
  22:  i64 field22,
  23:  i32 field23,
  24:  bool field24,
  25:  i32 field25,
  29:  i32 field29,
  30:  bool field30,
  59:  bool field59,
  60:  i32 field60,
  67:  i32 field67,
  68:  i32 field68,
  78:  bool field78,
  80:  bool field80,
  81:  bool field81,
  100:  i32 field100,
  101:  i32 field101,
  102:  string field102,
  103:  string field103,
  104:  i32 field104,
  128:  i32 field128,
  129:  string field129,
  130:  i32 field130,
  131:  i32 field131,
  150:  i32 field150,
  271:  i32 field271,
  272:  i32 field272,
  280:  i32 field280,
}
service Greeter {
    BenchmarkMessage say(1:BenchmarkMessage name);
}
```

**Benchmark**

All RPCs are successful below.

1. gRPC-go, long connection

clients|Latency(median)|Latency(mean)|Latency(P99)|Latency(max)|QPS|CPU
-------------|-------------|-------------|-------------|-------------|-------------|-------------
100|0|1ms|8ms|200ms|95k|2000%
1000|2ms|7ms|400ms|2s|130k|2000%
5000|5ms|35ms|800ms|20s|130k|2000%
10000|8ms|65ms|1.6s|30s|100k|2000%

2. gRPC-go, short connection

clients|Latency(median)|Latency(mean)|Latency(P99)|Latency(max)|QPS|CPU
-------------|-------------|-------------|-------------|-------------|-------------|-------------
10|1ms|1ms|4ms|41ms|6k|600%
50|4ms|4ms|16ms|1s|10k|2000%
100|4ms|6ms|1s|3s|13k|2700%

3. thrift-go, long connection

clients|Latency(median)|Latency(mean)|Latency(P99)|Latency(max)|QPS|CPU
-------------|-------------|-------------|-------------|-------------|-------------|-------------
100|0|1ms|4ms|40ms|100k|1200%
1000|5ms|7ms|220ms|3s|120k|1200%
5000|11ms|37ms|900ms|25s|110k|1200%
10000|13ms|73ms|1.7s|54s|90k|1200%

4. thrift-go, short connection

clients|Latency(median)|Latency(mean)|Latency(P99)|Latency(max)|QPS|CPU
-------------|-------------|-------------|-------------|-------------|-------------|-------------
10|0|0|2ms|27ms|12k|300%
50|0|1ms|13ms|200ms|28k|1400%
100|0|1ms|200ms|210ms|31k|1600%

**Conclusion**

* For long connection, no big difference in QPS, gRPC is better in less latency, thrift is better in less CPU usage
* For short connection, thrift(TCP) outperforms gRPC(HTTP2) in QPS/CPU/Lantency 
* As a result of the work pattern(one goroutine for each connection), for short connection, the CPU usage increases obviously as connections increase, connection limitation is very important, in case of errors as bellow
  > grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: dial tcp: getsockopt: connection refused"
* Latency for a small portion of connections can not be controlled , because GC of Go would stop the world, for real time applications with a very strict latency requirement, use C/C++ version instead

**Thanks**

Inspired by and thanks to [rpcx-ecosystem/rpcx-benchmark](https://github.com/rpcx-ecosystem/rpcx-benchmark).
