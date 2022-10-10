---
permalink: /Imperial%20CTF%202022/EscapeMaster/
---

# Escape Master

We are given an IP and port to connect via netcat.
It greets me with a description of the challenge and an example maze.

After starting the challenge by entering `Y`, I get a huge maze that I will have to solve in only a few seconds.
I will have to write a tool for it!

## Parsing the Maze

The most important step in solving the maze automatically is parsing it into a matrix of some sort. To better read it and automate everything I used `pwntools` to solve it.

```python
import pwn as p

conn = p.remote("192.168.125.100",9003)

conn.recvuntil(b'Ready? (Y/N)')
conn.sendline("Y")

# Getting and parsing the labyrinth by removing spaces & splitting into a matrix
lab = list(map(lambda x: list(x),conn.recvuntil("\n\n").decode().strip().replace(" ","").split("\n")))
```

## Actually solving the maze

To make it easier, I searched for a good pathfinding algorithm on the internet and found this one: [pathfinding](https://pypi.org/project/pathfinding/)</br>
I needed to convert it to a matrix of weights first so that the tool can understand it.
Lets just replace the `#`'s with 0 and everything else with 1.

```python
# A very complex way of replacing everything in the matrix
lab = list(map(lambda x: list(map(lambda y: int(y.replace("#", "0").replace("O", "1").replace("L", "1").replace("H", "1")), x)),lab))
```
And then solving it with the pathfinding package:

```python
from pathfinding.core.diagonal_movement import DiagonalMovement
from pathfinding.core.grid import Grid
from pathfinding.finder.a_star import AStarFinder

grid = Grid(matrix=lab)

start = grid.node(0, 0) # Where L was
end = grid.node(len(lab) - 1, len(lab[len(lab) - 1]) - 1) # Where H was
finder = AStarFinder(diagonal_movement=DiagonalMovement.never)
path, runs = finder.find_path(start, end, grid)
```

The path variable holds all of the moves it did to solve the maze as tuples `(0,1), (0,2), (1,2)...` but the challenge expects us to send the directions...

## Converting the moves to directions and sending the solution

```python
last = path[0]
directions = {(0, 1): "D", (0, -1): "U", (1, 0): "R", (-1, 0): "L"} # A dict that replaces multiple if statements
log = ""
for i in path:
    if last == i: # Ignoring the first run
        continue

    log += directions[tuple(map(lambda a, b: a - b, i, last))] # Subtracting two tuples
    last = i

conn.sendline(log) # Sending the directions
print(conn.recvall().decode()) # Prints the flag and the rest of the response
```

Done!
```
We sent the path to our agent. Let's see if he makes it.
You did it! Agent Lee has been extracted!
Here is a token of our appreciation:
ICTF{redacted}
```
