#### 【Redis_命令】

##### Sorted Set

```shell
BZPOPMAX key [key ...] timeout
summary: Remove and return the member with the highest score from one or more sorted sets, or block until one is available
since: 5.0.0

BZPOPMIN key [key ...] timeout
summary: Remove and return the member with the lowest score from one or more sorted sets, or block until one is available
since: 5.0.0

ZADD key [NX|XX] [CH] [INCR] score member [score member ...]
summary: Add one or more members to a sorted set, or update its score if it already exists

ZCARD key
summary: Get the number of members in a sorted set

ZCOUNT key min max
summary: Count the members in a sorted set with scores within the given values

ZINCRBY key increment member
summary: Increment the score of a member in a sorted set

ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM| MIN|MAX]
summary: Intersect multiple sorted sets and store the resulting sorted set in a new key

ZLEXCOUNT key min max
summary: Count the number of members in a sorted set between a given lexicogra phical range

ZPOPMAX key [count]
summary: Remove and return members with the highest scores in a sorted set
since: 5.0.0

ZPOPMIN key [count]
summary: Remove and return members with the lowest scores in a sorted set
since: 5.0.0


ZRANGE key start stop [WITHSCORES]
summary: Return a range of members in a sorted set, by index

ZRANGEBYLEX key min max [LIMIT offset count]
summary: Return a range of members in a sorted set, by lexicographical range

ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
summary: Return a range of members in a sorted set, by score

ZRANK key member
summary: Determine the index of a member in a sorted set

ZREM key member [member ...]
summary: Remove one or more members from a sorted set

ZREMRANGEBYLEX key min max
summary: Remove all members in a sorted set between the given lexicographical range

ZREMRANGEBYRANK key start stop
summary: Remove all members in a sorted set within the given indexes

ZREMRANGEBYSCORE key min max
summary: Remove all members in a sorted set within the given scores

ZREVRANGE key start stop [WITHSCORES]
summary: Return a range of members in a sorted set, by index, with scores orde red from high to low

ZREVRANGEBYLEX key max min [LIMIT offset count]
summary: Return a range of members in a sorted set, by lexicographical range,ordered from higher to lower strings.

ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
summary: Return a range of members in a sorted set, by score, with scores orde red from high to low

ZREVRANK key member
summary: Determine the index of a member in a sorted set, with scores ordered from high to low

ZSCAN key cursor [MATCH pattern] [COUNT count]
summary: Incrementally iterate sorted sets elements and associated scores

ZSCORE key member
summary: Get the score associated with the given member in a sorted set

ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]
summary: Add multiple sorted sets and store the resulting sorted set in a new key
```

