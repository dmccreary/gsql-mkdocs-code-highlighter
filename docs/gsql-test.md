# GSQL Syntax Highlighting Test

## Test 1

```sql
CREATE QUERY countPeople() FOR GRAPH Friends { 
    /* Initialize Seed */ 
    people = {Person.*};
    /* How many people vertices are there in the entire graph? */
    PRINT people.size();
}
```

## Test 2

```sql
CREATE QUERY topCoLiked( vertex<User> input_user,  INT topk) FOR GRAPH gsql_demo
{
  SumAccum<int>  @cnt = 0;
  # @cnt is a runtime attribute to be associated with each User vertex
  # to record how many times a user is liked.
    L0 = {input_user};
    L1 = SELECT tgt
    	FROM L0-(Liked_By)->User:tgt;
    L2 =  SELECT tgt
          FROM L1-(Liked)->:tgt
          WHERE  tgt != input_user
          ACCUM tgt.@cnt += 1
          ORDER BY tgt.@cnt DESC
          LIMIT topk;
  PRINT L2;
}
```

## PageRank Test

```sql
CREATE QUERY pageRank (float maxChange, int maxIteration, float dampingFactor)
FOR GRAPH gsql_demo
{
  # In each iteration, compute a score for each vertex:
  #   score = dampingFactor + (1-dampingFactor)* sum(received scores from its neighbors).
  # The pageRank algorithm stops when either of the following is true:
  #  a) it reaches maxIterations iterations;
  #  b) max score difference of any vertex compared to the last iteration <= maxChange.
  #   @@ prefix means a global accumulator;
  #   @ prefix means an individual accumulator associated with each vertex

  MaxAccum<float> @@maxDifference = 9999; # max score change in an iteration
  SumAccum<float> @received_score = 0; # sum of scores each vertex receives from neighbors
  SumAccum<float> @score = 1;   # initial score for every vertex is 1.

  AllV = {Page.*};   #  Start with all vertices of type Page
  WHILE @@maxDifference > maxChange LIMIT maxIteration DO
    @@maxDifference = 0;
    S = SELECT s
         FROM AllV:s-(Linkto)->:t
         ACCUM t.@received_score += s.@score/s.outdegree()
         POST-ACCUM s.@score = dampingFactor + (1-dampingFactor) * s.@received_score,
                    s.@received_score = 0,
                    @@maxDifference +=   abs(s.@score - s.@score');
    PRINT @@maxDifference; # print to default json result
  END; # end while loop
  #PRINT AllV.page_id, AllV.@score;       # print the results, JSON output API version v1
  PRINT AllV[AllV.page_id, AllV.@score];  # print the results, JSON output API version v2
} # end query
```