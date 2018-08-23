# Go

… or Golang. A programming language made to do work.

## Good Go Code

Check out [Effective Go](https://golang.org/doc/effective_go.html), [Arne's Styleguide](https://github.com/bahlo/go-styleguide), [Idiomatic Go](https://about.sourcegraph.com/go/idiomatic-go/) and [Writing Great Go Code](https://scene-si.org/2018/07/24/writing-great-go-code/). 
Some of the (imho) most important takeaways from the latter:

* Think about your packages, where to split things and where _not_ to split them. Don't overcomplicate.
* Use [pkg/errors](https://github.com/pkg/errors) and `errors.Wrap`, not the standard library.
* Use a logger package like [sirupsen/logrus](https://github.com/sirupsen/logrus) or [apex/log](https://github.com/apex/log). Try adding the file and line number to each log message (e.g. with `log.SetFlags(log.LstdFlags | log.Lshortfile)`).
* If you want to make sure that one of your types implements an interface, you can do something like `var _ io.Reader = &YourStruct{}`, which will cause an error if it doesn't implement the interface, but will be optimized away during compilation if everything's fine.

## Accessing the Mumble Link API for live Guild Wars 2 game data

GW2 exposes real-time per-frame character and camera position data by using [Mumble's Link API](https://wiki.mumble.info/wiki/Link). 
There's a [description on the GW2 Wiki](https://wiki.guildwars2.com/wiki/API:MumbleLink) on what's available, but it's very focused on C code.

I did some work on creating proof-of-concept quality code in Go. 
The challenges have been accessing the API using Windows's File Mapping and parsing the data structure inside. 
This is what I've built:

```go
package main

import (
	"fmt"
	"log"
	"syscall"
	"time"
	"unsafe"
	"unicode/utf16"
	"unicode/utf8"
	"bytes"
)

type MumbleVector [3]float32
type WChar uint16

type MumbleIdentity [256]uint16
func (id MumbleIdentity) String() string {
	buf := make([]byte, 4)
	var ret bytes.Buffer
	runes := utf16.Decode(id[:])
	count := 0
	for _, rune := range runes {
		utf8.EncodeRune(buf, rune)
		ret.WriteString(string(rune))
		count++
	}
	return ret.String()
}

type MumblePosition struct {
	Position MumbleVector
	Front MumbleVector
	Top MumbleVector
}

type MumbleData struct {
	Version uint32
	Tick uint32
	Avatar MumblePosition
	Name [256]WChar
	Camera MumblePosition
	Identity MumbleIdentity
	ContextLength uint32
	Context [256]byte
	Description [2048]WChar
}

func main() {
	file, _ := syscall.UTF16PtrFromString("MumbleLink")
	size := 100000 // I’ve tried unsafe.Sizeof(MumbleData{}) but that didn’t work.
	handle, err := syscall.CreateFileMapping(0, nil, syscall.PAGE_READWRITE, 0, uint32(size), file)
	if err != nil {
		log.Fatal(err)
	}
	defer syscall.CloseHandle(handle)
	addr, err := syscall.MapViewOfFile(handle, syscall.FILE_MAP_READ, 0, 0, 0)
	if err != nil {
		log.Fatal(err)
	}
	for {
		data := (*MumbleData)(unsafe.Pointer(addr))
		time.Sleep(1 * time.Second)
		fmt.Printf("ava %v cam %v id %v\n", data.Avatar.Position, data.Camera, data.Identity)
	}
}
```

It's not great Go code, but it works.

### Resources I've used

* [Windows DLLs](https://github.com/golang/go/wiki/WindowsDLLs) on the Go Wiki
* [`mumble.py`](https://github.com/TheTerrasque/gw2lib/blob/master/gw2lib/mumble.py) from [TheTerrasque/gw2lib](https://github.com/TheTerrasque/gw2lib)
* [`MumbleLink.cs`](https://github.com/cvpcs/GuildWars2/blob/master/GuildWars2.ArenaNet.MumbleLink/MumbleLink.cs) from [cvpcs/GuildWars2](https://github.com/cvpcs/GuildWars2)
* [`mmap_windows.go`](https://github.com/etsy/hound/blob/master/codesearch/index/mmap_windows.go) from [etsy/hound](https://github.com/etsy/hound)
* some other questions/answers and examples regarding Python, Java and MumbleLink that are already dead links
