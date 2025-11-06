发布订阅模式：

```
package main

import (
	"strings"
	"sync"
	"time"
)

type (
	subscriber chan interface{}
	topicFunc  func(v interface{}) bool
)

type Publisher struct {
	m           sync.RWMutex
	buffer      int
	timeout     time.Duration
	subscribers map[subscriber]topicFunc
}

// 构建一个发布者对象，可以设置发布超时时间和缓存队列的长度
func NewPublisher(publishTimeout time.Duration, buffer int) *Publisher {
	return &Publisher{
		timeout:     publishTimeout,
		buffer:      buffer,
		subscribers: make(map[subscriber]topicFunc),
	}
}

// 添加一个新的订阅者，订阅全部主题
func (p *Publisher) AddSubscriber() chan interface{} {
	return p.SubscribeTopic(nil)
}

// 添加一个订阅者，订阅过滤晒选后的主题
func (p *Publisher) SubscribeTopic(topic topicFunc) chan interface{} {
	ch := make(chan interface{}, p.buffer)
	p.m.Lock()
	p.subscribers[ch] = topic
	p.m.Unlock()
	return ch
}

// 退出
func (p *Publisher) Evict(sub chan interface{}) {
	p.m.Lock()
	defer p.m.Unlock()
	delete(p.subscribers, sub)
	close(sub)
}

// 发布一个主题
func (p *Publisher) Publish(v interface{}) {
	p.m.RLock()
	defer p.m.RUnlock()

	var wg sync.WaitGroup
	for sub, topic := range p.subscribers {
		wg.Add(1)
		go p.sendTopic(sub, topic, v, &wg)
	}
	wg.Wait()
}

// 关闭发布者对象，同时关闭所有订阅者管道
func (p *Publisher) Close() {
	p.m.Lock()
	defer p.m.Unlock()
	for sub := range p.subscribers {
		close(sub)
		delete(p.subscribers, sub)
	}
}

// 发送主题。可以容忍一定的超时
func (p *Publisher) sendTopic(sub subscriber, topic topicFunc, v interface{}, wg *sync.WaitGroup) {
	//fmt.Printf("Send topic: %v -> %v\n", v, sub)
	defer wg.Done()
	if topic != nil && !topic(v) {
		return
	}
	select {
	case sub <- v:
	case <-time.After(p.timeout):
	}
}
func main() {
	p := NewPublisher(time.Second*100, 10)
	defer p.Close()

	all := p.AddSubscriber()
	golang := p.SubscribeTopic(func(v interface{}) bool {
		if s, ok := v.(string); ok {
			return strings.Contains(s, "Golang")
		}
		return false
	})
	p.Publish("Hello, world!")
	p.Publish("Hello, Golang!")

	go func() {
		for v := range all {
			println("All:", v)
		}
	}()
	go func() {
		for v := range golang {
			println("Golang:", v)
		}
	}()

	time.Sleep(time.Second * 3)
}

```

