---
layout: default
---

# EdgeServe

EdgeServe is a decentralized model serving system that optimizes data movement and model placements within an cluster.

## Motivation and Design Principles
Machine learning deployments are getting more complex where models may source feature data from a variety of disparate services hosted on different devices. Examples include:
- ** Network Security . ** Packet catpure data collected from different nodes in a network may have to be combined to make global decisions about whether an attack is taking place.
- ** Systematic Stock Trading . ** Data from a variety of data streams, often sourced from different vendors, are combined to predict the movement of stock prices. 
- ** Multimodal Augmented Reality . ** Data from different sensors placed around a room, e.g., multiple video cameras, can be integrated into a single global model to make inferences about activities taking place in the room.

EdgeServe recognizes a need for a model serving system that can coordinate data from multiple data services and/or connect the results from multiple models. We have currently developed a research prototype with a Python programming interface and we encourage external usage and collaboration for feedback. 

## Installation
```bash
git clone https://github.com/swjz/EdgeServe.git
cd EdgeServe
pip3 -r requirements.txt
pip3 install -e .
```

EdgeServe depends on Apache Pulsar as its default message broker service. For how to set up an Apache Pulsar server in docker, please refer to [Apache Pulsar Doc](https://pulsar.apache.org/docs/2.11.x/getting-started-docker/).

## Usage
EdgeServe has an iterator-based Python interface designed for streaming data.
Each node can run multiple tasks at the same time, and each task can be replicated on multiple nodes as they consume the shared message queue in parallel.
### Working as a data source node
```python
from edgeserve.data_source import DataSource

node = 'pulsar://server-address:6650'

def stream():
    while True:
        yield sensor.read()

with DataSource(stream(), node, source_id='data1', topic='topic_in') as ds:
    # every next() call sends an example to the message queue
    next(ds)
```

### Working as a prediction node
```python
from edgeserve.compute import Compute

node = 'pulsar://server-address:6650'

def task(data1, data2, data3, data4):
    # actual prediction work here
    return aggregate_and_predict(data1, data2, data3, data4)

with Compute(task, node, worker_id='worker1', topic_in='topic_in', topic_out='topic_out') as compute:
    # every next() call consumes a message from the queue and runs task() on it
    # the result is sent to another message queue
    next(compute)
```

### Working as a destination node
```python
from edgeserve.materialize import Materialize

node = 'pulsar://server-address:6650'

def final_process(y_pred):
    # reformat the prediction result, materialize and/or use it for decision making
    make_decisions(y_pred)

with Materialize(final_process, node, topic='topic_out') as materialize:
    # every next() call consumes a message from prediction output queue and runs final_process() on it
    next(materialize)
```
