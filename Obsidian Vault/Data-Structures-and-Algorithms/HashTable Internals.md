# Introduction
This series dives deep into the amazing world of Hash Tables and covers the construction and the internals of it with a practical approach. The Goal is to write it your own implementation in your favorite language (Rust). Originally inspired from a twitter thread where you gauge the experience of a senior engineer by the extent of knowledge on this particular data-structure.. 

# Basics & Performance of a Hash Table 

Hash tables store key-value pairs and support constant time operations like insertion, update, and deletion.  The internal structure involves mapping application keys to hash keys through hashing functions. By mapping hash keys to a smaller range, hash tables become efficient and avoid excessive memory usage. Dynamic resizing of the array allows for accommodating varying numbers of keys. 

Every language has it's own implementation such as:

Go - Maps.  Python - Dictionary 

It's used for fundamental base for constructing stuff like class and it's members and looking up variables by a programming language's runtime.

*Example*: 

- Classes in PL
	Take a class microService and a method callAPI. When you use something like microService.callAPI, the language needs to know which is the object and which is the method and execute that exact function.
	
	(object : method -> function )

-  Variable Lookup Tables aka Symbol Tables
	 Take this code snippet:
	 
	 var name type =  value
	 fmt.println(name)
	 
	 the language needs to know where name is located and guess how it knows.


**Hash Tables are constructed in two phases**:
1. Key -> Hash over a range of [ 0, n )
2. Hash -> Value over a smaller range of [ 0, m )

Take a example:  string(customerID) : int(69420). How do you store a string with an int? 

We map the incoming key of limited type over a wide range of int (2^32 for example). It's limited to specific types such as string, tuple, int, etc.  You can't put a list as a key in python but it's perfectly A-OK as a value. 

You can use custom types as keys with a hash function which most languages give you the ability to write your own hash function. An example in python is you want to place a object of a class as the key in a HashTable, this class would need a __ hash __  which returns an integer for the object that becomes the hash-key.

a mathematical model would look like:  string -> f -> int over a wide range of domain [ 0, 2^32 )

**Basic Approach**:

We already have int *i* from key *k* and by basically storing the value *v* at index i, we get O(1) operation as everything is in memory however this applies only when N is small. 

Imagine:
N = 10   ->   4 x 10 = 40B
N  = 100 ->  4  x 100 = 400B
N = 1,000,000 -> 4 x 1,000,000 = ~4MB

Now you apply N over i32 range, 4 x 4 Billion = ~16 GB

Where does the computer even go for such a big contiguous memory? and it's mostly going to be wasted space as most of them will be empty.

**This is why Phase 2 is important**. Given there are *k* keys that are needed to be stored, we can set an array of size *m* such that *m* ∈ O(*k*). This prevents us from creating an array of i32 and basically finding the sweet spot how big the array should be. This varies in Hash Table implementations.

We map the hash-key in range of [ 0, 2^32 ) -> smaller range [ 0, m ) where m is the length of an array. 

For example, Let's say we gotta store 5 keys in the Hash Table. We can take a bin (holding array) of size 10. For every key, we get an hash-key over a range of (0,2^32) and this is mapped to a smaller range from (0,8).

imagine the holding array of index 0-9

customer-uno -> f -> 69420 -> 0 - index 0
customer-duo -> f -> 42069 -> 4 - index 4
customer-tres -> f -> 69420 -> 6 - index 6

When you want to add more keys than the bin holds, the bin is doubled and given it to the next person (no, seriously) : 

*m* -> 2 x *m* (just your standard implementation, you can vary it to your liking).

By just re-allocating and re-indexing which might take a slight pause, the Hash Table allows more insertions now. This make the process so more efficient and we don't even need to alter the hash function.

Why even bother hashing?  for starters, it's optimal to do int -> int conversion due to the design of the Hash Table. Plus great abstraction such as it enables support for custom data types as keys - take a class of User and you want to add the object of User as a key, all you need is to implement the hash function and it gives you an int. 

------------------------------------------------------------------------

Hash Tables do give constant time performance where there are 0 collisions BUT in real production systems it's not the case. As the Hash table gets filled, the performance starts to degrade. This is where Load Factor comes in.

Load factor α = n / m,

where n = number of elements in the hash table & m = slots in the hash table

Good co-relation between α and time. Performance will go down anyway in both cases before the operations start to degrade.

Let's take the different resolution strategies where Load Factor comes  in: 

1. **Chaining**: The table can never be filled since we can always add a new node to the list and add the key. 
   
   Load Factor = Average number of elements stored per linked list
   
   Time to Resolve Collision= O( 1+α ), where 1 for adding the node and α to traverse and resolve the conflicts. This is just Upper Bound and we can 100% optimize this.

2. **Open-Addressing**: The table will eventually run out of space and the writes would fail since there's no place to add the key.
   
   Collisions on one slot will interfere with probe on beginning on other that starts with another making analysis challenging. Let's take an example:
   
   Take 3 Keys - *A1*, *A2*, *A3*. Let *A1* be hashed and get placed at index 2,  when *A2* gets hashed at index 2 - it'll have to move to the next slot since *A1* already occupied slot 2. Similarly when *A3* gets hashed at index 3, it'll have to move to the next slow since *A2* is occupying at Slot 3 & so on.
   
   Two common ways to deal with collisions in Open Addressing is **Linear Probing** and **Quadratic hashing**. If one were to plot the number of probes ( how many slots to move before finding a free slot or do a key-lookup upon collision ) against hits and misses, The graph would turn our Exponential. 
   
   This means as the table keeps getting filled, the number of probes keeps increasing exponentially. Linear probing has a problem of cluster collisions and more look-ups due to poor hashing hence resulting in worse performance while Double Hashing, you use two hash functions making it slightly better. 

Cost of a single probe matters  ( how expensive is it to find the next slot if the current slot is already allotted ).  

1. Chained Hashing - Costly AF: Linear traversal of linked list combined with doing random access of the memory since the node can be allocated anywhere in the memory making it very cache (CPU) unfriendly.  
2. Double Hashing - CPU intensive and time consuming since it requires us to evaluate 2 hash functions. Not cache friendly due to random jumps in array.

Optimal strategy depends on use-case. Tune and Evaluate according to the load.  How do you compare and benchmark against all strategies?

 **Lookup-time as a function of load**: How long does it take to perform a key lookup as the load on Hash Table increases.

Take a Hash Table of size 1024 and insert n elements of varying 32 to 900. Lookup 1000 random keys and measure the high miss ratio. Do it maybe million times and you can see how the lookup-time changes. 

Things to consider:
1. **Performance for Open Addressing degrades as α -> 1:** As number of elements equals number of slots in the array thus lots of lookup to find an empty slot.
2. **Chained Approach degrades gracefully**: As you keep adding to the auxiliary data-structure, the change won't be drastic as you're not bounded by number of slots of the Hash Table. 
3. **Linear Probing would be slower than Double Hashing** : Lookup time increases when you have large number of collisions. Double Hashing jumps randomly so not many clustered collisions while Linear probing does see clustered collisions.
4. **Probes of Double Hashing will be shorter**: Since Double Hashing leverages entire slot of Hash Table, the number of probes per collision will much shorted compared to Double Hashing.

NOTE: Chained Hashing ! >> Open Addressing. It isn't cache-friendly when the tables are large ( Chained Hashing outperforms Open Addressing when tables are smaller ).

**Improving Cache-performance in Chained Hashing:**
To leverage cache in chained hashing, we allocate a pool for each slot so that the nodes of the list are close to each other in memory. Think of it like a Linked List of Arrays. 

# Conflict Resolution with Chained Hashing, Open Addressing & Double Hashing

ggg

# Linear & Quadratic Probing
In this video, I delved into the concept of linear probing in hash tables. Linear probing is a simple and efficient way to handle hashtable collisions by searching for the next available slot in a linear manner. I explained how hash table operations work with linear probing, including adding, looking up, and deleting keys. Linear probing leverages CPU cache for faster access and aims for constant time performance in average cases. However, challenges like bad hash functions and clustered collisions can impact its efficiency.

In the previous video, we explored quadratic probing as an alternative to linear probing to handle hash table collisions more effectively. Quadratic probing uses a quadratic function to find the next available slot, reducing clustered collisions compared to linear probing. While it offers better spacing for collided keys and leverages CPU cache, it is not immune to clustered collisions and lacks the same level of cache optimization as linear probing. Understanding these nuances helps in deciding between linear and quadratic probing for hashtable implementations.
# Resizing Hash Table at Maximum Capacity
ggg

# Implementing Hash Sets, Hash Maps + Resizing
gggg