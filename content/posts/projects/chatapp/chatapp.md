---
title: Multi-threaded Pub/Sub Chat App
summary: "A chat application that utilizes a multi-threaded pub/sub message queue, ring buffer, and linked list built in C and a server written in Golang."
date: 2022-10-17T23:04:49-04:00
url: ""
tags: ["Multi-threading", "Golang", "C", "Linked List", "Ring Buffer"]
draft: false
showtoc: true
cover:
    image: "/posts/projects/chatapp/pictures/libarch.png"
    alt: "picture"
    caption: ""
---

### Tech Stack

_Golang, C, and NCurses_

# What is it?

On a high level, this project is a CLI chat application that utilizes multi-threaded message queues, ring buffers, & linked lists so users can talk to each other on different channels (like [Discord](https://discord.com)). This project initially started as an assignment for my [Operating Systems course](https://www3.nd.edu/~pbui/teaching/cse.30341.fa22/project02.html) at Notre Dame, but I was so intrigued by it that I decided to keep building on it.

![img](/posts/projects/chatapp/pictures/new.png)

## Features

- Ability to subscribe and switch to different channels (sports, cooking, memes, etc.)
- Asynchronous I/O with threads and an event loop
- Cache history for every channel (with a configurable buffer size)
- Colorful GUI with NCurses
- Highlighted message when user is mentioned

# How does it work?

On a lower level, I had to build three things for this project: a multi-threaded pub/sub library, a chat application that utilized the library, and a server that could handle asynchronous I/O.

## Library

![library](/posts/projects/chatapp/pictures/libarch.png)

The purpose of the library is to _abstract_ all of the threading logic needed for the application. Since there are multiple threads accessing the same data (queues), there is a need to _protect_ the application from data races and deadlocks. Instead of putting this burden on the user trying to use the application, the user can just use the library apis which hides all of that logic. I personally decided to use [**semaphores**](<https://en.wikipedia.org/wiki/Semaphore_(programming)>) instead of mutexes/locks to ensure thread-safe code.

For example, a user can just simply call `queue_pop(q)` without having to worry about threading, and under the hood the library would handle it like this.

```c
Request* queue_pop(Queue *q) {
  sem_wait(&q->produced);
  sem_wait(&q->lock);
  Request *curr = q->head;
  q->head = queue->head->next;
  q->size--;
  sem_post(&q->lock);
  return curr;
}
```

The library is written in C, and it contains 2 queues: one **outgoing** that contains all of the client's requests (subscribing to a topic, sending a message, etc.) and an **incoming** that contains any messages for the client. These queues are implemented as Linked Lists, and the pusher thread will pop from outgoing and send to the server while the puller thread will get messages from the server and push it to the incoming.

## Chat App

The chat app was actually quite the headache to code (thanks NCurses). For the OS project, all we were required to do was create a working shell application that could utilize the library. However, I thought it was quite messy to have different topic messages show up on the same screen. This is why I decided to create a Discord-esque channel system where users can switch between different channels that would have their own separate history.

![channels](/posts/projects/chatapp/pictures/channels.png)

Users can type in `/subscribe {topic}` to subscribe and start receiving messages from a topic. They, however, won't be able to actually see any messages from the topic until they type `/switch {topic}` which switches channels. The user will be able to see all messages since they subscribed to the topic (if # doesn't exceed buffer limit) .

Because all of the different channels' chat history is stored on the heap, I added a configurable `MAX_MESSAGES` variable to control how far back the user wants their history to go. To achieve this, I implemented a [Linked List](https://en.wikipedia.org/wiki/Linked_list) of channels that utilize a [Ring Buffer/Circular Buffer](https://en.wikipedia.org/wiki/Circular_buffer) to store messages.

![buf3](/posts/projects/chatapp/pictures/buf5.png)
|:--:|
| _Showcase of the ring buffer working for `MAX_MESSAGES=5`_ |

This is basically whats happening under the hood for the ring buffer

``` c
// If we reached the end of the circular buffer, wrap around and remove oldest entry and update read
if (curr_chat->write >= MAX_MESSAGES) {
    curr_chat->read++;
    free(curr_chat->buffer_history[curr_chat->write % MAX_MESSAGES]);
}
curr_chat->buffer_history[curr_chat->write++ % MAX_MESSAGES] = dyn_msg
```

Some of the other features the chat app contains are highlighting if the user is mentioned (trivial to implement), "unique" but consistent username coloring (through trivial hashing algorithm), and a configurable `MAX_SUBS` to create bound for # of topics a user can subscribe to (for memory).

## Server

For this project, my professor gave us an asyncio python server that we could use. However, I thought this would be the perfect opportunity to get practice with [Golang](https://go.dev/) so I decided to rewrite the server. This was the first time that I built a non Hello-World project in Go, and it made me really appreciate the language.

The server itself isn't very complicated, it just had to be able to store which users were subscribed to what topic and be able to handle requests asynchronously. For the latter, Golang was _PERFECT_ because of the built-in Goroutines and Channels.

For example, on the client side, the user's puller thread sends `GET` requests to the server and it hangs until it gets something back. Similarly, the server should wait until there is a message for the user and then respond to the client's `GET` request. It is very important to do this in a non-blocking manner, and I was able to achieve this in Go without a library like python's `asyncio`.

``` go
// Queue Handler
func queueHandler(c *gin.Context) {
    queueName := c.Param("id");
    // Check if queue is in the system
    if _, exists := queues[queueName]; !exists {
        c.String(404, fmt.Sprintf("There is no queue named %s", queueName))
        return
    }
    // Wait until a message is ready
    func () {
        for {
        select {
            // If a message is ready, send and break
            case message := <- queues[queueName]:
                c.String(200, message)
                return
            // If there is no message, wait until one is ready
            default:
                time.Sleep(1 * time.Second)
            }
        }
    }()
}
```

In this code, the server spins up a Goroutine when a user sends a `GET` request and it simply waits until there is something in the users channel to send something back. On the other hand, this is what the code looks like to actually put something in a user's channel.

``` go
// Topic Handler
func topicHandler(c *gin.Context) {
    topic := c.Param("id")
    // Read the request body
    message, err := io.ReadAll(c.Request.Body)
    if err != nil {
        c.String(404, "Bad message")
    }
    // Send message to anyone who is subscribed to the topic
    subscribers := 0
    for queue, topics := range subscriptions {
        if _, exists := topics[topic]; exists {
            // Send the message to the queues channel
            queues[queue] <- string(message)
            subscribers++
        }
    }
    if subscribers == 0 {
        c.String(404, fmt.Sprintf("There are no subscribers for topic %s", topic))
    } else {
        c.String(200, fmt.Sprintf("Published message (%d bytes) to %d subscribers of %s", len(message), subscribers, topic))
    }
}
```

# Closing Remarks

All in all, I was really happy with this project, and I truly learned a ton about multi-threading (to always use multi-processing instead if you can, lol). Playing around with NCurses is always a pain, but I had fun trying to create a _somewhat_ enjoyable UI. Lastly, since this project, I have had a lot of fun learning more and more about Golang. It _miiight_ just be my favorite language right now! But, I am currently trying to learn Rust so we'll see how that plays out.
