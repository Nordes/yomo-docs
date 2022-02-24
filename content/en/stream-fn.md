---
title: Stream Functions
position: 4
category: Overview
---

## Introduction

To process continuous high-frequency data in real-time,
YoMo adopts the [Functional Reactive Programming (FRP)](https://en.wikipedia.org/wiki/Functional_reactive_programming) Paradigm and makes stream computing easier than ever.
YoMo uses the [QUIC transport protocol](https://en.wikipedia.org/wiki/QUIC),
which largely improves the speed of data transfer, and maps QUIC streams into Rx streams.
The user can then use the [stream operators](http://reactivex.io/documentation/operators.html) to process the stream as they like:

```go
func Handler(rxstream rx.Stream) rx.Stream {
	stream := rxstream.
		Subscribe(NoiseDataKey).
		Debounce(50).
		OnObserve(decode).
		Map(computePeek).
		SlidingWindowWithTime(SlidingWindowInMS, SlidingTimeInMS, slidingAvg).
		Encode(0x14)

	return stream
}
```

Besides translating the raw data into usable information,
a stream function can also display data in real-time on a web page, save the stream to a time series database, or print the nicely formatted data to `StdOut`:

```go
func Handler(rxstream rx.Stream) rx.Stream {
	stream := rxstream.
		Subscribe(0x14).
		OnObserve(callback).
		Map(store)
	return stream
}
```

A few things to note:

- The original stream sent by `zipper` is *immutable*. The `Handler` function operates on a stream by adding new items to the existing stream.
- The `Encode` operator is required in order to add items.

## Stream Operators

Most operators take a stream as input and return a new stream as output, 
so they can be chained together. Some of the operators are implemented by ReactiveX:

#### [Map](http://reactivex.io/documentation/operators/map.html) - applies a function to each item emitted by an Observable.

![Map](/flow/map.png)

#### [TakeLast](http://reactivex.io/documentation/operators/takelast.html) - takes the last n items emitted by an Observable.

![TakeLast](/flow/takeLast.png)

Some are unique to YoMo:

- `AuditTime`
- `DefaultIfEmptyWithTime`
- `Encode`
- `OnObserve`
- `RawBytes`
- `SlidingWindowWithCount`
- `SlidingWindowWithTime`
- `StdOut`
- `Subscribe`
- `ZipMultiObservers`

## Examples

1. A simple real-time noise monitoring system. [link](https://github.com/yomorun/yomo/tree/master/example/0-basic)
2. Working with multiple stream functions. [link](https://github.com/yomorun/yomo/tree/master/example/3-multi-sfn)
