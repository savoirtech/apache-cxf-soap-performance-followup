# Apache CXF SOAP Performance Follow Up

In our last
[post](https://github.com/savoirtech/apache-cxf-soap-performance), we
recounted our journey to processing 1 Billion SOAP invocations in an
eight-hour period.

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
$mvn mvn -Pclient -Dhost=192.168.50.154 -Dprotocol=http -Doperation=echoComplexTypeDoc -Dthreads=100 -Dtime=28800
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
Overall Throughput: echoComplexTypeDoc ??? (invocations/sec)
Overall AVG. response time: ??? (ms)
??? (invocations), running ??? (sec)
============================================
```

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
$mvn mvn -Pclient -Dhost=192.168.50.154 -Dprotocol=http -Doperation=echoComplexTypeDoc -Dthreads=100 -Dtime=28800
```

This resulted in:

``` bash
=============Overall Test Result============
Overall Throughput: echoComplexTypeDoc ??? (invocations/sec)
Overall AVG. response time: ??? (ms)
??? (invocations), running ??? (sec)
============================================
```

Note: in this iteration we do not tune ZGC, we only use its defaults.

# Can you run this using GraalVM?

Unfortunately, there does not appear to be a PPC64LE build available.

# Can you switch around Client and Server hosts?

Sure!

For our run we’ll reuse the existing Java 17 heap & and GC tuning, with
the x64 box being the server, and PPC64LE being the client host.

This resulted in:

``` bash
=============Overall Test Result============
Overall Throughput: echoComplexTypeDoc ??? (invocations/sec)
Overall AVG. response time: ??? (ms)
??? (invocations), running ??? (sec)
============================================
```
