prombbolt [![Build Status](https://travis-ci.org/prysmaticlabs/prombbolt.svg?branch=master)](https://travis-ci.org/prysmaticlabs/prombbolt) [![GoDoc](http://godoc.org/github.com/prysmaticlabs/prombbolt?status.svg)](http://godoc.org/github.com/prysmaticlabs/prombbolt) [![Report Card](https://goreportcard.com/badge/github.com/prysmaticlabs/prombbolt)](https://goreportcard.com/report/github.com/prysmaticlabs/prombbolt)
====

Package `prombbolt` provides a [Prometheus](https://prometheus.io/) metrics
collector for [bbolt](https://github.com/etcd-io/bbolt) databases.
MIT Licensed.

Forked from https://github.com/mdlayher/prombolt to use the maintained bbolt library.

Usage
-----

Instrumenting your application's Bolt database using `prombbolt` is trivial.
Simply wrap the database handle using `prombbolt.New` and register it with
Prometheus.

```go
const name = "prombbolt.db"

db, err := bolt.Open(name, 0666, nil)
if err != nil {
	log.Fatal(err)
}

// Register prombbolt handler with Prometheus
prometheus.MustRegister(prombbolt.New(name, db))

mux := http.NewServeMux()
mux.Handle("/", newHandler(db))
// Attach Prometheus metrics handler
mux.Handle("/metrics", prometheus.Handler())

http.ListenAndServe(":8080", mux)
```

At this point, Bolt metrics should be available for Prometheus to scrape from
the `/metrics` endpoint of your service.

```
$ curl -s http://localhost:8080/metrics | grep "bolt" | head -n 9
# HELP bolt_bucket_buckets Number of buckets within a bucket, including the top bucket.
# TYPE bolt_bucket_buckets gauge
bolt_bucket_buckets{bucket="foo",database="prombboltd.db"} 1
# HELP bolt_bucket_depth Number of levels in B+ tree for a bucket.
# TYPE bolt_bucket_depth gauge
bolt_bucket_depth{bucket="foo",database="prombboltd.db"} 1
# HELP bolt_bucket_inlined_buckets Number of inlined buckets for a bucket.
# TYPE bolt_bucket_inlined_buckets gauge
bolt_bucket_inlined_buckets{bucket="foo",database="prombboltd.db"} 1
```

FAQ
---

**Q: can `prombbolt` provide metrics for nested/child buckets?**

At this time, `prombbolt` is unable to retrieve metrics for nested/child buckets,
because Bolt does not currently provide functionality to iterate nested/child
buckets within a parent bucket.  See [boltdb/bolt#603](https://github.com/etcd-io/bbolt/issues/603).
