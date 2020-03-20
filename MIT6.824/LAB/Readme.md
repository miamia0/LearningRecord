当前进度

- [x] Lab 1
- [x] Lab 2
   - [x] Lab 2A
    - [x] Lab 2B
    - [x] Lab 2C
  - [ ] Lab 3
      - [ ] Lab 3A
      - [ ] Lab 3B
- [ ] Lab 4

  - [ ] Lab 4A
  - [ ] Lab 4B
  

### Lab1

---



### Lab2

---



LAB2遇到的一些问题

* 因为Leader在接收到更大的term的时候需要转变回Follower，但是如果转变term后仍然发送了heartbeat会覆盖其他server的log。
* candidate，leader在接收到合法的requestvote或者heartbeat的时候应该立即转变为follower。
* follower在收到任意不合法的requestvote时仍然需要将自己的term进行更新。
* 因为需要用到log的长度作为election的依据，所以在更新log的时候必须将之后的无用log完全丢弃。
* 因为candidate的选举是使用随机时钟来保证每个follower都有被选举的机会，所以在测试条件不是很好的情况下，随机时钟可以将随机返回扩大一些。
* 多线程同步真的很重要。。



写2C的时候感觉并没有写很多东西····

###Lab3

---

