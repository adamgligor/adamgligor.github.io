---
layout: post
title: Wcf soap proxy generation error
date: '2016-09-13'
tags: programming
---

In .net tooling there is a limitation in the XML Schema Definition Tool (xsd.exe) that causes the proxy classes to be generated incorrectly for certain cases of "unbounded" elements.

## The problem

I encountered this issue while working on a project that consumes a third party soap service.  

To consume soap services in c# client the common practice is to rely on framework tooling to generate service proxy classes (also known as "add service reference" from Visual Studio). Just point the XML Schema Definition Tool (xsd.exe) that is responsible for the proxy generation and the proxy will be created automatically. Once the proxy is there, it can be updated to get access to new methods and types, update is also an automated operation.

In my case the code and proxy was already there, I only had to update the proxy and expose some new functionality. The issue appeared right after updating the proxy, I got this error when executing any request:

**{"Unable to generate a temporary class (result=1).\r\nerror CS0030: Cannot convert type 'SomeType[]' to 'SomeType' error CS0030 ...**

Being unfamiliar with this service, first thing I did was to use a separate soap client application and check if that works and also look at some raw requests/responses. I used the WcfTestClient, that's the .net tool and SoapUI app which I found recently.

The first hint that this might be a proxy issue came when the WcfTestClient failed with the same error while SoapUI managed to build and execute requests just fine.

Next turning to google and stackoverflow I found a couple of references to similar issues. Looks like there is a limitation in the .net proxy generation
that is unable to correctly generate types in certain cases when the service schema contains unbounded elements.

Here's the best reference for the issue I found on a Microsoft site: [link](https://social.msdn.microsoft.com/Forums/en-US/e33305c3-b5f6-4922-8a3f-df202088d25a/unable-to-generate-temporary-classes-with-biztalk2006-published-webservices?forum=asmxandxml)

Moving on to the possible fixes, here are a couple of paths to explore.

## Option 1. Soap calls with plain http request/response

It is possible to make soap calls using plain http request/response without involving WCF. For small services this could work fine. This is a sample to get started:

```
public static void Execute()
{
    HttpWebRequest request = CreateWebRequest();
    XmlDocument soapEnvelopeXml = new XmlDocument();
    soapEnvelopeXml.LoadXml(@"<?xml version=""1.0"" encoding=""utf-8""?>
        <soap:Envelope
          xmlns:xsi=""http://www.w3.org/2001/XMLSchema-instance""
          xmlns:xsd=""http://www.w3.org/2001/XMLSchema""
          xmlns:soap=""http://schemas.xmlsoap.org/soap/envelope/"">
          <soap:Body>
            <HelloWorld xmlns=""http://tempuri.org/"" />
          </soap:Body>
        </soap:Envelope>");

    using (Stream stream = request.GetRequestStream())
    {
        soapEnvelopeXml.Save(stream);
    }

    using (WebResponse response = request.GetResponse())
    {
        using (StreamReader rd = new StreamReader(response.GetResponseStream()))
        {
            string soapResult = rd.ReadToEnd();
            Console.WriteLine(soapResult);
        }
    }
}

public static HttpWebRequest CreateWebRequest()
{
    HttpWebRequest webRequest = (HttpWebRequest)WebRequest
      .Create(@"http://localhost:56405/WebService1.asmx?op=HelloWorld");
    webRequest.Headers.Add(@"SOAP:Action");
    webRequest.ContentType = "text/xml;charset=\"utf-8\"";
    webRequest.Accept = "text/xml";
    webRequest.Method = "POST";
    return webRequest;
}
}
```
The code snippet is taken from stackoverflow from [this thread](http://stackoverflow.com/questions/4791794/client-to-send-soap-request-and-received-response)


To retain the ability to use classes versus strings and not worry too much about serialization look into options to customize the XmlSerializer.
For example a useful feature is the ability to add custom namespaces to the xml elements.

```
[XmlRoot("Node", Namespace="http://flibble")]
public class MyType {
    [XmlElement("childNode")]
    public string Value { get; set; }
}

static class Program
{
    static void Main()
    {
        XmlSerializerNamespaces ns = new XmlSerializerNamespaces();
        ns.Add("myNamespace", "http://flibble");
        XmlSerializer xser = new XmlSerializer(typeof(MyType));
        xser.Serialize(Console.Out, new MyType(), ns);
    }
}
```
The code snippet is taken from stackoverflow from [this thread](http://stackoverflow.com/questions/2339782/xml-serialization-and-namespace-prefixes)


## Option 2. Generate the service proxy manually

This solution could be suitable if the soap service only has a couple of types and methods, so nothing very complex.

If you have generated proxies manually in the past and still remember how to do it should be an easy task, just add the right attributes to the classes.

I haven't used soap in a while so I resorted to the following trick. The XML Schema definition Tool (xsd.exe) also used by visual studio to generate the service proxy has a few hidden options when executed manually among these is the ability to create and XSD from an XML.

Take a concrete request xml that is representative and generate the xsd schema, then use the xsd schema to generate the proxy classes. Then tweak the result manually if necessary.

```
xsd.exe myfile.xml //results in myfile.xsd
xsd.exe myfile.xsd //results in proxy classes
```

This approach can also serve a bit as a diagnostics tool, this is how I got the definitive confirmation that my issue was in fact causes by incorrectly generated types in the proxy.

## Option 3. Adjust the xml schema

If you own both the service and the client, the schema can be tweaked to prevent the issues with the proxy, but this scenario not likely. I've never seen such issue when both the service and client are generated with .net tools. The fix involves inserting dummy xml attributes after elements that are "unbounded".

Here is the article mentioning this fix: [link](https://social.msdn.microsoft.com/Forums/en-US/e33305c3-b5f6-4922-8a3f-df202088d25a/unable-to-generate-temporary-classes-with-biztalk2006-published-webservices?forum=asmxandxml)

```
change the following:
<xs:sequence>
  <xs:element maxOccurs="unbounded"/>
<xs:sequence>

into:
<xs:sequence>
  <xs:element maxOccurs="unbounded"/>
<xs:sequence>
<xs:attribute name="tmp" type="xs:string" />
```

## Option 4. Fix the generated code manually

After identifying the cause and weighing the possible fixes you might come to the conclusion that it's easier to just fix the generated proxy code manually and document the issue. This is what I ended up doing as well. It can be as simple as "find [][], replace with []" on the types that cause the issue.
