---
layout: post
title: "boost signal2"
description: ""
category: boost 
tags: [boost,signal2]
---
{% include JB/setup %}

ost.Signal2
----------------------

## Introduction
Boost.Signal2 라이브러리는 managed singals와 slots system을 구현한 라이브러리다. 여기서 signals은 event, slot은 event 발생시에 호출되는 callback receiver 라고 이해하면 된다. 


### Signals2 ###
Boost.Signals에서 thread-safe를 지원하기 위해 수정된 라이브러리가 Signals2 이다. 

## Tutorial ##

### Basic ###
Boost.Singals2를 이해하기 위해 Hello World 예제를 가지고 살펴보자. 우선 아그먼트가 없고 return 타입이 void인 sig를 선언한다. 
다음, 'Hello World' 를 출력하는 HellowWorld obj를 sig에 connect 한다. 이제 signal sig()를 호출하면 connect된 slot 'HelloWorld::operator()' 가 호출된다. 

    struct HelloWorld
    {
      void operator()() const
      {
        std::cout << "Hello, World!" << std::endl;
      }
    };

    // Signal with no arguments and a void return value
    boost::signals2::signal<void ()> sig;

    // Connect a HelloWorld slot
    HelloWorld hello;
    sig.connect(hello);

    // Call all of the slots
    sig();

또한 Signal에 Multiple Slot을 connect 할 수도 있고, Return되는 Slot의 순서를 정할 수도 있다. 

    boost::signals2::signal<void ()> sig;

    sig.connect(1, World());  // connect with group 1
    sig.connect(0, Hello());  // connect with group 0

    struct GoodMorning
    {
      void operator()() const
      {
        std::cout << "... and good morning!" << std::endl;
      }
    };

     // by default slots are connected at the end of the slot list
      sig.connect(GoodMorning());
    
      // slots are invoked this order:
      // 1) ungrouped slots connected with boost::signals2::at_front
      // 2) grouped slots according to ordering of their groups
      // 3) ungrouped slots connected with boost::signals2::at_back
      sig();

결과는 아래와 같다. 

    Hello, World!
    ... and good morning!

### Argument와 Return value가 있는 Slot 처리 ###
Slot은 Argument를 받을 수도 있고, Return값을 가질 수도 있다. 

    float product(float x, float y) { return x * y; }
    float quotient(float x, float y) { return x / y; }
    float sum(float x, float y) { return x + y; }
    float difference(float x, float y) { return x - y; 

    boost::signals2::signal<float (float, float)> sig;

    // The default combiner returns a boost::optional containing the return
    // value of the last slot in the slot list, in this case the
    // difference function.
    std::cout << *sig(5, 3) << std::endl;

### Conntect Management ###

Signal에 연결된 함수를 제거하기 위해 disconnect() 함수가 사용된다. 

    #include <boost/signal.hpp> 
    #include <iostream> 
    
    void func1() 
    { 
      std::cout << "Hello" << std::endl; 
    } 
    
    void func2() 
    { 
      std::cout << ", world!" << std::endl; 
    } 
    
    int main() 
    { 
      boost::signal<void ()> s; 
      s.connect(func1); 
      s.connect(func2); 
      s.disconnect(func2); 
      s(); 
    } 

signal이 trigger 되기 전에 func2가 제거된다. 따라서 결과는 Hello만 출력된다.


connect(), disconnect()외에도 signal에 연결된 함수의 개수를 리턴하는 num_slots(), empty(), disconnect\_all\_slots() 등의 메소드가 있다. 

    #include <boost/signal.hpp> 
    #include <iostream> 
    
    void func1() 
    { 
      std::cout << "Hello" << std::flush; 
    } 
    
    void func2() 
    { 
      std::cout << ", world!" << std::endl; 
    } 
    
    int main() 
    { 
      boost::signal<void ()> s; 
      s.connect(func1); 
      s.connect(func2); 
      std::cout << s.num_slots() << std::endl; 
      if (!s.empty()) 
        s(); 
      s.disconnect_all_slots(); 
    }

