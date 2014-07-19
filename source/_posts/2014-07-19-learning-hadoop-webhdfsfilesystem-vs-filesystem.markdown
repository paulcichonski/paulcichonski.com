---
layout: post
title: "Learning Hadoop: WebHdfsFileSystem vs FileSystem"
date: 2014-07-19 10:25
comments: true
published: true
categories: 
- hdfs 
- hadoop 
- scala
---

For the last few weeks I've had the chance to start digging into the [Hadoop](http://hadoop.apache.org/) ecosystem focusing mainly on 
[Spark](http://spark.apache.org/) for distributed compute (both batch and streaming) as well as 
[HDFS](http://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html) for data storage. The first thing
I noticed is how massive the Hadoop ecosystem. The word *Hadoop* refers to many different technologies that users
may deploy in many different ways to solve specific Big Data usecases. So referring to *Hadoop* is similar to referring to phrases
like *IT Security* in that it very ambiguous until you start digging down into the specifics of the Hadoop deployment.

Enough of the high level speak though, what I really want to talk about is the pain I experienced just trying to get data in and out of 
HDFS. Most of the pain was self-inflicted as my mental model going into the problem was induced from over a year working with 
[Cassandra](http://cassandra.apache.org/), which is a much simpler system for storing data albeit does not provide as good of a foundation 
for storing raw data in a [lambda architecture](http://lambda-architecture.net/) type design. In Cassandra you have the cluster and you
have the client, where the client is your application and it speaks to the cluster over the network in a fairly typical client-server model.
 
I quickly discovered that my [map was not the territory](http://en.wikipedia.org/wiki/Map%E2%80%93territory_relation) when I started writing some simple code for sending data into HDFS:

```scala
package data.generator

import com.twitter.elephantbird.mapreduce.io.ProtobufBlockWriter
import data.generator.DataUtil.TestMessage
import org.apache.hadoop.conf.Configuration
import org.apache.hadoop.fs.{FileSystem, Path}
import org.apache.hadoop.io.compress

import scala.compat.Platform
import scalaz.EphemeralStream

object DataWriterTest {

  def main(args: Array[String]) {
    writeSomeData("hdfs://192.168.50.25:50010", 1000000, "/tmp/test.snappy")
  }

  def writeSomeData(hdfsURI: String, numberOfMessages: Int, destPath: String) {
    def getSnappyWriter(): ProtobufBlockWriter[TestMessage] = {
      val conf = new Configuration()
      val fs = FileSystem.get(new Path(hdfsURI).toUri, conf)
      val outputStream = fs.create(new Path(destPath), true)
      val codec = new compress.SnappyCodec()
      codec.setConf(conf);
      val snappyOutputStream = codec.createOutputStream(outputStream)
      new ProtobufBlockWriter[TestMessage](snappyOutputStream, classOf[TestMessage])
    }
    def getMessage(messageId: Long): TestMessage = DataUtil.createRandomTestMessage(messageId);
    // produces a lazy stream of messages
    def getMessageStream(messageId: Long): EphemeralStream[TestMessage] = {
      val messageStream: EphemeralStream[TestMessage] = getMessage(messageId) ##:: getMessageStream(messageId + 1)
      return messageStream
    }
    val writer = getSnappyWriter()
    getMessageStream(1).takeWhile(message => message.getMessageId < numberOfMessages)
      .foreach(message => writer.write(message))
    writer.finish()
    writer.close()
  }
}
```

As far as I could tell this pattern for writing was consistent with most of the tutorials I found via Google, the only new thing I added
in was the use of Twitter's [elephantbird](https://github.com/kevinweil/elephant-bird/) library to write Snappy compressed Protocol Buffer data.
So I was surprised when I saw (and kept seeing) the following errors:

Client Errors:

```bash
Exception in thread "main" java.io.IOException: Failed on local exception: java.io.EOFException; Host Details : local host is: ".local/192.168.1.2"; destination host is: "localhost.localdomain":50010; 
	at org.apache.hadoop.net.NetUtils.wrapException(NetUtils.java:764)
	at org.apache.hadoop.ipc.Client.call(Client.java:1413)
	at org.apache.hadoop.ipc.Client.call(Client.java:1362)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:206)
	at com.sun.proxy.$Proxy9.create(Unknown Source)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:186)
	at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:102)
	at com.sun.proxy.$Proxy9.create(Unknown Source)
	at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.create(ClientNamenodeProtocolTranslatorPB.java:258)
	at org.apache.hadoop.hdfs.DFSOutputStream.newStreamForCreate(DFSOutputStream.java:1598)
	at org.apache.hadoop.hdfs.DFSClient.create(DFSClient.java:1460)
	at org.apache.hadoop.hdfs.DFSClient.create(DFSClient.java:1385)
	at org.apache.hadoop.hdfs.DistributedFileSystem$6.doCall(DistributedFileSystem.java:394)
	at org.apache.hadoop.hdfs.DistributedFileSystem$6.doCall(DistributedFileSystem.java:390)
	at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
	at org.apache.hadoop.hdfs.DistributedFileSystem.create(DistributedFileSystem.java:390)
	at org.apache.hadoop.hdfs.DistributedFileSystem.create(DistributedFileSystem.java:334)
	at org.apache.hadoop.fs.FileSystem.create(FileSystem.java:906)
	at org.apache.hadoop.fs.FileSystem.create(FileSystem.java:887)
	at org.apache.hadoop.fs.FileSystem.create(FileSystem.java:784)
	at data.generator.DataWriterTest$.getSnappyWriter$1(DataWriterTest.scala:23)
	at data.generator.DataWriterTest$.writeSomeData(DataWriterTest.scala:35)
	at data.generator.DataWriterTest$.main(DataWriterTest.scala:16)
	at data.generator.DataWriterTest.main(DataWriterTest.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:134)
Caused by: java.io.EOFException
	at java.io.DataInputStream.readInt(DataInputStream.java:392)
	at org.apache.hadoop.ipc.Client$Connection.receiveRpcResponse(Client.java:1053)
	at org.apache.hadoop.ipc.Client$Connection.run(Client.java:948)
```

Server Errors (hdfs datanode log):

```bash
ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: localhost.localdomain:50010:DataXceiver error processing unknown operation  src: /192.168.50.2:56439 dest: /192.168.50.25:50010
java.io.IOException: Version Mismatch (Expected: 28, Received: 26738 )
	at org.apache.hadoop.hdfs.protocol.datatransfer.Receiver.readOp(Receiver.java:57)
	at org.apache.hadoop.hdfs.server.datanode.DataXceiver.run(DataXceiver.java:206)
	at java.lang.Thread.run(Thread.java:744)
```

After about 8 hours of lots of googling and ensuring all client and server jars were the same version my surprise turned to frustration. Especially since
the error messages were so vague. 

Finally after a few circuits at the gym, I started from scratch and read through all the documentation I could find about typical data loading strategies for HDFS 
(something I should have done to begin with). This led me to the realization that my client-server mental model was flawed within the HDFS context 
since HDFS makes no assumption about where the data is being written from (in fact it seems to assume that the client is local to the cluster).
Some quick exploration of the `org.apache.hadoop.fs.FileSystem` class hierarchy showed that there are a variety of different ways for writing
to HDFS and only some of them are over TCP/IP. So with a little refactoring to use the `org.apache.hadoop.hdfs.web.WebHdfsFileSystem` implementation
my code works just fine:

Note the new `webhdfs://` protocol in the URI and the new port of `50070`. There seems to be a tight coupling of protocol to `FileSystem`
implementation as well as port mapping, but I have not found great documentation yet as to what this coupling is.

```scala
 package data.generator
 
 import java.net.URI
 
 import com.twitter.elephantbird.mapreduce.io.ProtobufBlockWriter
 import data.generator.DataUtil.TestMessage
 import org.apache.hadoop.conf.Configuration
 import org.apache.hadoop.fs.{FileSystem, Path}
 import org.apache.hadoop.hdfs.web.WebHdfsFileSystem
 import org.apache.hadoop.io.compress
 
 import scala.compat.Platform
 import scalaz.EphemeralStream
 
 object DataWriterTest {
 
   def main(args: Array[String]) {
     writeSomeData("webhdfs://192.168.50.25:50070", 1000000, "/tmp/test2.snappy")
   }
 
   def writeSomeData(hdfsURI: String, numberOfMessages: Int, destPath: String) {
     def getSnappyWriter(): ProtobufBlockWriter[TestMessage] = {
       val conf = new Configuration()
       val fs = new WebHdfsFileSystem();
       fs.initialize(new Path(hdfsURI).toUri, conf)
       val outputStream = fs.create(new Path(destPath), true)
       val codec = new compress.SnappyCodec()
       codec.setConf(conf);
       val snappyOutputStream = codec.createOutputStream(outputStream)
       new ProtobufBlockWriter[TestMessage](snappyOutputStream, classOf[TestMessage])
     }
     def getMessage(messageId: Long): TestMessage = DataUtil.createRandomTestMessage(messageId);
     // produces a lazy stream of messages
     def getMessageStream(messageId: Long): EphemeralStream[TestMessage] = {
       val messageStream: EphemeralStream[TestMessage] = getMessage(messageId) ##:: getMessageStream(messageId + 1)
       return messageStream
     }
     val writer = getSnappyWriter()
     getMessageStream(1).takeWhile(message => message.getMessageId < numberOfMessages)
       .foreach(message => writer.write(message))
     writer.finish()
     writer.close()
   }
 }
```
 
 File in hdfs:
 
```bash
$ hdfs dfs -ls -h /tmp
Found 1 item
-rw-r--r--   3 pcichonski supergroup    215.9 M 2014-07-19 08:44 /tmp/test2.snappy
```
 
I'm still not sure this is the write method for writing large amounts of data to HDFS, but more dots are starting to connect in my head
about how the component parts of the ecosystem fit together. Lots more to learn though.
 
