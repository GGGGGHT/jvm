# Automatic Memory Management 
HotSpot JVM uses automatic memory management <br/>
Memroy is automatically managed by a sub-system called the garbage collector <br/>
Garbage collector is responsible for <br/>
  - Allocations
  - keeping the alive objects in memory
  - Reclaiming the space used by unreachable objects

# Generational Garbage Collection 
Memory space is divided into generations <br/>
Separate pools holing objects of different age ranges <br/>
Based on hypothesis:
  - Most allocated objects die young
  - Few references from older to younger objects exist
To take advantage of this hypothesis,heap is divided into two generations 
  - Young: small and collected frequently
  - Old: larger and occupancy grows slowly
