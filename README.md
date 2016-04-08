# A Magic Rope is a Better Dynamo

(Note that "Rope" is used in this document to be synonymous with https://en.wikipedia.org/wiki/Rope_(data_structure) except that "String" is no more meaningful here than it is in SSTable (Static "string" table). A magic rope is a non-string-encoding rope with additional useful properties described below.

#### A total ordering over what?
Assertions of fact. 

#### What is the total ordering of assertions of fact?

* Domain (keyspace)
 * Table
     * Copy/Replica(?)/Functionality(oltp vs olap)
         * Additional hierarchy of locality, which could include NUMA domain, etc.
             * Partition
                 * Row/Value (going to ignore the fact that Cassandra is anything other than a K/V store of rows temporarily, but cassandra partitions being strongly ordered extends this perfectly)

Total ordering within a row might exist for historical reasons, but is immaterial to the model as you will never subdivide a row.

Segments have owning nodes/services (those two terms will be used interchangeably here). Various mutual exclusion rules can be added to ensure that an owner doesn't have segments that would violate rack-awareness/etc.

#### Minimum per-range-owner(equivalent to vnode) functionality to maintain a well balanced Rope?
* pub fn balance_segments(&mut self, domain_rope:MagicRope<DSegment>) -> MagicRope<DSegment>;
* fn oversized_segments(&self) -> Vec<DSegment>
* fn undersized_segments(&self) -> Vec<DSegment>

#### Minimum API exposed by the rope itself:
* pub fn key
* pub fn get_segment(&self, key:Key) -> DSegment;

Visualization Steps

1. Construct a total ordering of all of the atoms in your data universe. In a distributed key value store, those atoms are individual copies of your k/v pairs. Making a distinction between the abstract notion of KV=K->V and a particular recording of a particular K/V pair
2. Take that total ordering, and lay it out on a bounded or unbounded Rope.
3. Back the rope by a distributed persistent Finger Tree.
4. Have each node have agency over its own load and be able to dynamically shed or acquire *portion* of ranges (since they are ordered, it's just sending the portion and moving a pointer internally).

