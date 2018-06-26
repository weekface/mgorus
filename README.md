# Mongodb Hooks for [Logrus](https://github.com/sirupsen/logrus) <img src="http://i.imgur.com/hTeVwmJ.png" width="40" height="40" alt=":walrus:" class="emoji" title=":walrus:"/>

## Install

```shell
$ go get github.com/weekface/mgorus
```

## Usage

```go
package main

import (
	"github.com/sirupsen/logrus"
	"github.com/weekface/mgorus"
)

func main() {
	log := logrus.New()
	hooker, err := mgorus.NewHooker("localhost:27017", "db", "collection")
	if err == nil {
	    log.Hooks.Add(hooker)
	} else {
		fmt.Print(err)
	}

	log.WithFields(logrus.Fields{
		"name": "zhangsan",
		"age":  28,
	}).Error("Hello world!")
}
```

With authentication:

```go
package main

import (
	"github.com/sirupsen/logrus"
	"github.com/weekface/mgorus"
)

func main() {
	log := logrus.New()
	hooker, err := mgorus.NewHookerWithAuth("localhost:27017", "db", "collection", "user", "pass")
	if err == nil {
	    log.Hooks.Add(hooker)
	} else {
		fmt.Print(err)
	}

	log.WithFields(logrus.Fields{
		"name": "zhangsan",
		"age":  28,
	}).Error("Hello world!")
}
```

With a pre-existing collection

```go

package main

import (
	"crypto/tls"
	"log"
	"net"
	"time"
	
	mgo "gopkg.in/mgo.v2"

	"github.com/sirupsen/logrus"
	"github.com/weekface/mgorus"
)

func main() {

	s, err := mgo.DialWithInfo(&mgo.DialInfo{
		Addrs:    []string{"localhost:27017"},
		Timeout:  5 * time.Second,
		Database: "admin",
		Username: "",
		Password: "",
		DialServer: func(addr *mgo.ServerAddr) (net.Conn, error) {
			conn, err := tls.Dial("tcp", addr.String(), &tls.Config{InsecureSkipVerify: true})
			return conn, err
		},
	})
	if err != nil {
		log.Fatalf("can't create session: %s\n", err)
	}

	c := s.DB("db").C("collection")

	log := logrus.New()
	hooker := mgorus.NewHookerFromCollection(c)

	log.Hooks.Add(hooker)

	log.WithFields(logrus.Fields{
		"name": "zhangsan",
		"age":  28,
	}).Error("Hello world!")
}

```

## License
*MIT*
