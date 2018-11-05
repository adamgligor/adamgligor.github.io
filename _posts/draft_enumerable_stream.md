---
draft: true
---


```c#
  class Program
    {
        internal static IEnumerable<(int Id, string Title)> FetchAll()
        {
            //TODO: use entity framework to retrive some data
            return Enumerable.Empty<(int, string)>();
        }

        static void Main(string[] args)
        {

            var allBytes = FetchAll()
                .Select(r => $"{r.Id}|{r.Title}\r\n")
                .Select(s => Encoding.UTF8.GetBytes(s))
                .SelectMany(l => l);

            using (var sourceStream = new EnumerableStream(allBytes))
            {
                using (FileStream compressedFileStream = File.Create($"out.csv.gz.tmp"))
                {
                    using (var compressionStream = new GZipStream(compressedFileStream, CompressionMode.Compress))
                    {
                        sourceStream.CopyTo(compressionStream);
                    }
                }
            }

        }
    }
```

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

``link to stream inplementation 

https://docs.microsoft.com/en-us/dotnet/api/system.io.stream?view=netframework-4.7.2


//tip file naming foo.csv.gz