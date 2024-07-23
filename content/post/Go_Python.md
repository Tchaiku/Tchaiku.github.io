---
date: "2024-07-23"
lastmod: "2024-07-23"
draft: false
tags:
- Python
- Go
- Weiqi
- Baduk
- breadth-first search
- Algorithm
  
title: "Go in Python I: Liberties"
slug: go-python-1
editPost:
  Text: Source

summary: Yet another practice in Python.

description: This is a practice of implementing the Go/Baduk/Weiqi game in Python.

author: M. T.
# cover:
#   alt: "A real Galton board."
# #   caption: <text>
#   caption: "Refer to this if you're not sure about some parameters in the code. Note that the 'size' is always an even number."
#   image: "https://upload.wikimedia.org/wikipedia/commons/thumb/7/7a/Tabuleiros_de_Galton_%28antes_e_depois%29.jpg/1920px-Tabuleiros_de_Galton_%28antes_e_depois%29.jpg"
#   relative: false
---

{{< notice note >}}
This is a work in progress. I am by no means an expert in either Go or Python, so don't be too harsh on me if you find some mistakes. But you can point out anything in the comment section.
{{< /notice >}}


## What is Go?

First of all, If you're really unfamiliar with even the name, this post may not be for you. You can take a look at the [Wikipedia page](https://en.wikipedia.org/wiki/Go_(game)) to get some basic ideas.

Some of you may have heard of it from the [famous match](https://en.wikipedia.org/wiki/AlphaGo_versus_Lee_Sedol) between AlphaGo and Lee Sedol. And you may be wondering if this game would be hard to implement in Python. Nah, I guess the hardest part is only about the AI, that is, how to decide which move to go, among the many valid moves. But we are not going that far. A basic implementation of the game is quite a standard practice in Python. 

Let start!

I have no plan to implement all of the game today. At this moment let's quickly go thorough several ideas playing a center role in the logic of the game.

### Basic setting

- Assuming the board is $n\times n$, where n is a positive integer, for example $n=9, 13, 19.$ Now let's denote the set $\{1,\dots,9\}$ by $N$.
- A state or configuration is, mathematically speaking, a mapping $\phi$: $N\times N$ to $\{-1, 0, 1\}$, where $-1$ means black, $1$ means white, and $0$ means no stones. Note that is just a definition, and not all such states are legal. Therefore:
- A *valid state* is only proper subset of all states. The rules are too detailed as it's about the *different rule sets* of the game (Chinese rules, Japanese rules...). 

### Key ideas

so at this moment let's just list some most important ones:

 1. Stones which are neighbors and of the same color are called clusters (that is, a *connected component*, to put it mathematically). When counting liberties, stones in the same cluster are regarded as sharing the same liberties.
 2. A cluster having no liberties is called dead, and they *cannot appear in a valid state*.
 3. Therefore, the liberty of a given single stone is defined to be the liberty of the cluster it belongs to.

Now let's translate the 3 things listed to programming language. We need to:

1. Define a function to tell if two stones are neighbors (regardless of color). This is very easy.
2. 1. Input: a stone, Output: the cluster it belongs to. This is not easy. I believe it should has some important algorithm (I used *Breadth-first search* for this attempt, but I have no idea on if it's the best for our purpose, nor, if there exist some pre-build libraries for doing this) behind it. Naively, we may use some for-loops to do it but it can be very non-efficient. This would be a key part of the main program.
   1. Then: given a state, we need to find all clusters. This is based on the previous item.
    
3. Given a cluster, we need to count its liberties. This is less difficult than 2.

Today, let's try a little bit. First of all we need to denote a state. The most math way of denoting a state should be a matrix of $n\times n$ whose entries are $-1, 0, 1$. But in Python, we can use a list of lists to denote it, thanks to the `numpy` library.


```python
import numpy as np

# first let's look at an example of a 9x9 state.3
# we can represent the board as a 9x9 numpy array

state = np.array([[0, 0, 0, 0, 0, 0, 0, 0, -1],
                   [0, 1, 1, 0, 0, -1, -1, -1, 0],
                   [0, 0, 1, -1, -1, -1, -1, 0, 0],
                   [0, 0, 0, 1, -1, -1, 0, 0, 0],
                   [0, 0, 1, 1, -1, 0, 0, 0, 0],
                   [1, 1, 1, 1, 0, -1, 0, 0, 0],
                   [0, 0, 1, 0, 0, -1, 1, 0, 0],
                   [0, 1, 1, 0, 0, 0, 0, 1, 0],
                   [-1, 0, 0, 0, 0, 0, 0, 0, -1]])

print(state)
print(state[(0,0)])
print(state[(0,8)])
print(state[(2,2)])
```

    [[ 0  0  0  0  0  0  0  0 -1]
     [ 0  1  1  0  0 -1 -1 -1  0]
     [ 0  0  1 -1 -1 -1 -1  0  0]
     [ 0  0  0  1 -1 -1  0  0  0]
     [ 0  0  1  1 -1  0  0  0  0]
     [ 1  1  1  1  0 -1  0  0  0]
     [ 0  0  1  0  0 -1  1  0  0]
     [ 0  1  1  0  0  0  0  1  0]
     [-1  0  0  0  0  0  0  0 -1]]
    0
    -1
    1
    

To precisely implement a state would be out of today's range. Now we only need to remember this kind of structure and write a function to tell if two coordinates are neighbors. Input: $x=(x_1,x_2)$ and $y=(y_1,y_2)$, output: `True` or `False`.

First we need to check if a coordinate $(x_1,x_2)$ is valid. Two things to check: 

1. if this is a tuple of length 2; 
2. if the entries are integers of $1,\dots,9$.


```python
def is_valid_coordinate(coord):
    if isinstance(coord, tuple) and len(coord) == 2:
        if 0 <= coord[0] < 9 and 0 <= coord[1] < 9:
            return True
    return False

```


```python
# test
print(is_valid_coordinate((0, 0))) # True
print(is_valid_coordinate((0, 9))) # False
print(is_valid_coordinate((9, 0))) # False
print(is_valid_coordinate((9, 9))) # False
print(is_valid_coordinate((0, 0, 0))) # False
print(is_valid_coordinate(0)) # False
print(is_valid_coordinate((0, 0.0))) # True
```

    True
    False
    False
    False
    False
    False
    True
    

Now we do `is_neighbor`. We say two valid coordinates are neighbors if they are adjacent in the horizontal or vertical direction, that is, if $|x_1-y_1|+|x_2-y_2|=1$. (Note that one stone has no way to neighbor itself.)


```python
def is_neighbor(coord_x, coord_y):
    if not is_valid_coordinate(coord_x) or not is_valid_coordinate(coord_y):
        raise ValueError("Invalid coordinates.")
    return abs(coord_x[0] - coord_y[0]) + abs(coord_x[1] - coord_y[1]) == 1
```


```python
# test
print(is_neighbor((0, 0), (0, 1))) # True
print(is_neighbor((0, 0), (1, 0))) # True
print(is_neighbor((0, 0), (1, 1))) # False
print(is_neighbor((0, 0), (0, 0))) # False
print(is_neighbor((0, 0), (0, 2))) # False
print(is_neighbor((0, 0), (2, 0))) # False
```

    True
    True
    False
    False
    False
    False
    

Then we define a function to tell if two stones of a state is of the same color. Input: a state and two coordinates, output: `True` or `False`.


```python
def is_same_color(coord_x, coord_y, state):
    if not is_valid_coordinate(coord_x) or not is_valid_coordinate(coord_y):
        raise ValueError("Invalid coordinates.")
    # note that two coordinates should be different
    if coord_x == coord_y:
        raise ValueError("Two coordinates should be different.")
    # note that 'no stone' is not a color
    if state[coord_x] == 0 or state[coord_y] == 0:
        raise ValueError("No stone at the given coordinates.")
    return state[coord_x] == state[coord_y]
```


```python
# Test 1
try:
    print(is_same_color((3, 5), (5, 5), state)) # Expected output: False
except ValueError as e:
    print("Error:", str(e))

# Test 2
try:
    print(is_same_color((1, 6), (1, 7), state)) # Expected output: False
except ValueError as e:
    print("Error:", str(e))

# Test 3
try:
    print(is_same_color((0, 0), (1, 1), state)) # Expected output: False
except ValueError as e:
    print("Error:", str(e))

# Test 4
try:
    print(is_same_color((0, 0), (0, 0), state)) # Expected output: True
except ValueError as e:
    print("Error:", str(e))

# Test 5
try:
    print(is_same_color((0, 0), (0, 2), state)) # Expected output: False
except ValueError as e:
    print("Error:", str(e))


```

    True
    True
    Error: No stone at the given coordinates.
    Error: Two coordinates should be different.
    Error: No stone at the given coordinates.
    


```python
def is_valid_coordinates(coords, state):
    if not isinstance(coords, set):
        raise ValueError("Input should be a set of coordinates.")
        return False
    if len(coords) == 0:
        raise ValueError("Input should not be empty.")
        return False
    for coord in coords:
        if not is_valid_coordinate(coord):
            raise ValueError("Invalid coordinates.")
            return False
    return True


# the difference here is, the next function checks if all stones belong to the same player, while the previous function only checks if all stones are legal on the board, regardless of color.
def is_valid_one_color_coordinates(coords, state):
    if not isinstance(coords, set):
        raise ValueError("Input should be a set of coordinates.")
        return False
    if len(coords) == 0:
        raise ValueError("Input should not be empty.")
        return False
    color = state[next(iter(coords))]
    if color == 0:
        raise ValueError("No stone at some given coordinates.")
        return False
    for coord in coords:
        if state[coord] != color:
            raise ValueError("All stones should have the same color.")
            return False
        if not is_valid_coordinate(coord):
            raise ValueError("Invalid coordinates.")
            return False
        
    return True
```


# Neighbors: frontiers, enemy frontiers and empty neighbors

For each stone on the board, define its frontier as the all coordinates which shares the same color and is a neighbor of the stone.

To do this, it's convenient for us to define a more general function.
1. Define `find_neighbors`, where neighbors are all coordinates which are valid, regardless of the color.
2. Among the neighbors, filter the coordinates with the same color as the given coordinate. Then according to different color, we can find `find_frontiers` (which are the neighbors with the same color as the given coordinate), `find_enemy_frontiers` (which are the neighbors with the different color as the given coordinate), and `find_empty_neighbors` (which are the neighbors with no stone).


```python
def find_neighbors(coords, state):
    if is_valid_coordinates(coords, state):
        neighbors = set()
        for coord in coords:
            n = state.shape[0]
            for i in range(n):
                for j in range(n):
                    if is_neighbor((i, j), coord) and (i, j) not in coords:
                        neighbors.add((i, j))
        return neighbors
    else:
        raise ValueError("Invalid coordinates.")
```


```python
# test
print(find_neighbors({(0, 0)}, state)) # Expected output: {(0, 1), (1, 0)}
# test for a set of coordinates of many isolated stones
print(find_neighbors({(0, 0), (1, 1), (8, 8)}, state)) # Expected output: {(0, 1), (1, 0), (1, 2), (2, 1)}
print(find_neighbors({(0, 0), (1, 1)}, state)) # Expected output: {(0, 1), (1, 0), (1, 2), (2, 1)}
```

    {(0, 1), (1, 0)}
    {(0, 1), (1, 2), (2, 1), (8, 7), (1, 0), (7, 8)}
    {(0, 1), (1, 0), (1, 2), (2, 1)}
    

`find_frontiers` is nothing but the coordinates with the same color as the given coordinates in the neighbors. Only thing you need to check is, if the input set is of the same color or not.


```python
def find_frontiers(coords, state):
    if is_valid_one_color_coordinates(coords, state):
        color = state[next(iter(coords))]
        neighbors = find_neighbors(coords, state)
        
        frontiers = set()
        for neighbor in neighbors:
            if state[neighbor] == color:
                frontiers.add(neighbor)
        return frontiers
    else:
        raise ValueError("Invalid coordinates.")
```

Similarly, we can find `find_enemy_frontiers` as follows:


```python
def find_enemy_frontiers(coords, state):
    if is_valid_one_color_coordinates(coords, state):
        color = state[next(iter(coords))]
        neighbors = find_neighbors(coords, state)
        
        enemy_frontiers = set()
        for neighbor in neighbors:
            if state[neighbor] == -color:
                enemy_frontiers.add(neighbor)
        return enemy_frontiers
    else:
        raise ValueError("Invalid coordinates.")
```

Lastly, we can `find_empty_neighbors` as follows:


```python
def find_empty_neighbors(coords, state):
    if is_valid_one_color_coordinates(coords, state):
        color = state[next(iter(coords))]
        neighbors = find_neighbors(coords, state)
        
        empty_neighbors = set()
        for neighbor in neighbors:
            if state[neighbor] == 0:
                empty_neighbors.add(neighbor)
        return empty_neighbors
    else:
        raise ValueError("Invalid coordinates.")
```


```python
# test
coords = {(1,6), (1,7)}
print(find_frontiers(coords, state))
```

    {(2, 6), (1, 5)}
    

### Breadth-first search

(TODO: add some explanation on BFS here.)

```python
def find_cluster(coords, state):
    if not is_valid_one_color_coordinates(coords, state):
        raise ValueError("Invalid coordinates.")
    if any(state[point] == 0 for point in coords):
        raise ValueError("One or more starting points have no stone.")
    
    visited = set()
    visitable = find_frontiers(coords, state)
    
    while len(visitable) > 0:
        current = visitable
        visited.update(current)
        frontiers = find_frontiers(current, state)
        visitable = frontiers - visited
    
    # lastly, the cluster should include the starting points
    visited.update(coords)

    return visited

```


```python
cluster = find_cluster({(1,7)}, state)
print(cluster)

cluster = find_cluster({(1,6), (1,7)}, state)
print(cluster)
```

    {(4, 4), (2, 4), (3, 4), (1, 5), (2, 3), (1, 7), (2, 6), (1, 6), (2, 5), (3, 5)}
    {(4, 4), (2, 4), (3, 4), (1, 5), (2, 3), (1, 7), (2, 6), (1, 6), (2, 5), (3, 5)}
    

So what's the difference from empty-neighbor to liberty? Well, when the set we input is a cluster, that is, a connected component of stones, the empty-neighbor is the liberty of the cluster.

If we start from any set of stone of the same color, to get its liberties, 
1. `find_cluster`; 
2. `find_empty_neighbors`; 
3. count the number of empty neighbors.


```python
def find_liberties(coords, state):
     color = state[next(iter(coords))]
    # 1. find the cluster
     cluster = find_cluster(coords, state)
    # 2. find the empty neighbors of the cluster
     emtpy_neighbors = find_empty_neighbors(cluster, state)
    # 3. count the number of empty neighbors
     return len(emtpy_neighbors)
```


```python
print(find_liberties({(1,7)}, state))
print(find_liberties({(0,8)}, state)) 
print(find_liberties({(7,7)}, state))
print(find_liberties({(1,1)}, state))
print(find_liberties({(3,3)}, state))
```

    10
    2
    4
    6
    11
    