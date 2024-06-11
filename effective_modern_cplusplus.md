# Effective Modern C++ Notes

WIP

### Unique Ptrs
* auto_ptr is unique_ptr for C++ 98 when move semantics did not exist. unused in modern C++
* same size as raw ptr
* exclusive ownership semantics. non-null unique ptr always owns the obj it points to
* moving a unique_ptr transfers ownership, so unique ptr is a move-only type
* upon destruction, destroys its resource
* use case can be factory function return type for objects in a hierarchy
* custom deleter objs, but must define as part of unique ptr (ex. unique_ptr<Obj, customDeleter>)
* cannot assign raw ptr to unique ptr. reset can be used instead
* deleting a derived class object via a base class ptr requires base class to have virtual destructor!
* array, vector, string are better than raw arrays

### Shared Ptrs
* lifetime managed by pointers through shared ownership
* when last shared_ptr stops point to an obj, the shared_ptr destroys the obj it is pointing to
* shared_ptrs consult a resource's reference_count to track how many shared_ptrs pointing to it
* sp1=sp2 points sp1 to the object pointed to by sp2. decrements sp1 obj reference_count and increments sp2 reference count
* Performance
    * twice the size of a raw ptr (raw ptr to resource + raw ptr to resource's reference_count)
    * reference_count is dynamically allocated
        * avoid this cost by using make_shared
    * reference count changes are atomic, so reading and writing is more costly
* reference_count is part of a data structure called control block
* control blocks are created during make_shared, shared_ptr created from unique_ptr, shared_ptr constructor is called with a raw ptr
* AVOID creating more than one shared_ptr from the same raw ptr b/c this creates two control blocks, leading to undefined behavior (ex. try to destroy the same obj twice due to two diff reference_counts in two control blocks)
* In general, avoid passing raw_ptr to shared_ptr constructor, use make_shared instead OR pass definition of raw ptr (ex. new Widget) directly into shared_ptr constructor to avoid re-using same raw ptr for another shared_ptr constructor
* enable_shared_from_this -> template for base class to create shared_ptr from this ptr
* moving a shared_ptr is faster than copying since reference_count does not need to be updated
* Specifying custom deleter doesn't change size of shared_ptr
* unique_ptr -> shared_ptr is easy, but reverse is not possible, even if reference_count is 1. If exclusive ownership is possible, go with unique_ptr

### Weak ptrs
* special type of shared ptr for pointers that can dangle (obj gets destroyed while ptr is pointing to it)
* cannot be dereferenced, increment smart ptr counter, or tested for nullness
* created from shared_ptrs, point to same obj, but don't affect reference_counter
* weak ptr expired function can check if obj has been destroyed
* What if we want to check and return dereferenced obj? 
    * atomic operation. otherwise a thread can destroy a shared_ptr, causing obj to be destroyed after the check has occurred on another thread
    * create shared ptr from weak ptr using weak ptr lock function (returns null if expired) OR pass weak ptr into shared_ptr constructor (if expired, exception is thrown)
* use case:
    * store weak_ptrs in a cache while shared_ptrs are dealt with
    * subjects (state may change) hold container of weak ptrs to observers (notified by subject state change) to figure out if observers have been destroyed
    * Consider A <-> B -> C. A -> B and B -> C is shared_ptr. What should B -> A be?
        * If raw_ptr, if A destroyed, then B -> A is dangling which is bad
        * If shared_ptr, A <-> B ptrs cannot be destroyed b/c reference_count is 1 for both. Leak.
        * If weak_ptr, if A destroyed, then B -> A can check if dangling first
* same as shared_ptr efficiency_wise



