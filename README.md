Copy files over SCP with Go
=============================
[![Go Report Card](https://goreportcard.com/badge/bramvdbogaerde/go-scp)](https://goreportcard.com/report/bramvdbogaerde/go-scp) [![](https://godoc.org/github.com/bramvdbogaerde/go-scp?status.svg)](https://godoc.org/github.com/bramvdbogaerde/go-scp)

This is a fork of Bram Vandenbogaerde's excellent go-scp module.
It adds a proxy io.Reader and io.Writer that could very well replace the more limited PassThru Factory.
The io.Reader proxy is added to the copyFromRemote, and intercepts the stream coming from the remote, providing a more accurate progress from what is downloaded (provided download is slower than writing to disk)
The io.Writer proxy is added to the CopyPassThru, and intercepts the stream going to the remote, providing a more accurate progress from what is uploaded (provided upload is slower than reading to disk)  The proxies could easily be extended to also intercept the local reading/writing, rendering PassThru completely obsolete.  But since it wasn't a requirement of this fork, it was left undone.

This package makes it very easy to copy files over scp in Go.
It uses the golang.org/x/crypto/ssh package to establish a secure connection to a remote server in order to copy the files via the SCP protocol.

### Example usage


```go
package main

import (
	"fmt"
	scp "github.com/bramvdbogaerde/go-scp"
	"github.com/bramvdbogaerde/go-scp/auth"
	"golang.org/x/crypto/ssh"
	"os"
        "context"
)

func main() {
	// Use SSH key authentication from the auth package
	// we ignore the host key in this example, please change this if you use this library
	clientConfig, _ := auth.PrivateKey("username", "/path/to/rsa/key", ssh.InsecureIgnoreHostKey())

	// For other authentication methods see ssh.ClientConfig and ssh.AuthMethod

	// Create a new SCP client
	client := scp.NewClient("example.com:22", &clientConfig)

	// Connect to the remote server
	err := client.Connect()
	if err != nil {
		fmt.Println("Couldn't establish a connection to the remote server ", err)
		return
	}

	// Open a file
	f, _ := os.Open("/path/to/local/file")

	// Close client connection after the file has been copied
	defer client.Close()

	// Close the file after it has been copied
	defer f.Close()

	// Finally, copy the file over
	// Usage: CopyFromFile(context, file, remotePath, permission)

        // the context can be adjusted to provide time-outs or inherit from other contexts if this is embedded in a larger application.
	err = client.CopyFromFile(context.Background(), *f, "/home/server/test.txt", "0655")

	if err != nil {
		fmt.Println("Error while copying file ", err)
	}
}
```

#### Using an existing SSH connection

If you have an existing established SSH connection, you can use that instead.

```go
func connectSSH() *ssh.Client {
   // setup SSH connection
}

func main() {
   sshClient := connectSSH()

   // Create a new SCP client, note that this function might
   // return an error, as a new SSH session is established using the existing connecton

   client, err := scp.NewClientBySSH(sshClient)
   if err != nil {
      fmt.Println("Error creating new SSH session from existing connection", err)
   }

   /* .. same as above .. */
}
```

#### Copying Files from Remote Server

It is also possible to copy remote files using this library. 
The usage is similar to the example at the top of this section, except that `CopyFromRemote` needsto be used instead.

For a more comprehensive example, please consult the `TestDownloadFile` function in t he `tests/basic_test.go` file.

### Using the Proxy reader or writer

It is as simple as providing an io.ReadCloser or io.WriteCloser as the last parameter (can be nil when not using this functionality)

### License

This library is licensed under the Mozilla Public License 2.0.    
A copy of the license is provided in the `LICENSE.txt` file.

Copyright (c) 2020 Bram Vandenbogaerde

Fork by Jean-Pierre Doyon for Newtrax/Sandvik