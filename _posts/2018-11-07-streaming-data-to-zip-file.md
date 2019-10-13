---
layout: post
title: Pipeline to stream data to a zip file
description: 
date: '2018-11-07'
tags: programming
---

How to export huge amounts of data from a data source like a database into flat files efficiently. 

## Solution

Build a streaming pipeline. The producer end reads and transforms the data, the consumer end archives and writes to a file.  


This will be a c# implementation. I'll use `Streams` and `Enumerables`. 


Streaming data from a database is supported using `Ado.Net's` `DataReader` or `EntityFramework` deferred execution operator like `Select`, these produce an `IEnumerable`. IEnumerable exposes and enumerator which supports iteration of the result. 


Files work with the `Stream` abstraction. `Streams` can be stacked to produce the compressed file directly. Wrap a `FileStream` into a `GZipStream` to write and compress at the same time. Streams work with byte arrays.


What's missing is a way to convert the producer format `IEnumerable` into the consumer format `Stream`

## An enumerable stream

So the simplest thing I came up with, say I have a byte enumerable, and I want to convert that into a stream that supports reading. 


The official documentation on the [Stream](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream?view=netframework-4.7.2) class provides a good guideline of implementing a custom stream. 


I have to override a single method from the stream `Read`, there's the implementation:

```c# 
 
    public class EnumerableStream : Stream
    {
        private readonly IEnumerator<byte> _source;

        public EnumerableStream(IEnumerable<byte> source)
        {
            _source = source.GetEnumerator();
        }

        public override bool CanRead
        {
            get { return true; }
        }

        public override int Read(byte[] buffer, int offset, int count)
        {
            var read = 0;

            while (_source.MoveNext())
            {
                buffer[offset + read] = _source.Current;
                read++;

                if (read == count)
                    break;
            }

            return read;
        }

        // override all other required stream methods and throw `NotImplementedException`
    }
```

## The pipeline

With this bit in place, all I have to do is stitch everything together. The main routine looks like this, with comments explaining the relevant lines.

```c#
class Program
{
    // this is the producer end 
    // use database calls here, I use just a generator function to illustrate the concept
    internal static IEnumerable<(int Id, string Title)> FetchAll()
    {
        return Enumerable.Range(0, 100000).Select(n => (n, n.ToString()));
    }

    static void Main(string[] args)
    {

        var allBytes = 
            //1. stream the data from the data source
            FetchAll() 
            //2. transform a source object into a string
            .Select(r => $"{r.Id}|{r.Title}\r\n") 
            //3. encode the string into a byte array
            .Select(s => Encoding.UTF8.GetBytes(s)) 
            //4. flatten the sequence
            .SelectMany(l => l);

        //5. conversion from IEnumerable to a readable Stream
        using (var sourceStream = new EnumerableStream(allBytes)) 
        {
            using (FileStream compressedFileStream = File.Create($"out.csv.gz.tmp"))
            {
                using (var compressionStream = 
                    new GZipStream(compressedFileStream, CompressionMode.Compress))
                {
                    //6. read the source stream and write it to the destination stream 
                    sourceStream.CopyTo(compressionStream);
                }
            }
        }

        File.Move("out.csv.gz.tmp", "out.csv.gz");
    }
}
```

**NOTE**

 The name of the file inside the archive is inferred from the name of the archive. In my case unpacking a `out.csv.gz` results in a file called `out.csv`

