# Apache CXF SOAP Performance Follow Up

In our last
[post](https://github.com/savoirtech/apache-cxf-soap-performance), we
recounted our journey to processing 1 Billion SOAP invocations in an
eight-hour period. This lead to several readers reaching out with
suggestions for tweaks to the scenarios run, or entirely new
configurations. In this post we seek to provide some answers.

<figure>
<img src="./assets/images/CXFLab.png" alt="CXFLab" />
</figure>

## Brief Recap

After several iterations we found that we could reach our target using
the following Client and Server side settings:

``` bash
MAVEN_OPTS="-Xms112640m -Xmx112640m -Dmaven.artifact.threads=5 -XX:MaxGCPauseMillis=600 -XX:+ParallelRefProcEnabled"
```

``` bash
$mvn -Pserver -Dhost=0.0.0.0 -Dprotocol=http
```

``` bash
$mvn -Pclient -Dhost=192.168.50.154 -Dprotocol=http -Doperation=echoComplexTypeDoc -Dthreads=100 -Dtime=28800
```

This resulted in:

``` bash
=============Overall Test Result============
Overall Throughput: echoComplexTypeDoc 348.5338799400768 (invocations/sec)
Overall AVG. response time: 2.86916152935241 (ms)
1.003791036E9 (invocations), running 2880038.624 (sec)
============================================
```

That’s 1,003,791,036 invocations in eight hours.

# Can you try Java 21 again with Java 17’s Heap & Tuning?

Sure!

This resulted in:

``` bash
=============Overall Test Result============
Overall Throughput: echoComplexTypeDoc 334.0005516005607 (invocations/sec)
Overall AVG. response time: 2.9940070314492293 (ms)
9.61925455E8 (invocations), running 2880011.5759999994 (sec)
============================================
```

## How does that compare to our last Java 21 test?

In our prior lab test we achieved 924,006,644 invocations in
eight-hours, this run provided 961,925,455 - a delta improvement of
37,918,811 invocations. Still well short of the 1 Billion goal.

# Can you try Java 17 with Zero GC instead of G1GC?

To attempt this run, we update JAVA_HOME to Java 17, then use the
following:

``` bash
MAVEN_OPTS="-XX:+UnlockExperimentalVMOptions -XX:+UseZGC -XX:+UseDynamicNumberOfGCThreads -Xms112640m -Xmx112640m -Dmaven.artifact.threads=5"
```

``` bash
$mvn -Pserver -Dhost=0.0.0.0 -Dprotocol=http
```

``` bash
$mvn -Pclient -Dhost=192.168.50.154 -Dprotocol=http -Doperation=echoComplexTypeDoc -Dthreads=100 -Dtime=28800
```

This resulted in:

``` bash
=============Overall Test Result============
Overall Throughput: echoComplexTypeDoc 304.85782225783754 (invocations/sec)
Overall AVG. response time: 3.2802176194588073 (ms)
8.77995015E8 (invocations), running 2880014.718 (sec)
============================================
```

## How does that compare to our tuned G1GC Java 17 test?

In this iteration we did not tune ZGC, we only use its defaults. The
final throughput came in at 877,995,015 invocations — significantly
behind all our other runs. We could likely spend some time improving the
configuration for this GC strategy.

# Can you run this using GraalVM?

Unfortunately, there does not appear to be a PPC64LE build available.

# Can you switch around Client and Server hosts?

Sure!

For our run we’ll reuse the existing Java 17 heap & and GC tuning, with
the x64 box being the server, and PPC64LE being the client host.

This resulted in:

``` bash
=============Overall Test Result============
Overall Throughput: echoComplexTypeDoc 171.77497864493685 (invocations/sec)
Overall AVG. response time: 5.821569636559379 (ms)
4.94715503E8 (invocations), running 2880020.751 (sec)
============================================
```

That outcome was not expected…​

The Intel Server - IBM Client system managed to process 494,715,503 in
eight hours - less than half the throughput.

Lets run a few quick tests to see where to dial in client numbers for
this system.

## Theory Time!

Given our goal of achieving 1 Billion invocations in an eight-hour
period, lets take a look at what velocity our clients will need to
maintain to collectively reach our goal line. Before setting up a full
testing run, we run a 60-second quick test to see what throughput we
might expect to see on our lab hardware (Intel running the Server-Side,
IBM running the Clients).

| Clients | Target Invocations/Second per client | Quick Test (Reality) per client |
|----|----|----|
| 1 | 34722.2 | 803.82 (re-ran test to confirm value) |
| 8 | 4340.27 | 1366.39 |
| 16 | 2170.14 | 1033.03 |
| 32 | 1085.07 | 548.34 |
| 64 | 542.53 | 275.75 |
| **100** | ***347.2*** | ***179.31*** (prior system thread count) |
| 128 | 271.27 | 142.45 |
| 256 | 135.63 | 70.24 |
| 512 | 67.81 | 36.75 |
| 1024 | 33.90 | 20.03 |

The pairing tests does not suggest a possible convergence point which
would suggest a 1 Billion invocation run would be successful.

# About the Authors

[Jamie
Goodyear](https://github.com/savoirtech/blogs/blob/main/authors/JamieGoodyear.md)

# Reaching Out

Please do not hesitate to reach out with questions and comments, here on
the Blog, or through the Savoir Technologies website at
<https://www.savoirtech.com>.

# With Thanks

Thank you to the Apache CXF community.

\(c\) 2024 Savoir Technologies
