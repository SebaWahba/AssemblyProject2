* * * * *

CSCE 231 / 2303 -- Computer Organization\
Spring 2025 -- Project 2

Cache Performance Evaluation Report
===================================

Group Members:

-   Seba Ahmed Wahba -- 900225470

Section: 10:00 am WU\
Instructor: Dr Mohammed Shalan

* * * * *

* * * * *

1\. Introduction
----------------

A cache is essentially a memory storage space that was introduced to tackle a significant problem that emerged in the world of computing in the late 1980s. That problem is basically the fact that before the 1980s the processor's speed and the main memory matched each other, but soon this ended with the computational advancements causing the processor to exceed the memory's speed. This caused the memory to become a bottleneck as it slowed down the processor and the processor advancements were no longer useful. So the cache solved this issue by acting as an intermediary between the processor and the memory such that the memory can be manufactured using the DRAM which is dense,slower, cheap and available in large sizes, while the cache is made out of the SRAM which is less dense, faster, expensive and available in small sizes. Now, the processor can get the data from the cache and if it finds it then it is a hit, if not then it is a miss and data is fetched from the memory with a miss penalty.

* * * * *

2\. Cache Simulator Design
--------------------------

Data structures: CacheLine, Cache class

I made use of the LRU replacement policy which checks for the least recently used data and replaces it with the data fetched from the main memory when the cache's capacity is full (Capacity Cache). This is done by adding an additional bit to each line in the cache called the counter bit which keeps track of the order of accessing each line such that in the end we can tell which line has the minimum counter bit and so it will be the least recently accessed line of data.

Cache parameters:

-   Fixed cache size = 64 KB

-   Line size: 16, 32, 64, 128 bytes

-   Associativity (ways): 1, 2, 4, 8, 16

How each component is calculated:

Offset: We calculate the log of the line size base 2.

Index bits: We calculate how many sets we have, and how many bits we need to pick a set. 

As for the actual index, it makes use of the index bits that were calculated. They are shifted to the right to remove block offset bits (they're not needed to find the set). Then we use a bitmask to extract just the index bits.

Now moving on to how each memory generator functions:

memGen1: It Sequentially walks through memory, Type and has a high spatial locality, It is also great for testing how well the cache handles predictable access

memGen2:  Random access in a very small region (24 KB), type and has a temporal locality, It is also good for testing small working sets that fit in cache.

memGen3:  Random access across the entire 64 MB, type and has a No locality, It also Simulates worst-case for cache (everything is a miss).

memGen4: Sequential access in a very small range (4 KB), type and has a very high spatial + temporal locality, It should result in near 100% hit rate after warm-up.

memGen5:  Sequential access in a 64 KB region, Medium spatial and temporal locality,Tests how cache handles a working set that matches cache size.

memGen6: Strided access, skips 32 bytes each time,Breaks spatial locality for small line sizes, Tests if line size is large enough to cover stride (e.g., stride = 32, so line size 64 or 128 helps).

* * * * *

3\. Experiment Setup
--------------------

I ran 1,000,000 memory accesses per test
----------------------------------------

Used 6 memory generators

Two experiments:\
Experiment 1: Fixed 4 sets, varied line size\
Experiment 2: Fixed line size = 64, varied ways (1--16)

C++ was used for cache simulation and Google Sheets was used for plotting, graphing and recording results.

To make sure the simulator works correctly I ran 2 test cases one for directly mapped cache and the other for 2-way set associative cache each with 4 or 5 addresses to be tested for hits and misses. I also ran 2 other tests to test 4-way set associative cache and fully associative cache and they also worked correctly.

Test 1: Direct-mapped, same address reuse

Access 0x0: Expected MISS, Got MISS

Access 0x40: Expected MISS, Got MISS

Access 0x0: Expected HIT, Got HIT

Access 0x10000: Expected MISS, Got MISS

Access 0x0: Expected MISS, Got MISS

Test 2: 2-way associativity

Access 0x0: Expected MISS, Got MISS

Access 0x8000: Expected MISS, Got MISS

Access 0x0: Expected HIT, Got HIT

Access 0x8000: Expected HIT, Got HIT

Test: Fully Associative Cache

Access 0x0: Expected MISS, Got MISS

Access 0x40: Expected MISS, Got MISS

Access 0x80: Expected MISS, Got MISS

Access 0xc0: Expected MISS, Got MISS

Access 0x100: Expected MISS, Got MISS

Access 0x0: Expected HIT, Got HIT

Access 0x40: Expected HIT, Got HIT

Access 0x80: Expected HIT, Got HIT

Access 0xc0: Expected HIT, Got HIT

Access 0x100: Expected HIT, Got HIT

Test: 4-Way Set-Associative Cache (No Eviction)

Access 0x0: Expected MISS, Got MISS

Access 0x4000: Expected MISS, Got MISS

Access 0x8000: Expected MISS, Got MISS

Access 0xc000: Expected MISS, Got MISS

Access 0x0: Expected HIT, Got HIT

Access 0x4000: Expected HIT, Got HIT

MemGen1:

Access pattern: Sequential through full DRAM range.

Behavior: Very low temporal locality but high spatial locality for small line sizes.

Expected results:

-   Exp 1 (Vary line size): Hit ratio increases as line size increases (more spatial locality).

-   Exp 2 (Vary associativity): Little improvement from associativity---still mostly cold misses due to lack of temporal locality.

memGen2:

Access pattern: Random addresses within 24KB range.

Behavior: Moderate temporal locality (due to reuse within a small range).

Expected results:

-   Exp 1: Moderate improvement with increasing line size.

-   Exp 2: Better hit ratios with increasing associativity, due to better conflict resolution.

memGen3:

Access pattern: Completely random across full DRAM.\
Behavior: Very little locality (temporal or spatial).

Expected results:

-   Exp 1: Line size has almost no effect. Always cold misses.

-   Exp 2: Associativity won't help much. Still low hit ratio (~0-1%).

memGen4:

Access pattern: Sequential within a small 4KB region.

Behavior: Very high spatial locality, and some temporal locality.

Expected results: 

Exp 1: Dramatic improvement with line size. Near 100% hit with large lines.

-   Exp 2: High hit ratios even with low associativity due to small working set.

memGen5:

Access pattern: Sequential within 64KB region.

Behavior: Spatial locality, but working set size equals cache size --- makes eviction more likely.

Expected results:

-   Exp 1: Some improvement with larger lines.

-   Exp 2: Moderate hit ratios; better with more associativity.

memGen6:

Access pattern: Strided access (stride = 32 bytes).

Behavior: Regular stride---can lead to cache set conflicts (especially in direct-mapped or low-associativity).

Expected results:

-   Exp 1: May benefit from increasing line size.

-   Exp 2: Associativity helps significantly---higher associativity reduces conflict misses.

* * * * *

4\. Results & Graphs
--------------------

### Experiment 1: Hit Ratio vs Line Size (Sets = 4)

MemGen1:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXe8S5ZafKtAqwbCmN8AvzEhuHvgMUCi0vgXwgMjHXR4wF2TKMXG4lBPnfOjLEQY1K4g7ieos5Cf3zox4XS6X1djzSfGAdgjPxRklhlfp5lXZZw3M4Y0Shz1tLsHNPqB1W9iebIMmw?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")MemGen2:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdSOaPdC6snCJehT4juFX2t3bDJmIaq3JG8alqQGtgP5qbB3ToKV_XrENN5HzZfi9OVkG5aV9UP9Krvli4KUlzElYg_0DP-6rhOVEcBfAbiiQ0pWgxV7mK2i-ChfFI9HhPId5qY?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")

MemGen3:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXetTBVVp2oOIi9npFBvPc0K_2felO8KY_jYjH3v_Mttq28ZN6jQt8eWQM8UJ9j8-Sb5jdHbNtS5a6dr4rh81-YLXfTnXBz91T-uMQEd_YY6F_vVT94l3m5Zjdr6yGHw9K5j-QhYdA?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")

MemGen4:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdwSguL_7agczMNQa765mxLtgVoShjTiDbiOEWrTLqaEvOjtIWjSGANHuVjOD6DwetdIN2wgyTBlhP1etmpNYSjzB0LvusIMGywi8Nv9CEKDQWVQnCLkaCq1wxxntAcBpGXiYveqw?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")

MemGen5:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfdDVY-ESj-BDvPz2Oon01K40nzSKCBOStvoiJ-OY4mAR-otinDcrkSNVAWW9fOFNEY5QY7TwfdUBbHdahTt5NAUfOPCcea5dTKoMg5phWSUofFdov39AkZ9qk4zdohE_FkMVEYAw?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")

MemGen6:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeK4SgEyPqzk1L4s29ylrzHAsSYHubrjU4zUGHwMuNqXeUmY8263Zud-B1B9O4TNKeQ53-qv1sNc8fVT-iA3tBOQ9jSW6qCTsxhQgQsMGd9TSJXtBSVFbLH1ZEvHMAe2oPlwYt2?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")

|

Generator

 |

Line Size (bytes)

 |

Hit Ratio (%)

 |
|

memGen1

 |

16

 |

93.75

 |
|

memGen1

 |

32

 |

96.88

 |
|

memGen1

 |

64

 |

98.44

 |
|

memGen1

 |

128

 |

99.22

 |
|

memGen2

 |

16

 |

99.85

 |
|

memGen2

 |

32

 |

99.92

 |
|

memGen2

 |

64

 |

99.96

 |
|

memGen2

 |

128

 |

99.98

 |
|

memGen3

 |

16

 |

0.1

 |
|

memGen3

 |

32

 |

0.1

 |
|

memGen3

 |

64

 |

0.1

 |
|

memGen3

 |

128

 |

0.1

 |
|

memGen4

 |

16

 |

99.97

 |
|

memGen4

 |

32

 |

99.99

 |
|

memGen4

 |

64

 |

99.99

 |
|

memGen4

 |

128

 |

100

 |
|

memGen5

 |

16

 |

99.59

 |
|

memGen5

 |

32

 |

99.8

 |
|

memGen5

 |

64

 |

99.9

 |
|

memGen5

 |

128

 |

99.95

 |
|

memGen6

 |

16

 |

0

 |
|

memGen6

 |

32

 |

0

 |
|

memGen6

 |

64

 |

50

 |
|

memGen6

 |

128

 |

75

 |

### Experiment 2: Hit Ratio vs Associativity (Line Size = 64)

MemGen1:\
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcuGucW4pC8jWsc_hPqX3Cp2NnqKiZNL_9N7Ekw9sdm3u32A3axLFyMIZgUnUFcmQP8zlLJNMlGq-MJp7UhiQMN7lHTEprxihJkqqGb4eB9KGbnfEb7yTltX0bG7RSmg_bONko9lA?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")

MemGen2:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXc8YC_9BwGulsXRoAtPSgD4Cb_brcTtNjEHY-HXLTgKFgTUt-I4EKyHH-Qccb-Z4LtX432-vR0LXh1AQT0PEetU_HQ7ON3D8ImRTg17QsDpQ2lga3e9jhbUAUChuoImePamQVP6?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")

MemGen3:![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcxpJj7pHKFXT0oPDfYEwnt4vO0SDrSxq1p77fOZzvRYP5_I4oG7_vLvbeII8NR0agAkb29XpjaL5fE3ujGWxpBtNfM_Cu1OJ9esKR-FlRNAS7o9fYR-Li6X5r-1yIIG9b3LCGUjA?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")

MemGen4:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXecmyjYdk7dc8GICc74kgyWQNNEvmNcnIZxiaWOJV81qXTMoNLMIHxE0eNmIlGVv3awqWJu1xLKhif15UmIVNNebOMrA28mKTAQDNTOdwdpU9MYedQqpCK0oEQoIugOMXKPqs7sIg?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")

MemGen5:![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcUL8WQGt0-RHapR6SL8GaDDXVfzgQ2vXPd-qrtPbjK4ISVuR707TBTdWfaSGGTT-VTRwRXGO7njFmhkW_ZNEea4iKdyHGLb5nUw_q0xuuGkpmmiT6E3ULaVGHIpdIPKJeag3Ozdg?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")

MemGen6:![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfYh9uz-mPdiZn5qdn8tV-TCh7-wD0_MMw-TnWbUTpmjNNPUW1cGkElXQmgpD2Y9eV0DQw_wOuqVnfqf-27lJloIclEvnOOn7xB5Tlx40dMkHkOSMXJR3wLlyZhKuymrZ4_otS5qA?key=Tb5DxvC0fziRF5sBl-xLKg "Chart")

* * * * *

|

Generator

 |

Line Size (bytes)

 |

Hit Ratio (%)

 |
|

memGen1

 |

16

 |

98.44

 |
|

memGen1

 |

32

 |

98.44

 |
|

memGen1

 |

64

 |

98.44

 |
|

memGen1

 |

128

 |

98.44

 |
|

memGen2

 |

16

 |

99.96

 |
|

memGen2

 |

32

 |

99.96

 |
|

memGen2

 |

64

 |

99.96

 |
|

memGen2

 |

128

 |

99.96

 |
|

memGen3

 |

16

 |

0.1

 |
|

memGen3

 |

32

 |

0.1

 |
|

memGen3

 |

64

 |

0.1

 |
|

memGen3

 |

128

 |

0.1

 |
|

memGen4

 |

16

 |

99.99

 |
|

memGen4

 |

32

 |

99.99

 |
|

memGen4

 |

64

 |

99.99

 |
|

memGen4

 |

128

 |

99.99

 |
|

memGen5

 |

16

 |

99.9

 |
|

memGen5

 |

32

 |

99.9

 |
|

memGen5

 |

64

 |

99.9

 |
|

memGen5

 |

128

 |

99.9

 |
|

memGen6

 |

16

 |

50

 |
|

memGen6

 |

32

 |

50

 |
|

memGen6

 |

64

 |

50

 |
|

memGen6

 |

128

 |

50

 |

5\. Analysis & Discussion

-   How did increasing line size affect the hit ratio?\
    Larger line size improved hit ratio for generators with spatial locality (e.g., memGen1, memGen4), but had no effect for random access (e.g., memGen3).

-   How did associativity impact performance?\
    Associativity helped very little in most patterns; hit ratio stayed flat for almost all memory generators.

-   Which generators benefited most from larger line sizes?\
    memGen1, memGen5, memGen6 --- because they access data sequentially or with predictable strides.

-   Which access patterns saw no improvement from higher associativity?\
    memGen3 and memGen6 --- both had flat hit ratios regardless of associativity.

-   Any unexpected results?\
    Associativity had almost no impact even in strided or larger working set patterns.

Experiment 1:

memGen1: Sequential

It starts at approximately 75% because it makes use of both temporal and spatial locality as well as the fact that we iterate 1 million times and so the number of lines in the cache is almost 1 million so any accesses above 1 million would eventually lead to and increase and levelling at 100% as the cache learns from repetitions over the same addresses so we reach 100% hit rate.

memGen2: Random over 24KB range

Since this generator has a randomized access technique so no spatial or temporal locality, however it does end up with 100% hit rate because it covers only a small part of the cache so 24KB instead of 64 KB and the number of associative sets is set at 4 which is large number that allows it to cover several conflict blocks thus reducing the possibility of misses.

memGen3:

That's a fully random generator over 64MB, and with 64KB cache, you only have room for 1 in every 1024 addresses. So you're very likely to miss. Hit rate of ~0.1% is expected.

memGen4:

Full hit rate because of high associativity

Memgen5:

Full hit rate because of high associativity

memGen6:

Starts at 0 hit rate because as long as we are below 32 lines the number of lines is less than the stride and so we will never find the block we are looking for.

Once 32 is reached it increases at a constant rate and once 64 is reached which is the size of 2 strides, it accelerates in hit rate.

Experiment 2:

memGen1:

Cache size is 64 KB and thes size of this generator is 24 KB so associativity does not matter if everything fits equivalent to experiment 1.

memGen2:

Same as experiment 1 associativity does not matter once data fits

memGen3:

Same as experiment 1 associativity would not help if there is no reuse anyway.

memGen4:

Only one cold start or compulsory miss then all hits

Memgen5:

With associativity at 1 no conflicts because mapping works otherwise after a full pass everything hits.

memGen6:

Half of the times the cache line accessed is missed so cold start or compulsory and the second time all hits especially that associativity prevents or reduces conflict misses.

* * * * *

6\. Conclusion
--------------

This project demonstrated how  two key configuration parameters: line size and associativity (ways) can impact cache performance. Through a set-associative cache simulator, and experimenting with six distinct memory access patterns, we deduced the following:

-   Larger line sizes significantly improved hit ratios for access patterns with high spatial locality (e.g., memGen1, memGen4, memGen5). In strided access patterns (memGen6), performance improved dramatically once the line size exceeded the stride size.

-   Increasing associativity (ways) had minimal impact for most access patterns. In nearly all cases, moving from 1-way (direct-mapped) to higher associativity produced little to no improvement, indicating that conflict misses were rare in the tested scenarios because the problem was due to the fact that the data was not in the set itself not that it was conflicting with other data in the same set.

-   Access patterns with high temporal locality's (e.g., memGen2, memGen4) ratios rockettes regardless of cache structure, while purely random access patterns (memGen3) saw no performance gain from any configuration.

-   Surprisingly, associativity had no impact on some patterns where it might be expected to help (e.g., memGen6), reinforcing the importance of line size alignment with memory access stride.

Overall, these findings highlight that cache performance depends more on access patterns and line size than on associativity alone. Understanding a program's memory behavior is crucial when tuning cache parameters for optimal performance.

* * * * *
