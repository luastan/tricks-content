---
title: Fancy Stuff
description: Cool tricks
badge: Go
position: 3
---


## Embed files into the executable

Assets such as templates, images or scripts can be embedded to the go binary using the [embed package](https://pkg.go.dev/embed) from the standard library.

Basic usage is the following:

```go
import _ "embed"

//go:embed {{ embedded-filename hello.txt }}
var s string

print(s)
```

You can also create a _virtual filesystem_ with multiple embeded:

```go
package main

import (
	"embed"
	"log"
	"net/http"
)

//go:embed internal/embedtest/testdata/*.txt
var content embed.FS

func main() {
	mux := http.NewServeMux()
	mux.Handle("/", http.FileServer(http.FS(content)))
	err := http.ListenAndServe(":8080", mux)
	if err != nil {
		log.Fatal(err)
	}
}
```
