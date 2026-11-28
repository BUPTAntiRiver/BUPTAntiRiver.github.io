Dijkstra tells us in a _graph_, from a starting point, what is the shortest path to all other vertices.

The algorithm works like this:

1. Build a distance map that tracks the shortest path length from start to that vertex, and initialize them with infinity first.
2. Build a previous vertex map to track the path information. Initialize with `null` first.
3. Use a _priority queue_ to track which vertex to process, the priority is the distance to that vertex, we have `(0, 0)` for starting point `0`, that is the initialization.
4. Now we start updating, get the vertex to visit from priority queue, note that we may visit same vertex many times, so check whether its _priority is greater than the distance_ to it, if it is, then it means we must not be visiting it for the first time, so skip such cases.
5. Then iterate over the vertex's neighbors, check whether the new distance `distance[current] + edge_weight` is smaller than the older one `distance[neighbor]`. If it is, then update, and push that vertex into priority queue.
