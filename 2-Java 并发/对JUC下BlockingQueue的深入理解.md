![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211116113520.png)

阻塞队列，是一个什么样的队列呢？我们知道BlockingQueue接口继承自Queue，其实在Queue接口中已经提供了对队列的各种操作，比如入队add, offer(与add相比，当队列满了后再用offer加入元素会返回false，而用add会抛出异常)，出队remove, poll(和remove相比，当队列为空时，poll会返回null，而remove抛出异常)；BlockingQueue又新增了take和put方法，提供一个阻塞的作用，当队列为空时去take会阻塞，当队列满了时去put会阻塞；所以BlockingQueue相比于Queue其实就是多了阻塞，当然还有offer(time), poll(time)在限定时间内进行取、放；

BlockingQueue通过锁保证线程安全，以此也就支持多生产者、多消费者模型。











