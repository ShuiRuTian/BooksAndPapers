Key word:
out-of-order execution, hardware level
compiler reorder, compiler level
memory barrier, hardware level, OS level, language level
memory model, hardware level, language level

Note: It might be confusing or unclear when reading, because it is too close to hardware, while some other level might also have the same/similar term. For example, about memory barrier, it depends on which level we are talking about.
This question confused me a lot: Should memory barrier cahnges cache? Cache coherency should have been guaranteed by hardware, should it? Should memory barrier flush cache or disable cache?
If we are talking on OS level, we expect that the memory barrier only incluence the memory ordering(Memory Consistency) accoring to linux doc: https://docs.kernel.org/staging/index.html#memory-barriers. C# define something similar to acquire/release barrier in language level: https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1454-volatile-fields. In hardware level, some books distinguish Memory Consistency and Cache Coherence(https://www.morganclaypool.com/doi/abs/10.2200/S00346ED1V01Y201104CAC016), this makes sense and make things more clear. However, in hardware implementation, memory barrier might be related to cache-coherency(http://www.puppetmastertrading.com/images/hwViewForSwHackers.pdf)

Memory barrier and volatile: https://stackoverflow.com/questions/1787450/how-do-i-understand-read-memory-barriers-and-volatile

LINUX KERNEL MEMORY BARRIERS: https://docs.kernel.org/staging/index.html#memory-barriers

Java memory model FAQ: http://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html

cache coherence: https://en.wikipedia.org/wiki/Cache_coherence

out-of-order execution and one core:
https://stackoverflow.com/questions/59217821/why-memory-reordering-is-not-a-problem-on-single-core-processor-machines
https://stackoverflow.com/questions/28285943/memory-barrier-on-single-core-arm

memory-barriers and cache-coherency:
https://stackoverflow.com/questions/30958375/memory-barriers-force-cache-coherency

Memory Barriers: a Hardware View for Software Hackers:http://www.puppetmastertrading.com/images/hwViewForSwHackers.pdf

A Primer on Memory Consistency and Cache Coherence: https://www.morganclaypool.com/doi/abs/10.2200/S00346ED1V01Y201104CAC016