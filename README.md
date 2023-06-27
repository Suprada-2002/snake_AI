# Snake Game using A * search Algorithm

A* search is the most commonly known form of best-first search. It uses heuristic function h(n), and cost to reach the node n from the start state g(n). A* search algorithm finds the shortest path through the search space using the heuristic function. This search algorithm expands less search tree and provides optimal result faster. A* algorithm is similar to UCS except that it uses g(n)+h(n) instead of g(n).

In A* search algorithm, we use the search heuristic as well as the cost to reach the node. Hence we can combine both costs as following, and this sum is called as a fitness number

```f(n) = g(n) + h(n)```
where,
```f(n)``` = estimated cost of the cheapest solution.
```g(n)``` = cost to reach node n from start state.
```h(n)``` = cost to reach from node n to goal node
