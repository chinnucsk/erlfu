# Erlfu #

Futures implemented in Erlang. Very basic implementation using
processes to represent a future.

## Goals ##

Implement futures/promises framework which allows to chain futures to
implement reusable mechanisms like timeouts, authentication, sharding,
etc.

Inspired by http://monkey.org/~marius/talks/twittersystems/

## Examples ##

Simple example with delayed setting of value:
```
1> F = future:new(fun() -> timer:sleep(10000), 10 end).
{future,<0.36.0>,#Ref<0.0.0.1736>,undefined}
2> F:get(). %% it hangs for 10 seconds
10
```

Exceptions are propagated with stacktrace preserved:
```
4> F = future:new(fun() -> a = b end).
{future,<0.41.0>,#Ref<0.0.0.21416>,undefined}
5> F:get().                                               
** exception error: no match of right hand side value b
     in function  erl_eval:expr/3 
     in call from future:get/1 (src/future.erl, line 94)
6> 
```

Values can be bound to future after it is created:
```
7> F = future:new().                                      
{future,<0.47.0>,#Ref<0.0.0.27235>,undefined}
8> spawn(fun() -> timer:sleep(10000), F:set(42) end).
<0.49.0>
9> F:get().
42
```

Multiple futures' values can be collected. If one future fails
everything will fail:
```
5> F1 = future:new(fun() -> timer:sleep(3000), 10 end).
{future,<0.50.0>,#Ref<0.0.0.76697>,undefined}
6> F2 = future:new(fun() -> timer:sleep(3000), 5 end). 
{future,<0.53.0>,#Ref<0.0.0.76941>,undefined}
7> lists:sum(future:collect([F1, F2])).
15
```
