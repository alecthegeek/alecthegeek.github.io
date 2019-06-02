---
layout: post
title: "Intialisation of unnamed fields in Go lang"
date: 2019-05-28 17:02
comments: true
categories: golang, programming
---

I don't program in Go every day (the joy of being a polyglot programmer),
but when I do I often use structures with embedded types using
anonymous fields. However initialising such data structures can be confusing,
so this note is here to help me the next time I need to know -- hopefully
it will be useful for others as well.

As an aside, if you want to see a real world
[example](https://github.com/PaperCutSoftware/PaperCutExamples/blob/master/Authentication/cust-auth-sync.go)
I recently created a demo program for [PaperCut](https://papercut.com/) customers
to show how to implement a specific type of user directory integration in Go.


Before going any further you should know how Go structure intialisation works
in general, there is good overview in the online book
[An Introduction to Programming in Go](https://www.golang-book.com/books/intro/9)
by [Caleb Doxsey](https://www.doxsey.net/).

Let's introduce an example about structure intialisation using anonymous fields.

Consider a data record that represents some information about networked hosts (this example is completely made up,
I'm no pretending it's complete or realistic).

It might contain the following fields

* Network details
* Service
* Operating system
* Location

and the network details might be broken down into

*	MAC adress
*	IP address
*   Host name
*	Domain name

So let's define a couple types to represent this

```go
type  networkDetailsT struct {
	MacAddress string  `json :"mac"`
	IPaddress string `json: "ip"`
	HostName string `json: "host_name"`
	DomainName string `json: "domain_name"`
}

type  hostT_V1 struct {
	network networkDetailsT
	ServiceName string `json: "service_name"`
	OperatingSystem string `json : "operating_system"`
	Location  string `json: "location"`
}
```

So now we can access the fields of a data record as follows

```go
var myHost_V1 hostT_V1

myHost_V1.network.IPaddress = "100.10.78.9"
myHost_V1.OperatingSystem = "WinNT"
```
and we can use structure intialisation

```go
myHost_V1 = hostT_1{
	network: networkDetailsT{
		IPPaddress: "192.168.1.27"
	},
	ServiceName: "smtp"
}
```
(Notice how we use the field name of the embedded type field, `network`,
in the intialisation)

However using an anonymous field can often be much more convenient.

```go
type hostT_V2 struct {
	networkDetailsT
	ServiceName string `json: "service_name"`
	OperatingSystem string `json : "operating_system"`
	Location  string `json: "location"`
}

var myHost_V2 hostT_V2

myHost_V2.IPaddress = "100.10.78.9"
myHost_V2.OperatingSystem = "WinNT"
```
However intialisation needs a bit of extra magic because we have to
provide a field name.

```go
myHost_V2 = hostT_V2{
	networkDetailsT: networkDetailsT{
			IPPaddress: "192.168.1.27",
	},
	ServiceName: "smtp",
}
```
**We use the type name as the field name.**

This is documented in the Go spec https://golang.org/ref/spec#Struct_types,
but I can never remember the correct terms to find it.

Of course you can avoid the having to name fields in the the intialisation if you provide all the
values in the correct order

```go
myHost2 := hostT_V2{
	networkDetailsT{
		"00:25:96:12:34:99", "192.168.1.29",
		"server2",
		"example2.com",
	},
	"accounts",
	"Linux",
	"Data Centre Rack 2",
}
```

However I think it's helpful for people who are reading the code,
probably you in six months time, to include the field names.

```go
myHost2 := hostT_V2{
	networkDetailsT: networkDetailsT{
		IPaddress:  "192.168.1.27",
		HostName:   "server",
		DomainName: "example.com",
		Mac:        "00:25:96:12:34:56",
	},
	ServiceName:     "smtp",
	OperatingSystem: "WinNT",
	Location:        "Library",
}
```

Not only do we get some notational convenience
by using anonymous fields, we can get flatter json objects
when we serialise our data structures. You will notice
that the above examples have json annotations. So
if I use ``"encoding/jon"`` module when we can get nice
flat json object.

```json
 {
  "Mac": "00:25:96:12:34:56",
  "IPaddress": "192.168.1.27",
  "HostName": "server",
  "DomainName": "example.com",
  "ServiceName": "smtp",
  "OperatingSystem": "WinNT",
  "Location": "Library"
}
```

Here is a complete program that shows all this off

```go
// Show how embedded types work
package main

import (
	"encoding/json"
	"log"
)

type networkDetailsT struct {
	Mac        string `json :"mac"`
	IPaddress  string `json: "ip"`
	HostName   string `json: "host_name"`
	DomainName string `json: "domain_name"`
}

type hostT struct {
	networkDetailsT
	ServiceName     string `json: "service_name"`
	OperatingSystem string `json : "operating_system"`
	Location        string `json: "location"`
}

func main() {
	myHost := hostT{
		networkDetailsT: networkDetailsT{
			IPaddress:  "192.168.1.27",
			HostName:   "server",
			DomainName: "example.com",
			Mac:        "00:25:96:12:34:56",
		},
		ServiceName:     "smtp",
		OperatingSystem: "WinNT",
		Location:        "Library",
	}

	bytes, err := json.MarshalIndent(myHost, "", "  ")
	if err != nil {
		log.Fatalln(err)
	}

	log.Println(string(bytes))
	myHost2 := hostT{
		networkDetailsT{
			"00:25:96:12:34:99", "192.168.1.29",
			"server2",
			"example2.com",
		},
		"accounts",
		"Linux",
		"Data Centre Rack 2",
	}
	bytes, err = json.MarshalIndent(myHost2, "", "  ")
	if err != nil {
		log.Fatalln(err)
	}
	log.Println(string(bytes))
}
```





