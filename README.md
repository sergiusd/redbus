# Reliable Easy Data BUS

[![License](https://img.shields.io/badge/license-MIT-green)](https://github.com/sergiusd/redbus/blob/master/LICENSE)

<img src="./doc/logo.jpeg" height="347"/>

RED Bus allows you to publish messages and process them with control over the result of processing. You can set a retry
strategy for each consumer and the number of retries after which a message will be marked as failed by that consumer.
The administrator can start reprocessing of unsuccessfully processed messages after fixing the problem through the
web interface.

## Issue

When you use messages in your system to maintain eventual consistency, it's important that every message is processed, 
and you must handle errors and implement a retry algorithm in every service in your system. You also need a tool to 
view failed messages and the ability to reprocess on demand.

## Resolve

Produce messages from anyway and process it with repeat retries.  
RED Bus will do the rest for you.

```plantuml
@startuml

!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

LAYOUT_TOP_DOWN()
title Publish and consume messages

Boundary(app, "Your application"){
    Boundary(appProducer, "Some service with publisher"){
        System_Ext(publisher, "Producer", "Produce message")
    }
    Boundary(appConsumer1, "Some service with consumer N"){
        System_Ext(consumer11, "Consumer N.1", "Consume and process message")
        System_Ext(consumer12, "Consumer N.M", "Consume and process message")
    }
}
Boundary(redbus, "RED Bus"){
    Person(human, "Administrator")
    System(databus, "DataBus", "Send message to MQ. Store MQ consumers, send message to consumers, and receive processing results.", "Go")
    System(repeater, "Repeater", "Reprocesses failed messages by consumers according to the retry strategy.", "Go")
    System(admin, "Admin panel", "Viewing the actual status and managing the retry system.", "Vue")
}

BiRel_L(databus, repeater, "Save failed message and process them")
BiRel_D(repeater, consumer11, "Process message", "GRPC")
BiRel_D(repeater, consumer12, "Process message", "GRPC")
Rel_R(human, admin, "View and manage")
Rel_Down(admin, databus, "Get data")

Rel(publisher, databus, "Publish message", "GRPC")
BiRel_D(databus, consumer11, "Consume message", "GRPC")
BiRel_D(databus, consumer12, "Consume message", "GRPC")

@enduml
```

## Build

```shell
    make build
```

## How to try

1. Start essential environment

   ```shell
       docker-compose -f example/docker-compose.yml up   
   ```

2. Run data bus service

   ```shell
       ./bin/databus
   ```

3. Try consumer and producer

   - [GoLang client](./example/golang/README.md)
   - [Scala client](./example/scala/README.md) (isn't working yet)