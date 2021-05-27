raftdb
======
* from geekbang course: <分布式协议与算法实战> https://time.geekbang.org/column/intro/279
    * https://www.yuque.com/hxzhouh/lcgsuz/lu71ec
    * <基于Raft的分布式KV系统开发实战>
* first run
    * `go build` to build `raftdb` executable.
    ```
    # fxrc @ popos in ~/SourceCode/raftdb-geekbang on git:main x [23:04:38] C:2
    $ ./raftdb -id node01 ~/.raftdb

    2021-05-26T23:04:54.177-0700 [INFO]  raft: initial configuration: index=0 servers=[]
    [store] 2021/05/26 23:04:54 bootstrap needed
    2021-05-26T23:04:54.177-0700 [INFO]  raft: entering follower state: follower="Node at 127.0.0.1:8089 [Follower]" leader=
    2021/05/26 23:04:54 no join addresses set
    2021-05-26T23:04:55.628-0700 [WARN]  raft: heartbeat timeout reached, starting election: last-leader=
    2021-05-26T23:04:55.628-0700 [INFO]  raft: entering candidate state: node="Node at 127.0.0.1:8089 [Candidate]" term=2
    2021-05-26T23:04:55.634-0700 [DEBUG] raft: votes: needed=1
    2021-05-26T23:04:55.634-0700 [DEBUG] raft: vote granted: from=node01 term=2 tally=1
    2021-05-26T23:04:55.634-0700 [INFO]  raft: election won: tally=1
    2021-05-26T23:04:55.634-0700 [INFO]  raft: entering leader state: leader="Node at 127.0.0.1:8089 [Leader]"
    [store] 2021/05/26 23:04:55 waiting for up to 2m0s for application of initial logs
    2021/05/26 23:04:55 raftdb started successfully
    ```



---
raftdb is a simple distributed key value store based on the Raft consensus protocol. It can be run on Linux, OSX, and Windows.

## Running raftdb
*Building raftdb requires Go 1.9 or later. [gvm](https://github.com/moovweb/gvm) is a great tool for installing and managing your versions of Go.*

Starting and running a raftdb cluster is easy. Download raftdb like so:
```bash
go get github.com/hanj4096/raftdb
```

Build raftdb like so:
```bash
cd $GOPATH/src/github.com/hanj4096/raftdb
go install
```

### Bring up a cluster
Run your first raftdb node like so:
```bash
$GOPATH/bin/raftdb -id node01  -haddr raft-cluster-host01:8091 -raddr raft-cluster-host01:8089 ~/.raftdb
```

Let's bring up 2 more nodes, so we have a 3-node cluster. That way we can tolerate the failure of 1 node:
```bash
$GOPATH/bin/raftdb -id node02 -haddr raft-cluster-host02:8091 -raddr raft-cluster-host02:8089 -join raft-cluster-host01:8091 ~/.raftdb

$GOPATH/bin/raftdb -id node03 -haddr raft-cluster-host03:8091 -raddr raft-cluster-host03:8089 -join raft-cluster-host01:8091 ~/.raftdb
```

## Reading and writing keys
You can now set a key and read its value back:
```bash
curl -X POST raft-cluster-host01:8091/key -d '{"foo": "bar"}' -L
curl -X GET raft-cluster-host01:8091/key/foo -L
```

You can now delete a key and its value:
```bash
curl -X DELETE raft-cluster-host02:8091/key/foo -L
```

### Three read consistency level
You can now read the key's value by different read consistency level:
```bash
curl -X GET raft-cluster-host02:8091/key/foo?level=stale
curl -X GET raft-cluster-host02:8091/key/foo?level=default  -L
curl -X GET raft-cluster-host02:8091/key/foo?level=consistent  -L
```


