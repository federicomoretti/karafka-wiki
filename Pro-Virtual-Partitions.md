Virtual Partitions allow you to parallelize the processing of data from a single partition. This can drastically increase throughput when IO operations are involved.

While the default scaling strategy for Kafka consumers is to increase partitions count and number of consumers, in many cases, this will not provide you with desired effects. In the end, you cannot go with this strategy beyond assigning one process per single topic partition. That means that without a way to parallelize the work further, IO may become your biggest bottleneck.

Virtual Partitions solve this problem by providing you with the means to further parallelize work by creating "virtual" partitions that will operate independently but will, as a collective processing unit, obey all the Kafka warranties.

<p align="center">
  <img src="https://raw.githubusercontent.com/karafka/misc/master/stats/virtual_partitions_performance.png" />
</p>
<p align="center">
  <small>*This example illustrates the throughput difference for IO intense work, where the IO cost of processing a single message is 1ms.
  </small>
</p>

## Using virtual partitions

The only thing you need to add to your setup is the `virtual_partitions` definition for topics for which you want to enable it:

```ruby
class KarafkaApp < Karafka::App
  setup do |config|
    # ...
  end

  routes.draw do
    topic :orders_states do
      consumer OrdersStatesConsumer

      # Distribute work to virtual partitions per order id
      virtual_partitions(
        partitioner: ->(message) { message.headers['order_id'] },
        # Defines how many concurrent virtual partitions will be created for this
        # topic partition. When not specified, Karafka global concurrency setting
        # will be used to make sure to accommodate as many worker threads as possible.
        max_partitions: 5
      )
    end
  end
end
```

No other changes are needed.

The virtual `partitioner` requires to respond to a `#call` method, and it accepts a single Karafka message as an argument.

The return value of this partitioner needs to classify messages that should be grouped uniquely. We recommend using simple types like strings or integers.

## Messages distribution

Message distribution is based on the outcome of the `virtual_partitions` settings. Karafka will make sure to distribute work into jobs with a similar number of messages in them (as long as possible). It will also take into consideration the current `concurrency` setting and the `max_partitions` setting defined within the `virtual_partitions` method.

Below is a diagram illustrating an example partitioning flow of a single partition data. Each job will be picked by a separate worker and executed in parallel (or concurrently when IO is involved).

<p align="center">
  <img src="https://raw.githubusercontent.com/karafka/misc/master/charts/virtual_partitions_partitioner.svg" />
</p>

### Partitioning based on the message key

Suppose you already use message keys to direct messages to partitions automatically. In that case, you can use those keys to distribute work to virtual partitions without any risks of distributing data incorrectly (splitting dependent data to different virtual partitions):

```ruby
routes.draw do
  topic :orders_states do
    consumer OrdersStatesConsumer

    # Distribute work to virtual partitions based on the message key
    virtual_partitions(
      partitioner: ->(message) { message.key }
    )
  end
end
```

### Partitioning based on the message payload

Since the virtual partitioner accepts the message as the argument, you can use both `#raw_payload` as well as `#payload` to compute your distribution key:

```ruby
routes.draw do
  topic :orders_states do
    consumer OrdersStatesConsumer

    # Distribute work to virtual partitions based on the user id, ensuring,
    # that per user, everything is in order
    virtual_partitions(
      partitioner: ->(message) { message.payload.fetch('user_id') }
    )
  end
end
```

**Note**: Keep in mind that Karafka provides [lazy deserialization](https://github.com/karafka/karafka/wiki/Deserialization#lazy-deserialization). If you decide to use payload data, deserialization will happen in the main thread before the processing. That is why, unless needed, it is not recommended.

### Partitioning randomly

If your messages are independent, you can distribute them randomly by running `rand(Karafka::App.config.concurrency)` for even work distribution:

```ruby
routes.draw do
  topic :orders_states do
    consumer OrdersStatesConsumer

    # Distribute work to virtual partitions based on the user id, ensuring,
    # that per user, everything is in order
    virtual_partitions(
      partitioner: ->(_) { rand(Karafka::App.config.concurrency) }
    )
  end
end
```

## Managing number of Virtual Partitions

By default, Karafka will create at most `Karafka::App.config.concurrency` concurrent Virtual Partitions. This approach allows Karafka to occupy all the threads under optimal conditions.

### Limiting number of Virtual Partitions

However, it also means that other topics may not get their fair share of resources. To mitigate this, you may dedicate only 80% of the available threads to Virtual Partitions.

```ruby
setup do |config|
  config.concurrency = 10
end

routes.draw do
  topic :orders_states do
    consumer OrdersStatesConsumer

    virtual_partitions(
      partitioner: ->(message) { message.payload.fetch('user_id') },
      # Leave two threads for other work of other topics partitions
      # (non VP or VP of other partitions)
      max_partitions: 8
    )
  end
end
```

**Note**: Virtual Partitions `max_partitions` setting applies per topic partition. In the case of processing multiple partitions, there may be a case where all the work happens on behalf of Virtual Partitions.

### Increasing number of Virtual Partitions

There are specific scenarios where you may be interested in having more Virtual Partitions than threads. One example would be to create one Virtual Partition for the data of each user. If you set the `max_partitions` to match the `max_messages`, Karafka will create each Virtual Partition based on your grouping without reducing it to match number of worker threads.

```ruby
setup do |config|
  config.concurrency = 10
  config.max_messages = 200
end

routes.draw do
  topic :orders_states do
    consumer OrdersStatesConsumer

    virtual_partitions(
      partitioner: ->(message) { message.payload.fetch('user_id') },
      # Make sure, that each virtual partition always contains data of only a single user
      max_partitions: 200
    )
  end
end
```

**Note**: Please remember that Virtual Partitions are long-lived and will stay in the memory for as long as the Karafka process owns the given partition.

## Behaviour on errors

For a single partition-based Virtual Partitions group, offset management and retries policies are entangled. They behave [on errors](Error-handling-and-back-off-policy#runtime) precisely the same way as regular partitions with one difference: back-offs and retries are applied to the underlying regular partition. This means that if an error occurs in one of the virtual partitions, Karafka will pause based on the first offset received from the regular partition.

<p align="center">
  <img src="https://raw.githubusercontent.com/karafka/misc/master/charts/virtual_partitions_error_handling.svg" />
</p>

If processing in all virtual partitions ends up successfully, Karafka will mark the last message from the underlying partition as consumed.

**Note**: Since pausing happens in Kafka, the re-fetched data may contain more or fewer messages. This means that after retry, the number of messages and their partition distribution may differ. Despite that, all ordering warranties will be maintained.

### Collapsing

When an error occurs in virtual partitions, pause, retry and collapse will occur. Collapsing allows virtual partitions to temporarily restore all the Kafka ordering warranties allowing for the usage of things like offset marking and Dead-Letter Queue.

You can detect that your Virtual Partitions consumers are operating in the collapsed mode by invoking the `#collapsed?` method:

```ruby
class EventsConsumer < ApplicationConsumer
  def consume
    messages.each do |message|
      Event.store!(message.payload)

      # When consumer operates in a collective mode, strong Kafka
      # ordering warranties apply throughout all the messages
      mark_as_consumed(message) if collapsed?
    end
  end
end
```

<p align="center">
  <img src="https://raw.githubusercontent.com/karafka/misc/master/charts/virtual_partitions_collapse.svg" />
</p>
<p align="center">
  <small>*This example illustrates the retry and collapse of two virtual partitions into one upon errors.
  </small>
</p>


### Usage with Dead Letter Queue

Virtual Partitions can be used together with the Dead Letter Queue. This can be done due to Virtual Partitions' ability to collapse upon errors.

The only limitation when combining Virtual Partitions with the Dead Letter Queue is the minimum number of retries. It needs to be set to at least `1`:

```ruby
routes.draw do
  topic :orders_states do
    consumer OrdersStatesConsumer
    virtual_partitions(
      partitioner: ->(message) { message.headers['order_id'] }
    )
    dead_letter_queue(
      topic: 'dead_messages',
      # Minimum one retry because VPs needs to switch to the collapsed mode
      max_retries: 1
    )
  end
end
```

## Ordering warranties

Virtual Partitions provide three types of warranties in regards to order:

- Standard warranties per virtual partitions group - that is, from the "outside" of the virtual partitions group Kafka ordering warranties are preserved.
- Inside each virtual partition - the partitioner order is always preserved. That is, offsets may not be continuous (1, 2, 3, 4), but lower offsets will always precede larger (1, 2, 4, 9). This depends on the `virtual_partitions` `partitioner` used for partitioning a given topic.
- Strong Kafka ordering warranties when operating in the `collapsed` mode.

<p align="center">
  <img src="https://raw.githubusercontent.com/karafka/misc/master/charts/virtual_partitions_order.svg" />
</p>
<p align="center">
  <small>*Example distribution of messages in between two virtual partitions.
  </small>
</p>

## Monitoring

Karafka default [DataDog/StatsD](Monitoring-and-logging#datadog-and-statsd-integration) monitor and dashboard work with virtual partitions out of the box. No changes are needed. Virtual batches are reported as they would be regular batches.

## Manual offset management

Manual offset management as well as checkpointing during virtual partitions execution is **not** recommended. Virtual Partitions group order is not deterministic, which means that if you mark the message as processed from a virtual batch, it may not mean that messages with earlier offset from a different virtual partition were processed.

## Shutdown and revocation handlers

Both `#shutdown` and `#revoked` handlers work the same as within [regular consumers](Consuming-messages#shutdown-and-partition-revocation-handlers).

For each virtual consumer instance, both are executed when shutdown or revocation occurs. Please keep in mind that those are executed for **each** instance. That is, upon shutdown, if you used ten threads and they were all used with virtual partitions, the `#shutdown` method will be called ten times. Once per each virtual consumer instance that was in use.

<p align="center">
  <img src="https://raw.githubusercontent.com/karafka/misc/master/charts/virtual_partitions_shutdown.svg" />
</p>

## Customizing the partitioning engine / Load aware partitioning

There are scenarios upon which you can differentiate your partitioning strategy based on the number of received messages per topic partition. It is impossible to set it easily using the default partitioning API, as this partitioner accepts single messages. However, Pro users can use the `Karafka::Pro::Processing::Partitioner` as a base for a custom partitioner that can achieve something like this.

One great example of this is a scenario where you may want to partition messages in such a way as to always end up with at most `5 000` messages in a single Virtual Partition.

```ruby
# This is a whole process partitioner, not a per topic one
class CustomPartitioner < Karafka::Pro::Processing::Partitioner
  def call(topic_name, messages, coordinator)
    # Apply the "special" strategy for this special topic unless VPs were collapsed
    # In the case of collapse you want to process with the default flow.
    if topic_name == 'balanced_topic' && !coordinator.collapsed?
      balanced_strategy(messages)
    else
      # Apply standard behaviours to other topics
      super
    end
  end

  private

  # Make sure you end up with virtual partitions that always have at most 5 000 messages and create
  # as few partitions as possible
  def balanced_strategy(messages)
    messages.each_slice(5_000).with_index do |slice, index|
      yield(index, slice)
    end
  end
end
```

Once you create your custom partitioner, you need to overwrite the default one in your configuration:

```ruby
class KarafkaApp < Karafka::App
  setup do |config|
    config.internal.processing.partitioner_class = CustomPartitioner
  end
end
```

When used that way, your `balanced_topic` will not use the per topic `partitioner` nor `max_partitions`. This topic data distribution will solely rely on your `balanced_strategy` logic.

## Example use-cases

Here are some use cases from various industries where Karafka's virtual partition feature can be beneficial:

- Adtech: An ad-tech company may need to process a large number of ad impressions or clicks coming in from a single Kafka topic partition. By using virtual partitions to parallelize the processing of these events, they can improve the efficiency and speed of their ad-serving system, which often involves database operations.

- E-commerce: In the e-commerce industry, processing many product orders or inventory updates can be IO bound. By using virtual partitions to parallelize the processing of these events, e-commerce companies can improve the efficiency and speed of their systems, enabling them to serve more customers and update inventory more quickly.

- Logistics: A logistics company may need to process a large volume of shipment or tracking data coming in from a single Kafka topic partition. By using virtual partitions to parallelize the processing of this data, they can improve the efficiency of their logistics operations and reduce delivery times.

- Healthcare: In healthcare, processing a large volume of patient data can be IO bound, particularly when interacting with electronic health records (EHR) or other databases. By using virtual partitions to parallelize the processing of patient data coming from a single Kafka topic partition, healthcare organizations can improve the efficiency of their data analysis and provide more timely care to patients.

- Social Media: Social media platforms often need to process many user interactions, such as likes, comments, and shares, coming in from a single Kafka topic partition. By using virtual partitions to parallelize the processing of these events, they can improve the responsiveness of their platform and enhance the user experience.

Overall, virtual partitions can be beneficial in any industry where large volumes of data need to be processed quickly and efficiently, particularly when processing is IO bound. By parallelizing the processing of data from a single Kafka topic partition, organizations can improve the performance and scalability of their systems, enabling them to make more informed decisions and deliver better results.
