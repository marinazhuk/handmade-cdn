# Handmade Cdn

The goal is to create a handmade Cdn for delivering millions of images across the globe.

Using:
* Bind9 with GeoIP-based access control lists for GeoDNS
* 2 Ngix proxy servers for:
  * Load balancer 1 - to serve others
  * Load balancer 2 - to serve North America clients
* 4 Ngix caching servers:
  * 2 servers for first Load Balancer
  * 2 servers for second Load Balancer 

## Testing

### Bind configuration
 * [GeoIP.acl](bind%2Fconf%2FGeoIP.acl) file was modified for testing purposes:
```properties
acl US {
    10.5.0.10; #-record was added
	1.1.1.0/32;
	1.1.1.2/31;
        ...
```

* 10.5.0.10 - north-america region

  goes to Load Balancer 2 (IP 10.5.0.4 ) for north-america    

* 10.5.0.11 - others

  goes to Load Balancer 1 (IP 10.5.0.3) for other countries

### Load Balancers configuration

* Load balancer 1 with Least Connections balancing strategy
* Load balancer 2 with Round Robin balancing strategy


* Headers were added:
  * **X-Client-Ip** - shows client IP
  * **X-Load-Balancer-Name** with values **lb_1_others** or **lb_2_north-america** - shows selected Load Balancer
  * **X-Caching-Server-Ip** - shows the node selected by Load Balancer



### Tests
1) Start **'client'** container with IP *10.5.0.10* 

Comand:

```
curl -I  http://cdn.zhuk.com/cat.jpg
```

first run  result: 
Load Balancer 2, Caching-Server-Ip=10.5.0.7
```properties
 HTTP/1.1 200 OK
 Server: nginx/1.25.2
 Date: Mon, 08 Jan 2024 23:22:03 GMT
 Content-Type: image/jpeg
 Content-Length: 234782
 Connection: keep-alive
 Last-Modified: Mon, 08 Jan 2024 03:29:50 GMT
 ETag: "659b6c2e-3951e"
 Accept-Ranges: bytes
 X-Cache-Status: HIT

 X-Client-Ip: 10.5.0.10
 X-Load-Balancer-Name: lb_2_north-america
 X-Caching-Server-Ip: 10.5.0.7:80
```
second run result:
   Load Balancer 2, Caching-Server-Ip=10.5.0.8
```properties
 HTTP/1.1 200 OK
 Server: nginx/1.25.2
 Date: Mon, 08 Jan 2024 23:22:43 GMT
 Content-Type: image/jpeg
 Content-Length: 234782
 Connection: keep-alive
 Last-Modified: Mon, 08 Jan 2024 03:29:50 GMT
 ETag: "659b6c2e-3951e"
 Accept-Ranges: bytes
 X-Cache-Status: HIT

 X-Client-Ip: 10.5.0.10
 X-Load-Balancer-Name: lb_2_north-america
 X-Caching-Server-Ip: 10.5.0.8:80
```

2) Start **'client'** container with IP *10.5.0.11*

```
curl -I  http://localhost/cat.jpg
```
first run result: Load Balancer 1, Caching-Server-Ip=10.5.0.5
```properties
 HTTP/1.1 200 OK
 Server: nginx/1.25.2
 Date: Mon, 08 Jan 2024 23:24:51 GMT
 Content-Type: image/jpeg
 Content-Length: 234782
 Connection: keep-alive
 Last-Modified: Mon, 08 Jan 2024 03:29:50 GMT
 ETag: "659b6c2e-3951e"
 Accept-Ranges: bytes
 X-Cache-Status: MISS

 X-Client-Ip: 10.5.0.11
 X-Load-Balancer-Name: lb_1_others
 X-Caching-Server-Ip: 10.5.0.5:80
```

second run result: Load Balancer 1, Caching-Server-Ip=10.5.0.6
```properties
 HTTP/1.1 200 OK
 Server: nginx/1.25.2
 Date: Mon, 08 Jan 2024 23:27:10 GMT
 Content-Type: image/jpeg
 Content-Length: 234782
 Connection: keep-alive
 Last-Modified: Mon, 08 Jan 2024 03:29:50 GMT
 ETag: "659b6c2e-3951e"
 Accept-Ranges: bytes
 X-Cache-Status: MISS

 X-Client-Ip: 10.5.0.11
 X-Load-Balancer-Name: lb_1_others
 X-Caching-Server-Ip: 10.5.0.6:80
```
### Metrics Results

Run **'Siege'** container with command:

```
siege -c25 -t15S "http://cdn.zhuk.com/cat.jpg"
```

|                         | With Cache + Least Conn | With Cache + Round Robin | Without Cache + Least Conn | Without Cache + Round Robin |
|-------------------------|-------------------------|--------------------------|----------------------------|:---------------------------:|
| Transactions            | 1414 hits               | 1421 hits                |                
| Availability            | 100.00 %                | 100.00 %                 |                
| Elapsed time            | 14.60 secs              | 14.55 secs               |                
| Data transferred        | 316.60 MB               | 318.17 MB                |            
| Response time           | 0.01 secs               | 0.01 secs                |               
| Transaction rate        | 96.85 trans/sec         | 97.66 trans/sec          |            
| Throughput              | 21.69 MB/sec            | 21.87 MB/sec             |                  
| Concurrency             | 0.94                    | 0.87                     |                 
| Successful transactions | 1414                    | 1421                     |     
| Failed transactions     | 0                       | 0                        |         
| Longest transaction     | 0.25                    | 0.23                     |         
| Shortest transaction    | 0.00                    | 0.00                     |        	