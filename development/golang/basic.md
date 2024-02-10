---
title: Basic Usage
description: Golang basic usage for scripting
badge: Go
position: 1
---


## Basic cli tool

Project initialization for the <smart-variable variable="toolname" default-value="example"></smart-variable> command-line tool.

``` shell
cd "$(go env GOPATH)/src"
mkdir -p {{ repo github.com/luastan }}/{{ toolname example }}
cd {{ repo github.com/luastan }}/{{ toolname example }}
go mod init {{ repo github.com/luastan }}/{{ toolname example }}
```

### Command line options with cobra CLI

I usually implement command line options using [cobra CLI](https://cobra.dev/) with the [Cobra Generator](https://github.com/spf13/cobra-cli/blob/main/README.md) for basic boilerplate:

<smart-tabs variable="cobra-basic-vs-options" :tabs="{'basic': 'Basic', 'options': 'With options'}">
<template v-slot:basic>

``` shell
cd $(go env GOPATH)/src/{{ repo github.com/luastan }}/{{ toolname example }}
cobra-cli init
go run main.go
```

</template>
<template v-slot:options>

``` shell
cd $(go env GOPATH)/src/{{ repo github.com/luastan }}/{{ toolname example }}
cobra-cli init --author "{{ author you you@example.com }}" --license {{ license apache }}
go run main.go
```

</template>
</smart-tabs>



``` shell
cobra-cli add {{ command-name config }}
```




## Boilerplate code

Sample Go code to make a request through a Socks5 or http proxy

``` go
package main

import (
	"crypto/tls"
	"flag"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"net/url"
	"os"
)

var (
	Client           http.Client
	ErrorLogger      *log.Logger
)


// Prints command's usage
func usage() {
	_, _ = fmt.Fprintf(os.Stderr, "Usage of %s:\n", os.Args[0])
	flag.PrintDefaults()
	print("\n")
}

// HTTP client and Loggers initialization
func init() {
	Client = http.Client{}
	ErrorLogger = log.New(os.Stderr, "ERROR: ", log.Ldate|log.Ltime)
	flag.Usage = usage
	
}

func main() {
	// Flag parsing
	httpProxy := flag.String("proxy", "", "HTTP proxy to use. HTTP, HTTPs and SOCKS5 are supported")
	flag.Parse()

	// Set up HTTP Proxy
	if len(*httpProxy) > 0 {
		proxyURL, err := url.Parse(*httpProxy)
		if err != nil {
			ErrorLogger.Fatalln(err.Error())
		}
		Client.Transport = &http.Transport{
			Proxy:           http.ProxyURL(proxyURL),
			TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
		}
	} else {
		Client.Transport = &http.Transport{
			TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
		}
	}
	
	response, err := Client.Get("https://google.com")
	if err != nil {
		ErrorLogger.Fatalln(err.Error())
	}

	bodyBytes, err := ioutil.ReadAll(response.Body)
	if err != nil {
		ErrorLogger.Fatalln(err.Error())
	}

	fmt.Println(string(bodyBytes))	
}
```
