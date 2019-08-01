+++
date = "2019-07-25T14:00:00+09:00"
title = "GopherCon 2019: Optimization for Number of goroutines Using Feedback Control"
tags = [ "golang", "gophercon" ]
+++

**The slides and speaker-notes about optimization for the number of goroutines I talked at [GopherCon 2019](https://www.gophercon.com/agenda/speakers/442434).**


<script async class="speakerdeck-embed" data-id="aa53e4353d9b4efc9064eefba40e13b7" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

> The design for the number of concurrency is important to achieve both speed and stability. To give a good performance without depending on platform and load conditions, it’s desirable for the number to be dynamic and rapidly controlled. In this talk, I will propose an architecture to solve this by utilizing feedback control.

# 1. Title

Good afternoon, everyone.
Thank you very much for coming today.

# 2. Self-introduction

My name is Yusuke MIYAKE.
My social account is @monochromegane.
I am a researcher of internet and operation technology, as well as a web developer.

# 3. Go and I

Let me introduce the relation between Go and me.
I am a Japanese gopher who loves writing OSS using Go.
My popular repositories are here (the_platinum_searcher is a fast grep tool).
And I am an organizer of the local Go community in Fukuoka, Japan.
So, we held Go Conference’19 summer in Fukuoka on 7/13 which was 2 weeks ago.
200 gophers in Japan gathered and enjoyed 30 sessions.
It was a lot of fun!
This is our conference gopher.
He has ramen which is a Japanese famous food on his head.
Isn't he good?

# 4. Agenda

Today I am here to talk to you about optimization for the number of goroutines.
My talk has 5 parts.
I’ll start by talking about why there is a need for optimization for the number of goroutines through the introduction and background section.
After that, I'll introduce my proposal that optimizes the number using feedback control.
And I will evaluate the efficacy of the method.
Finally, I will conclude about possible and some issues of the method.

# 5. Introduction

To begin with let’s speak to my experience of performance tuning of my OSS tool.

# 6. How many are the optimal number of goroutines?

I am developing a fast grep tool named "the platinum searcher".
One day I conducted a measurement of the optimal number of goroutines to achieve good performance.

# 7. Performance tuning in case of pt

"the platinum searcher" uses many goroutines to search string matching to pattern from files.
So, I measured performance while bounding concurrency using a semaphore.
Here is a result.
The X-axis represents the number of goroutines.
It is a log scale.
The Y-axis represents the speedup of processing time against sequential processing.
The result shows 8 or 32 goroutines make performance good.
This is 2 or 8 times more than the number of CPUs.

# 8. Performance tuning in case of pt

However, measurement in different environments showed a different result.
The result shows 16 goroutines make performance good.
This is 2 times more than the number of CPUs.
But I found that more goroutines cause performance degradation.
In this environment, the real-time virus scan process was running at the same time.
The process turns system-call of opening file slow and consumes CPU resources.
As a result, performance is degraded when searching for many files at the same time.

# 9. Performance tuning in case of pt

Besides, there were other results.
When the real-time virus scan process stops, 4 goroutines made performance good.
This is half times less than the number of CPUs.
I don't know how many the optimal number of goroutines are?

# 10. Performance tuning in case of pt

Currently, "the platinum searcher" is bounding concurrency using the number of CPUs.
Namely, I am choosing a safer way.
But it isn't the fastest.

# 11. How many are the optimal number of goroutines in each case?

So, that concludes this part of my talk on my experience of performance tuning.
Achieving both speed and stability, we have to decide optimum number of goroutines through our experiences and continuous tuning.
And, in most case, the environment on which the program is tuned and the environment on which it will be executed are different.
Thus, I'd like to consider optimization for number of goroutines without depending on environments.

# 12. Background

In this section, I will organize the issues about design for concurrency.

# 13.

I’m going to organize these issues in the following order;
Let us start with the first, which is the complexity of concurrency.
Then we come to the next, which is the contribution of Go to solve the complexity.
We will find an approach to solve the remaining issues, finally.

# 14. Concurrency and complexity

In this page, I will organize the relation of concurrency and complexity.

We introduce concurrency to improve performance.
Concurrency brings our application good performance and complexity.
Because concurrent processing is often considered as compared to simple serial processing.
This figure shows the common issues of concurrency.
We have to consider efficient thread management from a parallelism of view.
And we have to avoid race conditions and synchronize memory access.
And we have to design to be scalable.
Like this, we are dealing with many issues to improve performance.

# 15.

Then, I will organize the contribution of Go to solve the complexity.

# 16. Concurrency and Go

This is one of the reasons why we are interested in Go.
"Go" hides some complexities.
Because Go has rich features that support concurrency.
For example, we are freed from managing thread by runtime scheduler of Go.
And we also can avoid race conditions by using a channel.
In other words, Go decouples between your code and the complexity of parallelism.

# 17. Runtime scheduler of Go

In this page, I'll explain how Go's runtime hide the complexity.
This figure shows Go’s scheduler workflow.
Go’s scheduler has three basic concepts.
G is goroutine.
M is an OS thread.
P is a processor.
P handles multiplexing some goroutines onto some OS threads.

So, We can think of Goroutines as application-level threads. 
OS thread behaves like a worker for goroutine using a run queue.
What is important is P uses a smart scheduling strategy called a work-stealing algorithm.
Therefore P efficiently schedules available goroutines onto OS threads.
As Go's runtime automatically dose these, we don't need to consider the complexity of parallelism.

# 18. New “g”

Besides, lightweight goroutines make Go's scheduler more practical.
Creating a goroutine is cheap.
Because a newly created goroutine is allocated only 2kB stack.
Switching a goroutine is also cheap.
Because goroutine has only minimal context.
These points show that many goroutines can run at the same time.
As this practical runtime gives the illusion of parallelism more than the actual number of CPUs, we don't need to consider the complexity of parallelism.

# 19.

Finally, I will organize the remaining issues and find an approach to solve them.

# 20. Concurrency and application

Due to Go hides the complexity of parallelism, we can focus on concurrency issues in our application.
One of them is the design of the number of concurrency.
The reason is that the number of concurrency affects the performance depending on the characteristics of the application.
Certainly, the cost of generating and switching goroutines is very cheap.
On the other hand, when tasks on goroutine use shared resources, the upper limit of shared resources become a bottleneck.
Therefore, in this case, it is necessary to find the optimal number of concurrency to improve performance.

# 21. Concurrency and application

But the design for the number of concurrency is difficult.
Because the optimal number depends on app, environments and load condition.
And, in most cases, the environment in which the program is tuned and the environment in which it will be executed is different.
Thus, I'd like to consider optimization for the number of goroutines without depending on environments.
In order to achieve this, it's desirable for the number of concurrency to be determined dynamically and be controlled rapidly by detection the bottleneck on running program.

# 22. Proposal

In this section, I'll introduce my proposal that optimizes the number of concurrency.

# 23. Goal

The proposed method's goal is here.
The optimal number of concurrency to be determined dynamically and be controlled rapidly.

# 24. Basic idea

To begin with let’s speak to the basic idea of the method.
At first, It increases the number of goroutines to performance target.

# 25. Basic idea

After that, it stops increasing the number of goroutines if it meets the performance target.

# 26. Basic idea

After a while, it attempts to decrease the number of goroutines.
If it fails to meet the performance target, it backs to step 1.

# 27. Issues to solve for the realization

We have two issues to solve for the realization of the idea.

First, Selection of performance metrics
Next, finding how to control rapidly and continuously

In the following subsections, I try to realize the proposal while solving these problems.

# 28. Performance metrics

In this subsection, I will consider performance metrics that are without depending on application characteristics.

# 29. Performance metrics

In the basic idea subsection, I was assuming that there is a performance target.

It's desirable for the performance target not to depend on resource type whom task use.
Because different applications have different bottlenecks.
For example, I/O, Capacity of memory and processing load due to the external process.
If we find an ideal performance target, it will allow us to use the proposed method in as many applications as possible.

I think that CPU usage or throughput suits for the purpose.
Go's scheduler turns blocking tasks into as CPU bound as possible by switching tasks continuously.
Thus, I adopt the CPU usage of the Go application as the performance target.

# 30. Performance metrics

But the performance upper limit is different for each application.
In other words, we don't know the actual value of the target.
Thus, it is calculated on running.

At first, it sets the target value high.
It increases the number of goroutines to performance targets.

# 31. Performance metrics

If it doesn't meet the performance target, it regards the point as metrics upper limit.
After that, it gradually adjusts it to the new upper limit value.

So, that concludes this part of my talk on performance metrics.
The summary is the following:
The proposed method uses CPU usage upper limit as the performance target.
And the value is calculated on running.

# 32. Determining

In this subsection, I will consider how to determine the number of concurrency rapidly, continuously and accurately.

# 33. Determine the number of goroutines

In the previous subsection, we defined performance metrics and dynamic value.
Next, we have to consider how to determine the number of goroutines to meet the target value.
In the basic idea subsection, we increased goroutine one by one
But it has to determine rapidly, continuously, accurately to adjust the number of goroutines to rapid changes in the actual environment.
I think that feedback control is a good way to meet these conditions.

# 34. Feedback control

In this page, I will explain the basis of feedback control.

This figure shows the structure of a feedback loop.
There is a system with input and output.
The output is sent back and compared to the set-point to calculate a new input.
The error is a deviation of the output from the set-point.
The controller calculates how much to increase (or decrease) based on the error.

Feedback control applies an automatic correction continuously.
Determining set-point and Identifying suitable input/output is important.
Instead, Feedback control does not need to know the detail of the system.
I think that the robustness of feedback control is effective for rapid changes in the actual environment.

# 35. PID Controller (1/2)

In this page, I will explain a controller used frequently.

As I mentioned before, the job of controllers is to calculate the value of new input based on the error.
PID Controller applies a correction rapidly and accurately.
The controller has three sub-controllers inside.
The output is a combination of its proportional, integral and derivative subcontrol.

Now I’d like to look at the strategy of each sub-controller.

# 36. PID Controller (2/2)

The proportional control output is proportional to the error.
Namely, a large error will lead to a large adjustment.
KP is the controller gain.

The integral control output is proportional to the integral of the error over time.
The output becomes a cumulative sum of the error if it is a computer implementation.
KI is the controller gain.
The I controller deal with the small error that the P controller loses its effectiveness.

The derivative control output is proportional to the derivative of the error.
The output becomes the amount that has changed since the previous time step if it is a computer implementation.
KD is the controller gain.
The D controller encourages converge using the output.

So, that concludes this part of my talk on basic of PID controller.
The controller is intended to take the system closer to a set-point.
However, the set-point has to be determined beforehand.
In the proposed method, the value of the set-point is calculated on running.

Now I’d like to consider the structure of the controller to meet this condition.

# 37. Dynamic Target Controller

This figure shows the structure of the controller of the proposed method.
This controller has two nested control loops.
The inner loop is the PID controller which we have learned just before.
The inner loop's input is current CPU usage and its output is the number of goroutines.
The inner loop attempts to increase goroutine until CPU usage upper limit which is a set-point.
However, the value of the set-point needs to be calculated on running.
The job of the outer loop is to calculate this value.
The outer loop's input is current CPU usage as same as the inner loop.
Its output is a new set-point of the inner loop.
Namely, the inner loop's set-point is changed by the outer loop's output.

The outer loop behavior has been mentioned at the performance metrics subsection.
When CPU usage reached the upper limit by many goroutines, the outer loop begins to use its value as a new set-point.
In the current implementation, the outer loop sets a new set-point when CPU usage changes significantly.
Because Go's scheduler attempts to consume CPU resources incessantly by switching goroutines.
Namely, CPU usage stays on the upper limit in most cases.
Therefore, I figured it would be easier to detect change points in short-term observations than to detect stability in long-term observations.

(If time remains)
Indeed the design for the controller is pretty hard.
The current controller version is 11.
I published past designs and their evaluation.
if you have an interest in it, let's conversation about a better way during this conference.

So, that concludes this talk of my talk on dynamic target controller.
Now, we should be able to determine the optimal number of goroutines rapidly, continuously, accurately.
Finally, I’d like to consider how to bound concurrency with the determined number.

# 38. Bounding

In this subsection, I will explain about bounding concurrency dynamically.

# 39. To bound concurrency

This is a Go code that appeared after a long time.
We often write such code to bound concurrency in Go.
We usually use the buffered channel as a semaphore.
In this example, the buffer for the channel is 3.
Therefore, only three goroutines run at the same time.

On the other hand, in the proposed method the optimal number of goroutines will be different in every iteration of the feedback loop.
However, we can't change the buffer size of the channel later.
Therefore, we need a dynamic semaphore for the method.

# 40. Elastic semaphore

This figure shows the elastic semaphore.
This elastic semaphore provides "wait" and "signal" operations as same as a semaphore.
"wait" operation decrements the value of the semaphore variable.
If the new value of the semaphore variable is negative, it is blocked.
"signal" operation increments the value of the semaphore variable.
"incrementLimit" changes upper of the semaphore variable.
The value of the limit is determined by the feedback controller in the proposed method.
This elastic semaphore ensures the atomicity of these operations.

(If time remains)

This semaphore is not strict.
Because it allows that goroutines more than the upper limit run temporarily.
For example, 10 goroutines are running when the upper limit is 10.
After that, if the upper limit turns 5, 10 goroutines are still running until they finish their every task.
Of course, generating new goroutine is blocked.
Therefore, a long term goroutine like a worker process is not suitable for the elastic semaphore.
Fortunately, generating goroutine is cheap in Go.
So, we can generate it each time.

# 41. Kaburaya

In this subsection, I will explain kaburaya as the implementation of the proposed method.

# 42. Architecture of kaburaya

As I mentioned before, the goal of the proposed method is here.

I'd like to determine the number of concurrency dynamically.
I'd like to control the number of concurrency rapidly.

For the purpose,

Firstly, we decided to use CPU usage upper limit as performance metrics.
The value is calculated on running.
Secondly, we decided to determine the optimal number of goroutine using the metrics.
The value is determined rapidly, continuously, accurately using feedback control.
Finally, we decided to bound concurrency using the number.
The concurrency is bounded using elastic semaphore dynamically.

I am developing this implementation as OSS named kaburaya.
The URL is here.

As a side note, Kaburaya is the name of a Japanese arrow with a whistle.

# 43. monochromegane/kaburaya

This code is the usage of kaburaya.
It is very similar to the code of bounding concurrency with buffered channels.

"NewSem" specifies the period for feedback control and creates a new semaphore.
We can use "Wait" and "Signal" like a send or receive of a channel.
Kaburaya changes the limit of semaphore variables automatically and continuously.
So, we don't need the "incrementLimit" operation.
Lastly, if the tasks finished, we have to stop kaburaya to stop the feedback loop.

So, that concludes this part of my talk on the proposed method and its implementation.
I’d like to speak about the evaluation of kaburaya.

# 44. Evaluation

In this section, I will explain the evaluation of kaburaya.

# 45. Evaluation

There are 6 patterns of environments.
I evaluated the efficacy of kaburaya in each environment.
There are 3 contents.
They are speedup of processing, CPU usage based on time series, and limit of semaphore on time series.
I found good and not so good points in each environment accordingly.


# 46. pt_mac-scan

At first, I will explain this environment and task.

In this environment, "the platinum searcher" runs on Mac with a real-time virus scan.
The number of CPU is 8 and the memory size is 16GB.
GOMAXPROCS is set 8.

The graph represents performance against the number of goroutines.
The X-axis represents the number of goroutines.
It is a log scale.
The Y-axis represents the speedup of processing time against sequential processing.
The result shows 16 goroutines make performance itself good.
This task is more degradation of performance due to increased goroutines

Next, I’d like to take a look at the experiment result of kaburaya.

# 47. pt_mac-scan

This is a good pattern.
Because kaburaya achieved both speed and stability.

These figures show the results of the evaluation in the environment.
The number of the left graph which is 11 represents the median value of goroutine number determined by kaburaya.
As a consequence, kaburaya let processing time 6 times faster than sequential processing.

There are 3 plots based on time series in the right graph.
The blue dash line represents the set-point of CPU usage.
The blue line represents the actual value of CPU usage.
And the green line represents the number of semaphores determined by kaburaya.

In this environment and task, Kaburaya found good set points and adjust numbers within the ideal range.
Kaburaya was able to avoid performance degradation by determining the minimum number of the semaphore to meet the performance metrics.
As a result, not the best, but it achieved a good speedup.
Namely, kaburaya achieved both speed and stability.


# 48. Task: pt_mac-no-scan

This is also a good pattern.
In this environment, I stopped the real-time virus scan process and ran "the platinum searcher".

The result shows 4 goroutines make performance good.
This task is less degradation of performance due to increased goroutines.

# 49. Task: pt_mac-no-scan (Good)

In this environment and task, Kaburaya found good set points and adjust numbers within the ideal range.

Please pay attention to the blue dash line in the right graph.
This shows that the feedback controller continued to set a new set-point when CPU usage changed significantly.
As a result, kaburaya adjusted the number of semaphore within a good range.
Not the best, but it achieved a good speedup as few concurrency as possible.

# 50. Task: pt_mac-no-scan (Bad)

Unfortunately, there is a bad pattern.
This is the same as the previous environment.
But the number of semaphores continued to increase.
Because kaburaya failed to reset the set point.
Therefore actual CPU usage was always lower than set-point.

The cause is the range of change rate is too large.
It is used as a condition of determining set-point.

So, I have to adjust the value in order to improve the controller.

#51. Task: pt_linux

This is not so bad pattern.

In this environment, "the platinum searcher" runs on Linux.
The number of CPU is 4 and the memory size is 4GB.
GOMAXPROCS is set 4.

The result shows 8 or 32 goroutines make performance good.
This task is less degradation of performance due to increased goroutines.

# 52. Task: pt_linux (Not so bad)

The number of the left graph which is 15 represents the median value of goroutine number determined by kaburaya.
As a consequence, kaburaya let processing time 3.5 times faster than sequential processing.

In this environment and task, Kaburaya found good set points and adjust numbers within the ideal range.
However, at the end of the period, the number of semaphores increased due to the failure to reset the set point.

The cause is the same as before.
So, I have to adjust the range of change rate.

# 53. Task: mem_4096000_10000

Here is another task.
The task uses the memory of a shared resource.
In this case, the size is 4MB.
Many concurrencies will cause starvation of resources.

The number of CPU is 4 and the memory size is 4GB.
GOMAXPROCS is set 4.

Better concurrent is 512.
This task leads to more degradation of performance (swap out) due to increased goroutines.

# 54. Task: mem_4096000_10000 (Good)

The number of the left graph which is 63 represents the median value of goroutine number determined by kaburaya.
As a consequence, kaburaya let processing time 3 times faster than sequential processing.
So, Kaburaya found good set points and adjust numbers within the ideal range.

What is important is that it avoided the starvation of resources.
These results show the good set-point keep an ideal number of semaphore.

# 55. Task: mem_40960_1000000

This is also a good pattern.

I changed memory size to 40kb from 4MB.
The environment is the same as before.
Better concurrent are from 4 to 64.

# 56. Task: mem_40960_1000000 (Good)

The number of the left graph which is 19 represents the median value of goroutine number determined by kaburaya.
As a consequence, kaburaya let processing time 2.5 times faster than sequential processing.
So, Kaburaya found good set points and adjust numbers within the ideal range.

# 57. Task: mem_409600_100000

At last, I will show you an interesting pattern.

I changed memory size to 400kB.
The environment is the same as before.

Although when the number of the semaphore is 16 it seemed performance reached the limit, more goroutines made performance good.
Perhaps better concurrent will be over 1024.

# 58. Task: mem_409600_100000 (Not so good)

The number of the left graph which is 29 represents the median value of goroutine number determined by kaburaya.
As a consequence, kaburaya let processing time 2 times faster than sequential processing.

The results show kaburaya solved this task with a local solution.
Certainly, Kaburaya found set points and adjusted the numbers within a range.
But it didn't reach the upper of performance metrics.

So, I think that Kaburaya has to explore the set-point aggressively when the set-point is too low.

# 59. Experimental results

In this page, I conclude experimental results.

Kaburaya finds a good set point and adjusts numbers within ideal range by feedback control and elastic semaphore continuously.

On the other hand, we found some key factors to improve kaburaya.

The most important is the prediction accuracy of the set point.
The improvement will avoid that the number of the semaphore is too much or too less.

And finding optimal parameters is also important.
Based on my experience, I got good results in most cases when the gain of the feedback controller is from 0.1 to 0.3 and the range of change rate is 0.3.
However, I think that we have to tune the parameters as necessary.

I expect that there is a solution in the related researches on feedback control.

# 60. Conclusion

OK, This is a conclusion.

# 61. Conclusion

I proposed kaburaya to control the number of goroutines without depending on the platform, runtime, and current load.
Experimental results show possible and some issues of kaburaya.
In particular, improvement in detecting performance upper limit of the task is important.
In the future, I will also consider the application to auto-scaling of cloud computing.

# 62. Appendix

This is the Appendix.

# 63. Reference

Here is Reference.
If you want to learn more about kaburaya, you can jump to the link.
Sorry, some texts are in Japanese.

# 64. Go gopher

The Go gophers in this slide were drawn by Keita Kawamoto.

# 65. Thank you!

Thank you for listening.
It was a pleasure being here today.
