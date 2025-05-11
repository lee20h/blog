---
title: "CPU Synchronization에 대해 알아보자"
date: 2021-09-21
tags:
  - CPU
categories:
  - operation system
publishResources: true
---


# CPU Synchronization

## Race Condition

CPU 동기화하는데에 있어 **Race Condition** 문제점이 있는데
```c
//count = 5일때 counter++, counter-- 연산
S0: producer execute register1 = counter {register1 = 5}  
S1: producer execute register1 = register1 + 1 {register1 = 6}  
S2: consumer execute register2 = counter{register2 = 5}  
S3: consumer execute register2 = register2 –1 {register2 = 4}  
S4: producer execute counter = register1 {counter = 6 }  
S5: consumer execute counter = register2 {counter = 4}
```  
이런식으로 실행할 때마다 답이 달라지는 경우를 Race Condition이라고 한다.

## Critical Section

프로세스끼리 변수를 공유하여 만들어진 시스템에 있어서 같은 변수를 사용하거나 table을 업데이트하거나 file을 쓰는 코드가 쓰인 부분을 **Critical Section**이라고 한다.  
Critical Section은 Entry Section과 Exit Section으로 나누어지는데 나머지를 Remainder Section이라 부른다.  
![image](https://github.com/lee20h/blog/assets/59367782/29c8a769-9c80-45e3-8dec-18b36586653f)

Critical Section은 3가지 필요조건을 만족해야한다.  
*1. Mutual Exclusion* - 한 프로세스가 C-S을 수행중이라면 다른 프로세스들은 C-S을 수행할 수 없다.  
*2. Progress* - C-S을 수행하는 프로세스가 없고 C-S을 수행하고자하는 프로세스가 있을 때 반드시 수행되어야 한다.  
*3. Bounded Waiting* - C-S을 요청한 프로세스는 무한히 대기하면 안된다.  
이 세가지 조건을 다 만족한 Peterson's Solution을 봐보자
```c
// int turn = 0; bool flag[3]; flag[1] = flag[2] = false;
//Process P₁			|	//Process P₂
flag[1] = true;			|	flag[2] = true;
turn = 1;			|	turn = 0;
while (flag[2] && (turn == 1));	|	while (flag[1] && (turn == 0));
	critical section	|		critical section
flag[1] = false;		|	flag[2] = false;
	remainder section	|		remainder section
```
이러한 3가지 조건을 만족하는 해결법은 크게 소프트웨어적, 하드웨어적 해결법으로 나뉜다.  
소프트웨어적 해결법은 test_and_set과 compare_and_swap 등이 있다.
```c
boolean test_and_set (boolean * target) {
	boolean rv = *target;
	*target = TRUE;
	return rv;
}
// 인자 값을 그대로 반환하되, 인자를 TRUE로 변경한다.
do {
	while (test_and_set(&lock)); /* do nothing */
		/* critical section */
	lock = false;
		/* remainder section */
	} while (true);
// lock이 false로 초기화되어있으므로 처음엔 바로 C-S을 수행하지만 lock이 true가 되어 다른 프로세스들은 수행하지 못한다. 이후 lock을 바꾸면서 수행하여 Mutual Exclusion 조건을 해결하였다.

do {
	waiting[i] = true;
	key = true;
	while (waiting[i] && key)
		key = test_and_set(&lock);
	waiting[i] = false;
	/* critical sectioin */
	next = (i+1) % n;
	while ((next != i) && !waiting[next])
		next = (next + 1) % n;
	if (next == i)
		lock = false;
	else
		waiting[next] = false;
	/*remainder section */
} while (true);
```
이 코드는 Bounded Waiting도 해결한 코드이다.

```c
int compare_and_swap(int *value, int expected, int new_value) {
	int temp = *value;
	if (*value == expected)
		*value = new_value;
	return temp;
}
do {
	while (compare_and_swap(&lock, 0, 1) != 0);
			/* do nothing */
		/* ciritcal section */
	lock = 0;
		/* remainder section */
	} while (true);
```
이 방법 또한 Mutual exlusion 을 해결했다. 차이점은 반환형이 다르다는 점이다.

### 하드웨어적 해결법
하드웨어적 해결법은 3가지가 있다.  
**1. Mutex lock (spinlock)** - 가장 기본적인 형태의 방법이다. acquire()와 release()로 잠금과 잠금해제를 한다. 잠그면 CS에 접근할 수 없다. test__and_set과 비슷하나, busy wait을 사용하므로 cpu를 계속 사용한다. 빠른 진입이 가능하나, CPU가 낭비된다. 따라서 CPU을 낭비 안할 때나, 금방 이용가능 할 때, Critical Section이 짧은 경우 사용되면 좋다.  
**2. Semaphore** - Binary와 Counting으로 나뉘어서 spinlock과 비슷하게 P() == wait()과 V() == signal()을 사용한다. Counting Semaphore는 Critical Section에 들어갈 수 있는 프로세스 갯수를 S 변수에 넣어 여러개로 관리가 가능하다. Binary Semaphore은 Mutex Lock와 같다. Busy wait을 사용유무로 종류가 네 가지가 나온다. 그 중에 일반적인 세마포어는 Counting하며, Non-busy wait을 사용하는 세마포어이다. 따라서 context switch 시간보다 길고 연산외에 시스템콜이나 예측불가한 작업시에는 세마포어를 사용하고 Critical Section 진입이 시간이 짧다면 스핀 락을 사용하는게 맞다.    
*3. Monitor* - 가장 high-level부분으로 가장 사용하기 편한 방법이라고 하나 크게 다루지 않았다.  
추가적으로 Deadlock과 Starvation이 있는데  
먼저 Deadlock은 여러 프로세스들이 수행될 때 프로세스 전부 wait상태에 빠진 경우다.  
Starvation은 특정 프로세스의 우선순위가 낮아서 자원 할당을 받지 못하는 경우이다.

## CPU동기화에 신경써야할 문제 3가지
### 1) Bounded-Buffer Problem

Bounded-Buffer Problem은 생산자와 소비자가 같은 버퍼를 점유할 때 일어나는 문제이다.

![Bounded-Buffer](https://github.com/lee20h/blog/assets/59367782/6975f309-0a1e-4120-9589-6b4f94c3e0c5)

*Bounded-Buffer Problem Solution*  
Empty : 버퍼 내에 저장할 공간이 있음을 표시, 생산자의 진입을 관리  
Full : 버퍼 내에 소비할 아이템이 있음을 표시, 소비자의 진입을 관리  
Mutex : 버퍼에 대한 접근을 관리, 생산자와 소비자가 empty, full 세마포어에 진입한 경우 버퍼의 상태 값을 변경하기위한 세마포어  
세마포어 value의 초기값은 full = 0, empty = n, mutex = 1로 생산자와 소비자의 프로세스을 정리한다.
```c
// 생산자 프로세스		            			|  소비자 프로세스      
Do {			        		       	|  Do {
	...				                |  	wait(full);
	/* produce an item in next_produced */		|  	wait(mutex);
	...                     			|  	...
	wait(empty);			            	|  	/* remove an item from buffe to next_consumed */
	wait(mutex);                    	    	|  	...
	...         					|  	signal(mutex);
	/* add next produced to the buffer */		|  	signal(empty);
	...			                        |  	...
	signal(mutex);		            		|  	/* consume the item in next consumed */
	siganl(full);	            			|  	...
} while(true);				            	|  } while(true);
```  

### 2) Readers-Writers Problem

![Readers-Writers](https://github.com/lee20h/blog/assets/59367782/e2944a85-cf9f-4cc6-baba-7ad9907410b4)

*Readers-Writers Problem*
- Readers : 공유 데이터를 읽는다.
	+ 여러 Reader는 동시에 데이터를 접근할 수 있다.
- Writers : 공유 데이터에 쓴다.
	+ Writer가 데이터를 수정할 때는 reader나 다른 writer가 접근하면 안된다.
```c
/* do while 생략 */
1)  
Writer				            		 |  Reader
wait(wrt);		//entry section			 |  wait(mutex);
		...			    	         |  readcount++;
writing is performed // critical section		 |  if (readcount == 1)
		...		            		 |  	wait(wrt);		// 어떤 writer도 수행X
signal(wrt);	//exit section				 |  signal(mutex);
					                 |  ... reading is performed... // critical section
				                	 |  wait(mutex);
				               		 |  readcount--;
				                	 |  if (readcount == 0)	
			                		 |  	signal(wrt);
				            		 |  signal(mutex);
```  
이때 Writer의 Starvation이 일어난다. 그 이유는 계속 Reader들이 진입하게되면 Writer는 Critical section에 진입할 수 없기 때문이다.

```c
2)
Writer					                    	|	Reader
								|
wait(wmutex); // Writer Process entry section			|	wait(read); // Reader Process entry section
writedcount++;			                		|	signal(read);
if (writecount == 1)	            				|	wait(rmutex);
	wait(read);		                    		|	readcount++;
signal(wmutex);				                	|	if (readcount == 1)
wait(wrt);	                       				|		wait(wrt);
...writing is performed... // critical section			|	signal(rmutex);
signal(wrt); // Writer Process exit section	    		|	...reading is performed... // critical section
wait(wmutex);	                				|	wait(rmutex); // Reader Process exit sectioin
writecount--;					                |	readcount--;
if (writecount == 0)		            			|	if (readcount == 0)
	signal(read);			                	|		signal(wrt);
signal(wmutex);			                		|	signal(rmutex);
```  
`1)`의 Writer의 starvation을 해결하기 위해 짜여졌으나 `2)`에서는 Reader들이 오히려 starvation에 빠지게 된다.  
Reader가 초기값으로 진입했다면 Reader들이 계속 진입하다가 Writer가 진입하게되면 진입한 Reader이 모두 수행하면 Writer가 수행된다. 이 때 Reader가 더 이상 진입하지 못하여 Starvation이 일어난다.  
또, Writer가 초기값으로 진입해서 계속 Writer만 진입한다면 Reader가 Starvation에 빠지게 된다.
```c
3)
// Writer				               		|	Reader
wait(mutex); // Writer Process entry section			|	wait(mutex); // Reader Process entry section
if(rc>0 || wc>0 || rwc>0 || wwc>0) {	   		 	|	if(wc>0 || wwc>0) {
	wwc++;	                    				|		rwc++;  
	signal(mutex);		                    	   	|		signal(mutex);
	wait(wrt);				                |		wait(read);
	wait(mutex);			        		|		wait(mutex);
	wwc--;				                    	|		rwc--;
}							        |	}
wc++;						                |	rc++;
signal(mutex);			                	    	|	signal(mutex);
...writing is performed // critical section	  		|	...reading is performed // critical section
wait(mutex); // Writer Process exit section			|	wait(mutex); // Reader Process exit section
wc--;						              	|	rc--;
if(rwc>0) {					               	|	if(rc == 0 && wwc>0)
	for (i=0; i<rwc; i++)	      		          	|		signal(wrt);
		signal(read);		      		       	|	signal(mutex);
}							        |
else						               	|
	if (wwc>0) signal(wrt);	   	 	            	|
signal(mutex);				                	|
```
`3)`의 경우에는 모든 문제가 해결되어 동기화가 잘 이루어진다.  
Writer는 작업이 수행되거나 대기중인 다른 reader, writer가 있다면 대기한다. 그리고 수행 후 대기중인 reader들을 모두 수행한다.  
Reader는 writer가 기다리거나 작업중이라면 대기한다. Reader가 다 수행되면 대기 중인 writer을 수행한다.

### 3) Dining-Philosophers Problem
![Dining-Philosophers](https://github.com/lee20h/blog/assets/59367782/9c576884-5f79-40bb-8afc-a652fd7b732b)

*Dining-Philosophers Problem*은 젓가락이 5개가 있을 때 자신과 이웃한 젓가락만 들 수 있으며 젓가락을 2개 들었을 때 식사가 가능하다.
```c
1)
do {
		...
		think
		...
	wait(chopstick[i])
	wait(chopstick[(i+1) % 5])
		...
		eat
		...
	signal(chopstick[i])
	signal(chopstick[(i+1) % 5])
		...
} while(1);
```
동시에 젓가락을 집으면 deadlock이 발생한다.
```c
2)
do {
		...
		think
		...
	take_chopsticks(i);
		...
		eat
		...
	put_chopsticks(i)
		...
} while(1);

take_chopstics(int i) {			|		put_chopsticks(int i) {
	wait(mutex);	    		|			wait(mutex);
	state[i] = HUNGRY;		|			state[i] = THINK;
	test(i);		    	|			test(LEFT);
	signal(mutex);	    		|			test(RIGHT);
	signal(self[i]);		|			signal(mutex);
}		            		|		}

test(int i) {
	if (state[i] == HUNGRY && state[LEFT] != EATING && state[RIGHT] != EATING) {
		state[i] = EATING;
		signal(self[i]);
	}
}
```  
`2)`의 전략은 철학자들이 좌우 젓가락이 사용 가능할 때 critical section에 진입하여 deadlock과 starvation을 해결하였다. 