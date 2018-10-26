# AWS SQS Worker using Goroutine
`sqs-worker-go` is a package that makes consuming SQS messages easier. It supports message deletion after process and backup to a sepcified Firehose stream.

## Installing

Simply include the package in your code

```
import (
	"github.com/bufferapp/sqs-worker-go/worker"
)
```

### Using `go get`
Simply run this command to install the package and its underlying packages to your Go environment.
`go get -u github.com/bufferapp/sqs-worker-go/worker`

## Using `sqs-worker-go`
It's a simple create and start, then you are all set.

```
// Creating a new worker service
w, err := worker.NewService("your sqs queue name")
	if err != nil {
		log.Printf("Error creating new Worker Service: %s\n", err)
	}

// Start the worker
// Process is the function you would like to use to process each individual
// SQS message
w.Start(worker.HandlerFunc(Process))

// Start the worker with backup option
w.Backup("your backup firehose name").Start(worker.HandlerFunc(Process))
```
Please noe `Process` function should have the signature of `func(msg *sqs.Message) error`, while msg is the message for processing.

### Examples
* [elasticserach-indexer-worker](https://github.com/bufferapp/elasticserach-indexer-worker/blob/master/main.go)

### More Options
To control number of goroutine (`sqs-worker-go` will assign each message to a new goroutine), we could control the number of SQS messages for each batch. The option is available through exported variables.

```
// Assume worker service is created as w
w.MaxNumberOfMessage = 20 // Default is 10 messages
// Wait time for each poll
w.WaitTimeSecond = 30 // Default is 20 seconds
```

### Invalid Messages
Sometimes invalid messages will end up in the SQS and can't be processed properly. We may want to delete the invalid message as soon as we tried to process it, so it doesn't stay in the queue and get picked up again. For that effect, you may throw an `InvalidMessageError` from your message processor to worker.

```
return nil, worker.NewInvalidMessageError("invalid message", "Not a supported message type")
```

It's important to note, for other runtime errors, `sqs-worker-go` will `NOT` delete the message so we could try another time.
