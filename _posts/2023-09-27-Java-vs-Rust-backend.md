---
layout: post
title:  "Java faster than Rust?!"
date:   2023-09-27 08:30:00 -0300
categories: Rust Java benchmark
---

# Java Faster than Rust?!

A post claimed that Java was faster than Rust in a backend challenge:

**"How to use Java superpowers to beat Rust in a backend challenge"**

[https://www.linkedin.com/pulse/how-use-java-superpowers-beat-rust-backend-challenge-leonardo-zanivan/](https://www.linkedin.com/pulse/how-use-java-superpowers-beat-rust-backend-challenge-leonardo-zanivan/)

From the post:

```
Java: 50996 records inserted, a response time of 120ms @ p99 and a total number of requests of 119412 (all successful).

Rust: 47010 records inserted, a response time of 37284ms @ p99 and a total number of requests of 115005.
```

Codes that he used to compare:

Rust with actix-web:
[https://github.com/viniciusfonseca/rinha-backend-rust](https://github.com/viniciusfonseca/rinha-backend-rust)

Java with Vertx:
[https://github.com/panga/rinhabackend-vertx](https://github.com/panga/rinhabackend-vertx)

**How could Java beat the blazingly fast Rust?! =)**<br><br>
![ThePrimeagen Licking Rust](/assets/images/ThePrimeagen-licking-rust.webp)

<br>
Because of the way this benchmark challenge works, there's no way to have more than 46576 valid insertions.
So there's a problem in his validation code.

The Rust version also passed that limit, because it had a bug that it inserts ~500
records before the stress test begins, and it was not deleting those records.


TL;DR:

What happened is that he used a faster hardware (Apple M2 Pro) than the
one used in the challenge. With his hardware, the stress test was not really
stressing that much, and both solutions would have had the same number of insertions,
if not by the bugs mentioned above.

The same happened with MrPowerGamerBR:<br>
[The Results are wrong, and I can prove](https://www.youtube.com/watch?v=XqYdhlkRlus)

... where he showed that his and other solutions were able to achieve the same number
of insertions.

He used: AMD Ryzen 7 5800X3D, 32GBs and SSD NVMe


With faster HW, worse solutions sometimes can achieve the same result.


On the remaining of the post I'll show the numbers that I got running on my
 Lenovo ideapad S145:
 - 8GB
 - AMD Ryzen 5 3500u
 - SSD

|Code|Requests|KO|Insertions|
|----|--------|--|----------|
|[Java docker-compose.yml](#java-docker---compose.yml)| 68647    | 63543  | 19931 |
|[Java docker-compose-local.yml](#java-docker---compose-local.yml) | 93545    | 13015  | 26815 |
|[Java docker-compose-native.yml](#java-docker---compose-native.yml) | 99665    | 20730  | 39260 |
|[Java outside docker](#java-outside-docker) | 114975   | 0      | 46569 |
|[Rust](#rust)| 113360 | 4855 | 43278 |
|[Rust before Akita changes](https://github.com/tivrfoa/rbrust/tree/original-version)      | 109916   | 7373   | 41512 |
|[Rust without duplicate format](https://github.com/tivrfoa/rbrust/tree/fix-duplicate-format-and-unnecessary-clone)  | 116811   | 0      | 46570 |
|[Rust Axum Sqlx](#rust-axum-sqlx) | 114823 | 169 | 46409 |
|[Rust Axum Sqlx outside docker](#rust-axum-sqlx-outside-docker)  | 114995 | 0 | 46578 |

*ps: you should always confirm these results running on your machine. The numbers will be different, because of different hardwares, but the relative difference should be close ...
And sadly, some results are not consistent between two runs, because there's a randomization in the stress test¹.*

¹ MrPowerGamerBR showed how to make it more deterministic here:
[The Results are wrong, and I can prove](https://www.youtube.com/watch?v=XqYdhlkRlus)

*rinha-de-backend-2023-q3/stress-test/user-files/simulations/rinhabackend/RinhaBackendSimulation.scala*

```diff
-      constantUsersPerSec(5).during(15.seconds).randomized, // are you ready?
+      constantUsersPerSec(5).during(15.seconds), // are you ready?
```

# Challenge Instructions and Rules

[https://github.com/zanfranceschi/rinha-de-backend-2023-q3/blob/main/INSTRUCOES.md](https://github.com/zanfranceschi/rinha-de-backend-2023-q3/blob/main/INSTRUCOES.md)

# Java

I generated `.class` files using Correto 20:

`JAVA_HOME=/some-path/amazon-corretto-20.0.2.10.1-linux-x64 mvn clean package`

The Java project has more than one docker-compose:

  - docker-compose.yml
  - docker-compose-local.yml

## Using docker-compose.yml

### Java Errors on startup

The erros below are happening after `docker-compose up`, even before executing
the stress test.

```
api01_1  | Sep 18, 2023 8:19:18 PM io.vertx.core.impl.BlockedThreadChecker
api01_1  | WARNING: Thread Thread[#19,vert.x-eventloop-thread-1,5,main] has been blocked for 2005 ms, time limit is 2000 ms
api01_1  | Sep 18, 2023 8:19:20 PM io.vertx.core.impl.BlockedThreadChecker
api01_1  | WARNING: Thread Thread[#19,vert.x-eventloop-thread-1,5,main] has been blocked for 3800 ms, time limit is 2000 ms
api02_1  | Sep 18, 2023 8:19:18 PM io.vertx.core.impl.BlockedThreadChecker
api02_1  | WARNING: Thread Thread[#19,vert.x-eventloop-thread-1,5,main] has been blocked for 2108 ms, time limit is 2000 ms
api02_1  | Sep 18, 2023 8:19:20 PM io.vertx.core.impl.BlockedThreadChecker
api02_1  | WARNING: Thread Thread[#19,vert.x-eventloop-thread-1,5,main] has been blocked for 4203 ms, time limit is 2000 ms
api01_1  | Listening on port 80
api01_1  | Sep 18, 2023 8:19:20 PM io.vertx.core.impl.launcher.commands.VertxIsolatedDeployer
api01_1  | INFO: Succeeded in deploying verticle
api02_1  | Listening on port 80
api02_1  | Sep 18, 2023 8:19:21 PM io.vertx.core.impl.launcher.commands.VertxIsolatedDeployer
api02_1  | INFO: Succeeded in deploying verticle
```

### Errors during stress test

The error below happend a lot:

```
nginx_1  | 2023/09/18 20:25:18 [error] 32#32: *71036 no live upstreams while connecting to upstream, client: 172.18.0.1, server: , request: "POST /pessoas HTTP/1.1", upstream: "http://api/pessoas", host: "localhost:9999"
nginx_1  | 2023/09/18 20:25:20 [error] 32#32: *71039 no live upstreams while connecting to upstream, client: 172.18.0.1, server: , request: "GET /pessoas?t=FhgrjREPOCiYmdgUPuK+ETe HTTP/1.1", upstream: "http://api/pessoas?t=FhgrjREPOCiYmdgUPuK+ETe", host: "localhost:9999"
nginx_1  | 2023/09/18 20:25:21 [error] 32#32: *71114 no live upstreams while connecting to upstream, client: 172.18.0.1, server: , request: "GET /pessoas?t=xEuFfbusdVDQQQthpYPVFcaHuc+q HTTP/1.1", upstream: "http://api/pessoas?t=xEuFfbusdVDQQQthpYPVFcaHuc+q", host: "localhost:9999"
```

### Result

The Java code uses lots of CPU. 0.15 is just too litle for it ...

We can improve it a bit, giving it more cpu. More on that later.

```
---- Requests ------------------------------------------------------------------
> Global                                                   (OK=5104   KO=63543 )
> busca inválida                                           (OK=1930   KO=2260  )
> criação                                                  (OK=2903   KO=51734 )
> busca válida                                             (OK=113    KO=9477  )
> consulta                                                 (OK=158    KO=72    )
---- Errors --------------------------------------------------------------------
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms      24534 (38,61%)
> status.find.in(201,422,400), but actually found 502             22414 (35,27%)
> i.n.c.ConnectTimeoutException: connection timed out: localhost   9232 (14,53%)
/127.0.0.1:9999
> status.find.in([200, 209], 304), found 502                       3845 ( 6,05%)
> j.i.IOException: Premature close                                 1783 ( 2,81%)
> status.find.is(400), but actually found 502                      1631 ( 2,57%)
> status.find.in(201,422,400), but actually found 504               104 ( 0,16%)

================================================================================
---- Global Information --------------------------------------------------------
> request count                                      68647 (OK=5104   KO=63543 )
> min response time                                      0 (OK=4      KO=0     )
> max response time                                  60524 (OK=59975  KO=60524 )
> mean response time                                 27031 (OK=14865  KO=28008 )
> std deviation                                      25425 (OK=9210   KO=26051 )
> response time 50th percentile                      14899 (OK=14396  KO=15022 )
> response time 75th percentile                      60000 (OK=20434  KO=60000 )
> response time 95th percentile                      60001 (OK=31877  KO=60001 )
> response time 99th percentile                      60004 (OK=37900  KO=60004 )
> mean requests/sec                                258.071 (OK=19.188 KO=238.883)
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                           273 (  0%)
> 800 ms <= t < 1200 ms                                108 (  0%)
> t >= 1200 ms                                        4723 (  7%)
> failed                                             63543 ( 93%)
---- Errors --------------------------------------------------------------------
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms      24534 (38,61%)
> status.find.in(201,422,400), but actually found 502             22414 (35,27%)
> i.n.c.ConnectTimeoutException: connection timed out: localhost   9232 (14,53%)
/127.0.0.1:9999
> status.find.in([200, 209], 304), found 502                       3845 ( 6,05%)
> j.i.IOException: Premature close                                 1783 ( 2,81%)
> status.find.is(400), but actually found 502                      1631 ( 2,57%)
> status.find.in(201,422,400), but actually found 504               104 ( 0,16%)
================================================================================
<
* Connection #0 to host localhost left intact
19931
```

It's strange that it says it created 19931 registers, which it did, but
the `Response Time Distribution` does not show more than 5500 successful
requests ...

So it created, but failed to communicate it back to the client.

## Using docker-compose-local

No errors on startup.

The error `no live upstreams while connecting to upstream` did not happen.

### Result

```
---- Requests ------------------------------------------------------------------
> Global                                                   (OK=80530  KO=13015 )
> busca inválida                                           (OK=3675   KO=515   )
> criação                                                  (OK=46955  KO=7679  )
> busca válida                                             (OK=4883   KO=4707  )
> consulta                                                 (OK=25017  KO=114   )
---- Errors --------------------------------------------------------------------
> i.n.c.ConnectTimeoutException: connection timed out: localhost   5237 (40,24%)
/127.0.0.1:9999
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms       3754 (28,84%)
> status.find.in([200, 209], 304), found 500                       3450 (26,51%)
> j.i.IOException: Premature close                                  574 ( 4,41%)

================================================================================
---- Global Information --------------------------------------------------------
> request count                                      93545 (OK=80530  KO=13015 )
> min response time                                      0 (OK=0      KO=1     )
> max response time                                  60019 (OK=59962  KO=60019 )
> mean response time                                 13416 (OK=11952  KO=22473 )
> std deviation                                      15707 (OK=13189  KO=24526 )
> response time 50th percentile                       8682 (OK=6935   KO=10485 )
> response time 75th percentile                      20514 (OK=19835  KO=60000 )
> response time 95th percentile                      50461 (OK=39449  KO=60001 )
> response time 99th percentile                      60000 (OK=50910  KO=60001 )
> mean requests/sec                                369.743 (OK=318.3  KO=51.443)
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                         22222 ( 24%)
> 800 ms <= t < 1200 ms                               1302 (  1%)
> t >= 1200 ms                                       57006 ( 61%)
> failed                                             13015 ( 14%)
---- Errors --------------------------------------------------------------------
> i.n.c.ConnectTimeoutException: connection timed out: localhost   5237 (40,24%)
/127.0.0.1:9999
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms       3754 (28,84%)
> status.find.in([200, 209], 304), found 500                       3450 (26,51%)
> j.i.IOException: Premature close                                  574 ( 4,41%)
================================================================================

* Connection #0 to host localhost left intact
26815
```

# Java docker-compose-native.yml

Java Native with GraalVM

```
---- Requests ------------------------------------------------------------------
> Global                                                   (OK=78935  KO=20730 )
> busca inválida                                           (OK=3619   KO=571   )
> criação                                                  (OK=37573  KO=17056 )
> busca válida                                             (OK=6507   KO=3083  )
> consulta                                                 (OK=31236  KO=20    )
---- Errors --------------------------------------------------------------------
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms      10414 (50,24%)
> i.n.c.ConnectTimeoutException: connection timed out: localhost   8455 (40,79%)
/127.0.0.1:9999
> j.i.IOException: Premature close                                  995 ( 4,80%)
> j.n.ConnectException: connect(..) failed: Cannot assign reques    866 ( 4,18%)
ted address

================================================================================
---- Global Information --------------------------------------------------------
> request count                                      99665 (OK=78935  KO=20730 )
> min response time                                      0 (OK=0      KO=1427  )
> max response time                                  66642 (OK=66642  KO=65902 )
> mean response time                                 26114 (OK=22260  KO=40792 )
> std deviation                                      22508 (OK=21438  KO=20340 )
> response time 50th percentile                      24172 (OK=16021  KO=59551 )
> response time 75th percentile                      47109 (OK=44398  KO=60000 )
> response time 95th percentile                      60000 (OK=53325  KO=60001 )
> response time 99th percentile                      60001 (OK=58410  KO=60003 )
> mean requests/sec                                344.862 (OK=273.131 KO=71.73 )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                         25418 ( 26%)
> 800 ms <= t < 1200 ms                               1374 (  1%)
> t >= 1200 ms                                       52143 ( 52%)
> failed                                             20730 ( 21%)
---- Errors --------------------------------------------------------------------
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms      10414 (50,24%)
> i.n.c.ConnectTimeoutException: connection timed out: localhost   8455 (40,79%)
/127.0.0.1:9999
> j.i.IOException: Premature close                                  995 ( 4,80%)
> j.n.ConnectException: connect(..) failed: Cannot assign reques    866 ( 4,18%)
ted address
================================================================================

* Connection #0 to host localhost left intact
39260
```

## Java outside docker

Running the app without CPU/memory constraints.

This is a good test to check if the app is really working and to confirm
that the bad results were caused because of the resources constraints on the
docker containers.

The PostgreSQL database was still running as a container, though.

```
---- Requests ------------------------------------------------------------------
> Global                                                   (OK=114975 KO=0     )
> busca inválida                                           (OK=4190   KO=0     )
> criação                                                  (OK=54626  KO=0     )
> busca válida                                             (OK=9590   KO=0     )
> consulta                                                 (OK=46569  KO=0     )

================================================================================
---- Global Information --------------------------------------------------------
> request count                                     114975 (OK=114975 KO=0     )
> min response time                                      0 (OK=0      KO=-     )
> max response time                                  57071 (OK=57071  KO=-     )
> mean response time                                 11823 (OK=11823  KO=-     )
> std deviation                                      16495 (OK=16495  KO=-     )
> response time 50th percentile                       1410 (OK=1413   KO=-     )
> response time 75th percentile                      21695 (OK=21693  KO=-     )
> response time 95th percentile                      48101 (OK=48088  KO=-     )
> response time 99th percentile                      55121 (OK=55121  KO=-     )
> mean requests/sec                                432.237 (OK=432.237 KO=-     )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                         50764 ( 44%)
> 800 ms <= t < 1200 ms                               5761 (  5%)
> t >= 1200 ms                                       58450 ( 51%)
> failed                                                 0 (  0%)
================================================================================

* Connection #0 to host localhost left intact
46569
```

# Rust

```
---- Requests ------------------------------------------------------------------
> Global                                                   (OK=108505 KO=4855  )
> busca válida                                             (OK=8929   KO=661   )
> busca inválida                                           (OK=3922   KO=268   )
> criação                                                  (OK=50709  KO=3926  )
> consulta                                                 (OK=44945  KO=0     )
---- Errors --------------------------------------------------------------------
> i.n.c.ConnectTimeoutException: connection timed out: localhost   4819 (99,26%)
/127.0.0.1:9999
> j.i.IOException: Premature close                                   28 ( 0,58%)
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms          8 ( 0,16%)

================================================================================
---- Global Information --------------------------------------------------------
> request count                                     113360 (OK=108505 KO=4855  )
> min response time                                      0 (OK=0      KO=10000 )
> max response time                                  60002 (OK=50163  KO=60002 )
> mean response time                                  1303 (OK=905    KO=10196 )
> std deviation                                       2758 (OK=1980   KO=2722  )
> response time 50th percentile                         99 (OK=73     KO=10000 )
> response time 75th percentile                       1207 (OK=1059   KO=10000 )
> response time 95th percentile                       9269 (OK=4413   KO=10001 )
> response time 99th percentile                      10001 (OK=8935   KO=10009 )
> mean requests/sec                                482.383 (OK=461.723 KO=20.66 )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                         77496 ( 68%)
> 800 ms <= t < 1200 ms                               7361 (  6%)
> t >= 1200 ms                                       23648 ( 21%)
> failed                                              4855 (  4%)
---- Errors --------------------------------------------------------------------
> i.n.c.ConnectTimeoutException: connection timed out: localhost   4819 (99,26%)
/127.0.0.1:9999
> j.i.IOException: Premature close                                   28 ( 0,58%)
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms          8 ( 0,16%)
================================================================================

* Connection #0 to host localhost left intact
43278
```

# Rust Original Version - Before Akita Changes

```
---- Requests ------------------------------------------------------------------
> Global                                                   (OK=102543 KO=7373  )
> busca válida                                             (OK=8577   KO=1013  )
> busca inválida                                           (OK=3767   KO=423   )
> criação                                                  (OK=48687  KO=5937  )
> consulta                                                 (OK=41512  KO=0     )
---- Errors --------------------------------------------------------------------
> i.n.c.ConnectTimeoutException: connection timed out: localhost   7118 (96,54%)
/127.0.0.1:9999
> j.i.IOException: Premature close                                  246 ( 3,34%)
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms          9 ( 0,12%)

================================================================================
---- Global Information --------------------------------------------------------
> request count                                     109916 (OK=102543 KO=7373  )
> min response time                                      0 (OK=0      KO=9460  )
> max response time                                  60001 (OK=59151  KO=60001 )
> mean response time                                  1852 (OK=1237   KO=10410 )
> std deviation                                       3439 (OK=2527   KO=2988  )
> response time 50th percentile                        467 (OK=385    KO=10000 )
> response time 75th percentile                       1577 (OK=1315   KO=10001 )
> response time 95th percentile                      10000 (OK=4687   KO=10002 )
> response time 99th percentile                      10149 (OK=9633   KO=22768 )
> mean requests/sec                                467.728 (OK=436.353 KO=31.374)
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                         64951 ( 59%)
> 800 ms <= t < 1200 ms                               8239 (  7%)
> t >= 1200 ms                                       29353 ( 27%)
> failed                                              7373 (  7%)
---- Errors --------------------------------------------------------------------
> i.n.c.ConnectTimeoutException: connection timed out: localhost   7118 (96,54%)
/127.0.0.1:9999
> j.i.IOException: Premature close                                  246 ( 3,34%)
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms          9 ( 0,12%)
================================================================================
* Connection #0 to host localhost left intact
41512
```

# Rust Running on Ubuntu and removing duplicate format! and clone

```
---- Requests ------------------------------------------------------------------
> Global                                                   (OK=116811 KO=0     )
> busca válida                                             (OK=9590   KO=0     )
> criação                                                  (OK=54628  KO=0     )
> busca inválida                                           (OK=4190   KO=0     )
> consulta                                                 (OK=48403  KO=0     )

================================================================================
---- Global Information --------------------------------------------------------
> request count                                     116811 (OK=116811 KO=0     )
> min response time                                      0 (OK=0      KO=-     )
> max response time                                   3826 (OK=3826   KO=-     )
> mean response time                                   173 (OK=173    KO=-     )
> std deviation                                        241 (OK=241    KO=-     )
> response time 50th percentile                        107 (OK=107    KO=-     )
> response time 75th percentile                        239 (OK=239    KO=-     )
> response time 95th percentile                        668 (OK=669    KO=-     )
> response time 99th percentile                       1047 (OK=1048   KO=-     )
> mean requests/sec                                564.304 (OK=564.304 KO=-     )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                        113389 ( 97%)
> 800 ms <= t < 1200 ms                               2775 (  2%)
> t >= 1200 ms                                         647 (  1%)
> failed                                                 0 (  0%)
================================================================================

* Connection #0 to host localhost left intact
46570
```

# Rust Axum Sqlx

[https://github.com/tivrfoa/rinha-backend-2023-rust-axum-sqlx/commit/640b69868862570407fb13c7e7c517f235d22b7c](https://github.com/tivrfoa/rinha-backend-2023-rust-axum-sqlx/commit/640b69868862570407fb13c7e7c517f235d22b7c)

```
---- Requests ------------------------------------------------------------------
> Global                                                   (OK=114654 KO=169   )
> criação                                                  (OK=54634  KO=0     )
> busca inválida                                           (OK=4190   KO=0     )
> busca válida                                             (OK=9590   KO=0     )
> consulta                                                 (OK=46240  KO=169   )
---- Errors --------------------------------------------------------------------
> status.find.in([200, 209], 304), found 404                        169 (100,0%)

Simulation RinhaBackendSimulation completed in 211 seconds

================================================================================
---- Global Information --------------------------------------------------------
> request count                                     114823 (OK=114654 KO=169   )
> min response time                                      0 (OK=0      KO=5000  )
> max response time                                   6225 (OK=6225   KO=5273  )
> mean response time                                   490 (OK=483    KO=5081  )
> std deviation                                       1048 (OK=1034   KO=43    )
> response time 50th percentile                         52 (OK=51     KO=5083  )
> response time 75th percentile                        247 (OK=244    KO=5091  )
> response time 95th percentile                       3306 (OK=3280   KO=5176  )
> response time 99th percentile                       4571 (OK=4477   KO=5194  )
> mean requests/sec                                541.618 (OK=540.821 KO=0.797 )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                         95653 ( 83%)
> 800 ms <= t < 1200 ms                               3494 (  3%)
> t >= 1200 ms                                       15507 ( 14%)
> failed                                               169 (  0%)
---- Errors --------------------------------------------------------------------
> status.find.in([200, 209], 304), found 404                        169 (100,0%)
================================================================================
* Connection #0 to host localhost left intact
46409
```

# Rust Axum Sqlx outside docker

```sh
HTTP_PORT=9999 ./target/release/rinha-backend-axum
Max connections: 4
Acquire Timeout: 3
listening on 127.0.0.1:9999
```

```
---- Requests ------------------------------------------------------------------
> Global                                                   (OK=114995 KO=0     )
> criação                                                  (OK=54637  KO=0     )
> busca válida                                             (OK=9590   KO=0     )
> busca inválida                                           (OK=4190   KO=0     )
> consulta                                                 (OK=46578  KO=0     )

Simulation RinhaBackendSimulation completed in 207 seconds

================================================================================
---- Global Information --------------------------------------------------------
> request count                                     114995 (OK=114995 KO=0     )
> min response time                                      0 (OK=0      KO=-     )
> max response time                                   3628 (OK=3628   KO=-     )
> mean response time                                   103 (OK=103    KO=-     )
> std deviation                                        391 (OK=391    KO=-     )
> response time 50th percentile                          1 (OK=1      KO=-     )
> response time 75th percentile                          2 (OK=2      KO=-     )
> response time 95th percentile                        714 (OK=711    KO=-     )
> response time 99th percentile                       1993 (OK=1993   KO=-     )
> mean requests/sec                                552.861 (OK=552.861 KO=-     )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                        109661 ( 95%)
> 800 ms <= t < 1200 ms                               2678 (  2%)
> t >= 1200 ms                                        2656 (  2%)
> failed                                                 0 (  0%)
================================================================================

* Connection #0 to host localhost left intact
46578
```

# Improving the Java version

0.15 cpu is just too little for the JVM.

After startup, it's already using more than 20% cpu, doing only GOD knows what. :P<br>
It's probably JIT working ...

You can check that using `docker stats`

I didn't change the source code, only the configuration:

  - Increased app cpu to 0.3
  - Decreased 0.3 cpu from db
  - Increased Nginx to 0.4GB
  - Used `network_mode: host` in all containers
  - Changed code to connect to db using `localhost` instead of container name
  - Increased max db connections to 10 for each app

```
---- Requests ------------------------------------------------------------------
> Global                                                   (OK=33737  KO=48767 )
> busca inválida                                           (OK=2883   KO=1307  )
> busca válida                                             (OK=3109   KO=6481  )
> criação                                                  (OK=18675  KO=35948 )
> consulta                                                 (OK=9070   KO=5031  )
---- Errors --------------------------------------------------------------------
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms      25151 (51,57%)
> status.find.in(201,422,400), but actually found 502             12252 (25,12%)
> i.n.c.ConnectTimeoutException: connection timed out: localhost   7870 (16,14%)
/127.0.0.1:9999
> status.find.in([200, 209], 304), found 502                       2090 ( 4,29%)
> status.find.is(400), but actually found 502                       817 ( 1,68%)
> j.i.IOException: Premature close                                  476 ( 0,98%)
> status.find.in(201,422,400), but actually found 504               111 ( 0,23%)

Simulation RinhaBackendSimulation completed in 281 seconds

================================================================================
---- Global Information --------------------------------------------------------
> request count                                      82504 (OK=33737  KO=48767 )
> min response time                                      1 (OK=1      KO=2     )
> max response time                                  60258 (OK=60228  KO=60258 )
> mean response time                                 32955 (OK=31934  KO=33661 )
> std deviation                                      25746 (OK=22697  KO=27637 )
> response time 50th percentile                      34524 (OK=32061  KO=60000 )
> response time 75th percentile                      60000 (OK=53687  KO=60000 )
> response time 95th percentile                      60001 (OK=59099  KO=60001 )
> response time 99th percentile                      60013 (OK=59817  KO=60024 )
> mean requests/sec                                292.567 (OK=119.635 KO=172.933)
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                          4204 (  5%)
> 800 ms <= t < 1200 ms                                661 (  1%)
> t >= 1200 ms                                       28872 ( 35%)
> failed                                             48767 ( 59%)
---- Errors --------------------------------------------------------------------
> Request timeout to localhost/127.0.0.1:9999 after 60000 ms      25151 (51,57%)
> status.find.in(201,422,400), but actually found 502             12252 (25,12%)
> i.n.c.ConnectTimeoutException: connection timed out: localhost   7870 (16,14%)
/127.0.0.1:9999
> status.find.in([200, 209], 304), found 502                       2090 ( 4,29%)
> status.find.is(400), but actually found 502                       817 ( 1,68%)
> j.i.IOException: Premature close                                  476 ( 0,98%)
> status.find.in(201,422,400), but actually found 504               111 ( 0,23%)
================================================================================
<
* Connection #0 to host localhost left intact
30392
```

# Akita Video about the Benchmark

Fabio Akita did a wonderful video (in Portuguese) about this benchmark:

[16 Linguagens em 16 Dias: Minha Saga da Rinha de Backend](https://www.youtube.com/watch?v=EifK2a_5K_U)

His conclusion was great, but I think some people might be lead to think that the language
does not matter much. (and maybe that's indeed what he wanted/thinks)

Basically he points that you could have achieved great performance with any
programming language for that benchmark, so you should choose the one you're most confortable with.

That is true in many cases, but it's flawed in many others.

For this benchmark this is true, if the only thing you're considering is the
number of insertions. But there are at least three important things that were
not talked about: response time distribution, the resources needed to run
the app and time elapsed.

### Response Time Distribution

Basically, you want most responses to be < 800ms, so if lots of responses are greater
than that, you're delivering a worse user experience.

### App CPU and Memory

Languages with a runtime require more cpu and memory. The runtime is another
program that is competing for resources with your application. There's no free
lunch.

Why is this important?
  - You can give the extra cpu and memory to other resources;
  - Save money on your cloud provider bill.

### Time Elapsed

This one is connected with `Response Time Distribution`.

Some solutions are able to finish in ~210 seconds, but some need more than
250 seconds.

# Conclusion

There are many things that affect this benchmark, so the claim in the original
post about Java being faster than Rust is misleading.

  - The programs have different architecture;
  - Differences in the database schema;
  - Different resource (memory/cpu) distribution;
  - Different database configuration;
  - Different nginx configuration;
  - Different docker base image;

Besides, I was not able to reproduce his result.<br>
The Java version actually ran much worse on my computer, which is another thing
that affects benchmark ...<br>

Benchmarking is hard and takes time ..., but I learned a lot doing it =)

The app itself is simple, just 4 endpoints, but the amount of possible configurations is big!

And as the workhorse for this benchmark was actually PostgreSQL, if there's a
language winner here, it is C! =D

![PostgreSQL](/assets/images/postgres-C.png)

You can find some tests that I did here:
[https://github.com/tivrfoa/rinha-backend-2023-rust-axum-sqlx/commits/playing-with-docker-resources](https://github.com/tivrfoa/rinha-backend-2023-rust-axum-sqlx/commits/playing-with-docker-resources) and here [https://github.com/tivrfoa/rinha-backend-2023-rust-axum-sqlx/commits/use-hashmap-gambiarra-xD](https://github.com/tivrfoa/rinha-backend-2023-rust-axum-sqlx/commits/use-hashmap-gambiarra-xD)


# References

[Os Resultados da RINHA DE BACKEND estão ERRADOS, e eu posso provar](https://www.youtube.com/watch?v=XqYdhlkRlus)

## PostgreSQL

https://github.com/docker-library/postgres/issues/193

https://stackoverflow.com/questions/30848670/how-to-customize-the-configuration-file-of-the-official-postgresql-docker-image

https://stackoverflow.com/questions/47252026/how-to-increase-max-connection-in-the-official-postgresql-docker-image

https://www.postgresql.org/docs/8.1/gist.html

https://www.postgresql.org/docs/current/pgtrgm.html

https://www.postgresql.org/docs/9.1/textsearch-indexes.html

https://stackoverflow.com/questions/60193781/postgres-with-docker-compose-gives-fatal-role-root-does-not-exist-error

https://stackoverflow.com/questions/12452395/difference-between-like-and-in-postgres

## MySQL

https://planetscale.com/learn/courses/mysql-for-developers/schema/generated-columns



## Nginx

https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching

https://stackoverflow.com/questions/63077100/how-much-memory-and-cpu-nginx-and-nodejs-in-each-container-needs

https://www.nginx.com/resources/wiki/start/topics/examples/full/

https://www.nginx.com/blog/avoiding-top-10-nginx-configuration-mistakes/

https://www.nginx.com/blog/http-keepalives-and-web-performance/

https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive

## sql_builder

https://docs.rs/sql-builder/latest/sql_builder/

## Docker

https://hub.docker.com/_/ubuntu/tags

https://stackoverflow.com/questions/25845538/how-to-use-sudo-inside-a-docker-container

https://stackoverflow.com/questions/32804643/how-to-create-a-copy-of-exisiting-docker-image

https://stackoverflow.com/questions/35231362/dockerfile-and-docker-compose-not-updating-with-new-instructions

https://github.com/peter-evans/docker-compose-healthcheck

```
docker inspect rinha-backend-rust_nginx_1 | egrep -i '(Memory|cpu)'
```

## Java

https://docs.aws.amazon.com/corretto/latest/corretto-20-ug/downloads-list.html

## Rust

https://docs.rs/speedate/latest/src/speedate/date.rs.html#101-103

## Rust Postgres

https://www.craft.ai/post/implement-postgresql-pool-connection-tokio-runtime-rust

https://github.com/launchbadge/sqlx/issues/102

https://medium.com/@edandresvan/a-brief-introduction-about-rust-sqlx-5d3cea2e8544

## Sqlx

https://medium.com/@edandresvan/a-brief-introduction-about-rust-sqlx-5d3cea2e8544

https://codevoweb.com/rust-build-a-crud-api-with-sqlx-and-postgresql/

## Axum

https://docs.rs/axum/latest/axum/response/index.html

https://stackoverflow.com/questions/74891118/how-to-use-both-axumextractquery-and-axumextractstate-with-axum

https://stackoverflow.com/questions/73146289/how-can-i-get-an-axum-handler-function-to-return-a-vec

https://docs.rs/sqlx/latest/sqlx/enum.Error.html

https://github.com/launchbadge/sqlx/blob/main/examples/postgres/axum-social-with-tests/src/http/post/comment.rs

https://codevoweb.com/rust-crud-api-example-with-axum-and-postgresql/#google_vignette

# Running the stress test

```sh
curl -o gatling.zip https://repo1.maven.org/maven2/io/gatling/highcharts/gatling-charts-highcharts-bundle/3.9.5/gatling-charts-highcharts-bundle-3.9.5-bundle.zip
unzip gatling.zip
mkdir ~/gatling
mv gatling-charts-highcharts-bundle-3.9.5 ~/gatling/3.9.5
```

# Validating Requirements Using httpie


## POST pessoas

```sh
echo -n '{
  "apelido" : "josé",
  "nome" : "José Roberto",
  "nascimento" : "2000-10-01",
  "stack" : ["C#", "Node", "Oracle"]
}' | http POST localhost:9999/pessoas
```

Response:

```
HTTP/1.1 201 Created
Connection: keep-alive
Content-Length: 0
Date: Thu, 14 Sep 2023 10:41:34 GMT
Location: /pessoas/dcf1436d-d3f3-4a7e-a6ba-bc839f51da3c
Server: nginx/1.25.2
```

## GET pessoas/[:id] - Querying person

http localhost:9999/pessoas/dcf1436d-d3f3-4a7e-a6ba-bc839f51da3c

Response:

```json
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 143
Content-Type: application/json
Date: Thu, 14 Sep 2023 10:45:14 GMT
Server: nginx/1.25.2

{
    "apelido": "josé",
    "id": "dcf1436d-d3f3-4a7e-a6ba-bc839f51da3c",
    "nascimento": "2000-10-01",
    "nome": "José Roberto",
    "stack": [
        "C#",
        "Node",
        "Oracle"
    ]
}
```

## GET contagem-pessoas

http localhost:9999/contagem-pessoas

Response:

```
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 1
Date: Thu, 14 Sep 2023 10:52:32 GMT
Server: nginx/1.25.2

1
```

## GET /pessoas?t=[:termo da busca]

http localhost:9999/pessoas?t=Oracle

```json
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 145
Content-Type: application/json
Date: Thu, 14 Sep 2023 21:42:02 GMT
Server: nginx/1.25.2

[
    {
        "apelido": "josé",
        "id": "dcf1436d-d3f3-4a7e-a6ba-bc839f51da3c",
        "nascimento": "2000-10-01",
        "nome": "José Roberto",
        "stack": [
            "C#",
            "Node",
            "Oracle"
        ]
    }
]
```
