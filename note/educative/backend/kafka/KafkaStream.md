# Kafka stream

## Overview

2 types of API to build stream processing:

- DSL API: high level API to build stream process. Allow filter/map/aggregate + join/window operations
- Processor API: low level API. Allow defining custom stream processing operators and connect them to a graph ->
  processing topology

## Key concepts

- Streams : Continuous sequence of records produced/consumed in real-time. Contain ordered, replayable sequence data
  records (Key-Value)
- Topology: Direct acyclic graph represents stream processing pipeline in Kafka stream apps
- Stream partition: subset of a stream

## Handle state in Kafka Stream

State in Kafka Stream is used to store results of stream processing ops

