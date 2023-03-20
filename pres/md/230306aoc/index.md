<style>
.reveal .slides section .fragment.highlight-code-red {
    opacity: 1;
    visibility: inherit;
}
.reveal .slides section .fragment.highlight-code-red.current-fragment {
    background-color: orangered;
}
.reveal .slides section .fragment.highlight-code-bold {
    opacity: 1;
    visibility: inherit;
}
.reveal .slides section .fragment.highlight-code-bold.current-fragment {
    font-weight: bold;
    background-color: #17ff2e;
}
.hljs-variable {
    color: darkgoldenrod;
}

</style>
<div id="markdown-custom-center">HiCollege - Advent of Code</div>
<!-- .slide: class="center" style="font-size:80%; padding-left:20%; margin-top: -1%; width:80%;" data-background-transition="slide" data-background-image="images/ppt-arno-1.png" data-state="hide-footer" -->

# HiCollege <br/> Advent of Code
## Arno Lepisk
### <code style="background: inherit;">arno.lepisk@hiq.se</code> 
### <span class="twitter"><code style="background: inherit;">@arno_l</code></span> &nbsp; <span class="github"><code style="background: inherit;">f00ale</code></span>
### 2023-03-06

---
# Advent of code
* Yearly programming challange
  * One two-part problem / day 1-25 december
  * Started in 2015
* Global leaderboard - top 100 get points
  * Generally fills up in minutes, up to over an hour for harder problems
* Many participants - over 250000 in 2022
* Problems released 6AM (swedish time) each day
* You must solve part 1 before getting access to part 2

--
# Advent of code
* Problems have a solution.
  * If your program runs for to long, there most likely is a better algorithm
* Problem data is designed to be solvable 
  * In many cases you don't need to solve the general case, just the case for your input

* Most problems have a link to input data, some have the 
  inputdata (just a number/string) in the problem text

* Make sure you use numbers with enough range (applies mainly to C++ etc)

--
# This course
We will look at some techniques and general tips on how to solve 
some of these problems

NB! Solutions work, but might not always be the optimal solution

Python isn't my first language - expect strange constructs :D

(Neither is web :D) <!-- .element: class="fragment" -->

---
# How to begin?

It's the 30th of november, you've set your alarm - now what?

* create a template
* maybe some helper functions


???
You've set your alarm, you sit there in the morning - what to do?


--
# A note on parsing.
* Do whatever you feel most familiar with
  * regex
  * custom parser &leftarrow; _I generally use this_
* It's ok to be lazy - only parse what is needed

<pre class="text" style="font-size: xx-small"><code data-trim data-noescape>
Blueprint 1: Each ore robot costs 4 ore. Each clay robot costs 4 ore. Each obsidian robot costs 4 ore and 17 clay. Each geode robot costs 4 ore and 16 obsidian.
Blueprint 2: Each ore robot costs 4 ore. Each clay robot costs 4 ore. Each obsidian robot costs 4 ore and 20 clay. Each geode robot costs 2 ore and 8 obsidian.
...
</code></pre>
  * All data comes in the same order - only numbers are interesting
???
Example from 2022-19

---
# Let's start from the beginning
<a href="https://adventofcode.com/2015/day/1" target="_blank">2015-1</a>

--
# 2015-1
A challange of counting

Initialize a counter to zero, add one for each `(`, subtract one for each `)` 

--
<pre class="python" style="font-size: small"><code data-trim data-noescape>
def solve(str):
    ans1 = 0
    ans2 = None
    for c in str:
        if c == '(':
            ans1 += 1
        if c == ')':
            ans1 -= 1

    print(f'{ans1} {ans2}')
</code></pre>
--
# 2015-1 part 2

We can reuse much of the first solution, count the number 
of characters handled and just take a note of when we've entered
negative space

--
<pre class="python" style="font-size: small"><code data-trim data-noescape>
def solve(str):
    ans1 = 0
    ans2 = None
    pos = 0
    for c in str:
        if c == '(':
            ans1 += 1
            pos += 1
        if c == ')':
            pos += 1
            ans1 -= 1
            if ans1 < 0 and not ans2:
                ans2 = pos

    print(f'{ans1} {ans2}')
</code></pre>


---
# Let's make art 2019-8 
<a href="https://adventofcode.com/2019/day/8" target="_blank">2019-8</a>

--
# 2019-8 p1
We get a long string of digits - `012`

Need to split them up into layers/blocks, then count the number of 0/1/2s.

Find block with fewest 0s and multiply the number of 1s and 2s in that layer.

--
# 2019-8 p1 solution
<pre class="python" style="font-size: small"><code data-trim data-noescape>
def solve(str, width, height):
    lines = []
    start = 0
    while start < len(str):
        lines.append(str[start:(start+width*height)])
        start += width*height

    cnts = []

    for l in lines:
        cnts.append(l.count('0'))

    minidx = cnts.index(min(cnts))
    ans1 = lines[minidx].count('1')*lines[minidx].count('2')
</code></pre>

--
# 2019-8 p2
Now we need to construct the image

We can go through the output picture pixel by pixel, looking
at layer after layer until we encounter a 0 or a 1.

Finally, we can split the output picture into several lines and print it.

--
# 2019-8 p2 solution
<pre class="python" style="font-size: small"><code data-trim data-noescape>
    ans2 = ''
    for c in range(len(lines[0])):
        for l in range(len(lines)):
            if lines[l][c] == '2':
                continue
            else:
                if lines[l][c] == '0':
                    ans2 += ' '
                else:
                    ans2 += '#'
                break

    start = 0
    while start < len(ans2):
        print(ans2[start:start+width])
        start += width
</code></pre>
--
# 2019-8 p2 Output
```text
#    ###  ###   ##  #### 
#    #  # #  # #  # #    
#    ###  #  # #    ###  
#    #  # ###  #    #    
#    #  # # #  #  # #    
#### ###  #  #  ##  #### 
```
---
# Asteroids
<a href="https://adventofcode.com/2019/day/10" target="_blank">2019-10</a>

--
# 2019-10 p1

Need to find point from where most other points are visible.

First get a list of all points.

Then iterate over all points and count visible other points - but how?

Check ratios, shorten with gcd. If same ratio has been seen discard it. <!-- .element: class="fragment" -->

???

Rita 

--
# 2019-10 p1 solution
<pre class="python" style="font-size: small"><code data-trim data-noescape>
    ans1 = 0

    for p in asteroids:
        # p : (x,y)
        deltas = {}
        for o in asteroids:
            if p == o:
                continue
            d = (o[0]-p[0], o[1]-p[1])
            g = gcd(d[0],d[1])
            d = (d[0]//g, d[1]//g)
            deltas[d] = None
        if len(deltas) > ans1:
            ans1 = len(deltas)
</code></pre>
--
# 2019-10 p2

Now we need to order all asteroids in a clockwise fashion, and pick number 200.

Geometry to the rescue!

<span class="fragment">arctan (`atan2`). We also need to turn the coordinate system.</span>

Finally sort and pick element 200 <!-- .element: class="fragment" -->

--
# 2019-10 p2 solution
<div style="display: flex">
<div style="flex: 1">
<pre class="python" style="font-size: small"><code data-trim data-noescape>
    dirs = {}
    for p in asteroids:
        if p == bestp:
            continue
        dx = bestp[0]-p[0]
        dy = bestp[1]-p[1]
        g = gcd(dx,dy)
        dx = dx / g
        dy = dy / g
        v = atan2(-dx,dy)
        if v &lt; -.00001:
            v += 2 \* pi
        if (dx,dy) in dirs:
            dirs[(dx,dy)].append((v,g,p))
        else:
            dirs[(dx,dy)] = [ (v,g,p) ]
    order = [ v for v in dirs.values() ]
    for o in order:
        o.sort(key=lambda x : x[1])
    order.sort(key=lambda x : x[0])
</code></pre>

</div>
<div style="flex:1">
<pre class="python" style="font-size: small"><code data-trim data-noescape>
    destroyed = 0
    TOFIND = 200
    for o in order:
        if len(o) > 0:
            destroyed += 1
            if destroyed == TOFIND:
                lastp = o[0][2]
                ans2 = 100 \* lastp[0] + lastp[1]
                break
</code></pre>
</div>
</div>

--
# 2019-10 p2 notes
It might only be necessary to keep track of the visible asteroids - at
least my data had more than 200 visible asteroids from the beginning.

I.e. no need to handle several turns.

---
# Computer simulator
<a href="https://adventofcode.com/2016/day/12" target="_blank">2016-12</a>
--
# 2016-12 p1

A simple assembler - four instructions, four registers

We can just keep track of all registers and then do all instructions in
sequence.

-- 
# 2016-12 p1 solution
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
    cmds = [tuple(s.split(' ')) for s in input]
    idx = 0
    regs = {'a': 0, 'b': 0, 'c': 0, 'd': 0}
    while idx < len(cmds):
        (c,a,b) = cmds[idx] if len(cmds[idx]) == 3 else cmds[idx] + (None,)
        idx += 1
        if c == 'cpy':
            if a in regs:
                regs[b] = regs[a]
            else:
                regs[b] = int(a)
        elif c == 'inc':
            regs[a] += 1
        elif c == 'dec':
            regs[a] -= 1
        elif c == 'jnz':
            if a in regs:
                if regs[a] != 0:
                    idx += int(b) - 1
            elif int(a) != 0:
                idx += int(b) - 1
        else:
            print(f'Unknown command {c}')
    print(regs['a'])
</code></pre>
--
# 2016-12 p2

Same problem, different start

Just run it!

It's slow... but it terminates <!-- .element: class="fragment" -->
--
# 2016-12 analysis
<div style="display: flex">
<div style="flex: 1">
<pre class="asm" style="font-size: x-small"><code data-trim data-noescape>
cpy 1 a
cpy 1 b
cpy 26 d
jnz c 2
jnz 1 5
cpy 7 c
<span class="fragment highlight-code-bold" data-fragment-index="1">inc d
dec c
jnz c -2</span>
cpy a c
<span class="fragment highlight-code-bold" data-fragment-index="1">inc a
dec b
jnz b -2</span>
cpy c b
dec d
jnz d -6
cpy 14 c
cpy 14 d
<span class="fragment highlight-code-bold" data-fragment-index="1">inc a
dec d
jnz d -2</span>
dec c
jnz c -5
</code></pre>
</div>
<div style="flex: 1">
<pre class="asm fragment" data-fragment-index="2" style="font-size: x-small"><code data-trim data-noescape>
cpy 1 a
cpy 1 b
cpy 26 d
jnz c 2
jnz 1 5
cpy 7 c
<span class="fragment highlight-code-bold" data-fragment-index="3">add c d
cpy 0 c
nop</span>
cpy a c
<span class="fragment highlight-code-bold" data-fragment-index="3">add b a
cpy 0 b
nop</span>
cpy c b
dec d
jnz d -6
cpy 14 c
cpy 14 d
<span class="fragment highlight-code-bold" data-fragment-index="3">add d a
cpy 0 d
nop</span>
dec c
jnz c -5
</code></pre>
</div>
</div>
--
# 2016-12 optimized
```python
            elif c == 'nop':
                pass
            elif c == 'add':
                regs[b] += regs[a]
```
--
# 2016-12 transpile
<div style="display:flex">
<div style="flex:1">
<pre class="c++" style="font-size: small"><code data-trim data-noescape>
static long solve(long p) {
    long a = 0, b = 0, c = p, d = 0;
    a = 1;           // cpy 1 a
    b = 1;           // cpy 1 b
    d = 26;          // cpy 26 d
    if(c) goto lbl1; // jnz c 2
    if(1) goto lbl2; // jnz 1 5
lbl1: 
    c = 7;           // cpy 7 c
lbl3:
    d++;             // inc d
    c--;             // dec c
    if(c) goto lbl3; // jnz c -2
lbl2: 
lbl5:
    c = a;           // cpy a c
</code></pre>
</div>
<div style="flex:1">
<pre class="c++" style="font-size: small"><code data-trim data-noescape>
lbl4:
    a++;             // inc a
    b--;             // dec b
    if(b) goto lbl4; // jnz b -2
    b = c;           // cpy c b
    d--;             // dec d
    if(d) goto lbl5; // jnz d -6
    c = 14;          // cpy 14 c
lbl7:
    d = 14;          // cpy 14 d
lbl6:
    a++;             // inc a
    d--;             // dec d
    if(d) goto lbl6; // jnz d -2
    c--;             // dec c
    if(c) goto lbl7; // jnz c -5
    return a;
}
</code></pre>
</div></div>

<a href="https://godbolt.org/z/bsf3YvT8r" target="_blank" class="fragment">Clang can calculate this at compile-time!</a>

---
# Good datastructures to know
| Structure         | C++                             | Python        |
|-------------------|---------------------------------|---------------|
| indexed container | `std::vector`                   | `[]`          |
| tuple             | `std::tuple`                    | `()`          |
| set               | `std::set`/`std::unordered_set` | `set()`       |
| map               | `std::map`/`std::unordered_map` | `dict()`/`{}` |
| queue             | `std::queue`                    | `deque`       |
| stack             | `std::stack`                    | `deque`       |
| priority queue    | `std::priority_queue`           | `heapq.*`     |

There might be special cases for some (e.g. set)

--
# Optimized set
If one only needs to keep track of a small set one can use an integer and bit-operations.

---
# Mazes, mazes, mazes.
<a href="https://adventofcode.com/2016/day/13" target="_blank">2016-13</a>

Many AoC-problems handle mazes in some form, lets look at some

--
# 2016-13 p1
We have a function defining the maze:
<pre class="python" style="font-size: small"><code data-trim data-noescape>
    def popcnt(num:int):
        return num.bit_count()

    def tile(x,y,input):
        return '#' if popcnt(x*x + 3*x + 2*x*y + y + y*y + input) % 2 == 1 else ' '
</code></pre>

Now we must figure out how to traverse the maze - do a breadth-first search until we find the
target.

Use a queue and a set of visited tiles. <!-- .element: class="fragment" -->

???

rita

--
# 2016-13 p1 solution
<pre class="python" style="font-size: small"><code data-trim data-noescape>
    target = (31,39)
    q = deque()
    q.append((0,1,1))
    visited = set()
    while q:
        (s,x,y) = q.popleft()
        if (x,y) in visited:
            continue
        if tile(x,y,inp) == '#':
            continue
        if (x,y) == target:
            ans1 = s
            break
        visited.add((x,y))
        if x > 0:
            q.append((s+1,x-1,y))
        if y > 0:
            q.append((s+1,x,y-1))
        q.append((s+1,x+1,y))
        q.append((s+1,x,y+1))
</code></pre>
--
# 2016-13 p2

More or less the same, but we don't look for a specific target, but count the number
of places we visit.
--
# 2016-13 p2 solution
<pre class="python" style="font-size: small"><code data-trim data-noescape>
    q.append((0,1,1))
    visited = set()
    while q:
        (s,x,y) = q.popleft()
        if s > 50:
            continue
        if tile(x,y,inp) == '#':
            continue
        if (x,y) in visited:
            continue
        visited.add((x,y))
        if x > 0:
            q.append((s+1,x-1,y))
        if y > 0:
            q.append((s+1,x,y-1))
        q.append((s+1,x+1,y))
        q.append((s+1,x,y+1))

    ans2 = len(visited)
</code></pre>
---
# Another maze!
<a href="https://adventofcode.com/2021/day/15" target="_blank">2021-15</a>
--
# 2021-15 p1

Now we must navigate through a maze while minimizing risk (cost)

One way would be to use exactly the same approach as the previous problem, but instead
of just keeping track of which places we've visited, keep track of the cost it has taken
us to get to a specific place.

If we get to a place with a higher cost than we've visited before stop, otherwise continue.

Also, if we use a priority queue instead of a normal queue we can keep 
execution time down
--
# 2021-15 p1 solution
<pre class="python" style="font-size: small"><code data-trim data-noescape>
    # create and fille scores with large value
    scores[0][0] = 0
    q = [(0,0,0)]
    while q:
        (s,x,y) = heapq.heappop(q)
        if s > scores[y][x]:
            continue
        for (dx,dy) in [(-1,0), (1,0), (0,-1), (0,1)]:
            ny = y + dy
            if ny < 0 or ny >= MAXY:
                continue
            nx = x + dx
            if nx < 0 or nx >= MAXX:
                continue
            if s + int(data[ny][nx]) < scores[ny][nx]:
                scores[ny][nx] = s + int(data[ny][nx])
                heapq.heappush(q, (scores[ny][nx],nx,ny))

    return scores[-1][-1]
</code></pre>

--
# 2021-15 p2

Here we just have to modify the input data and then run the same algorithm

---
# Regular maze
<a href="https://adventofcode.com/2018/day/20" target="_blank">2018-20</a>

???

Most likely the problem that has went best for me. Globally 16 on part 1, 14 on part 2!
--
# 2018-20 p1
We have kind of a map to all rooms in a maze.

We have to figure out how many steps the room that is the furthest away is.

For each step we take we make a record over how far we've come, and keep
track of how many steps we've taken if we reach an unvisited room. If we come to 
a place where we can choose ways (`(`) we push our position to a stack, so we can "back up"
to that position when we reach a `|` or `)`.

When we've traversed all data we know all rooms and how many steps were needed to reach them.
We only need to look for the room with the highest number of steps.

???
NB This isn't 100% correct - but correct enough to solve this problem.
--
# 2018-20 p1 solution
<div style="display: flex"><div style="flex:1">
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
    stack = deque()
    (s, x, y) = (0,0,0)
    visited = {}
    for c in reg:
        if c in 'WNSE':
            s += 1
            if c == 'W':
                x -= 1
            elif c == 'E':
                x += 1
            elif c == 'S':
               y -= 1
            else: # c == 'N'
                y += 1
            if not (x,y) in visited:
                visited[(x,y)] = s
        elif c == '(':
            stack.append((x,y,s))
        elif c == '|':
            (x,y,s) = stack[-1]
        elif c == ')':
            (x,y,s) = stack.pop()
</code></pre>
</div><div style="flex: 1">
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
    ans1 = 0
    for st in visited.values():
        if st > ans1:
            ans1 = st
    print(ans1)
</code></pre>
</div></div>
--
# 2018-20 p2
Now we only need to look for all rooms where we need more than 1000 steps!
--
# 2018-20 p2 solution
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
    ans2 = 0
    for st in visited.values():
        if st >= 1000:
            ans2 += 1
    print(ans2)
</code></pre>

---
# Stange maths
<a href="https://adventofcode.com/2020/day/18" target="_blank">2020-18</a>
--
# 2020-18 p1
We need to parse an expression with different priority rules.

How do we parse expressions normally?

Dijkstra's algorithm - but not the most famous one?

<a href="https://en.wikipedia.org/wiki/Shunting_yard_algorithm" target="_blank">Shunting yard algorithm</a>

Produces a Reverse Polish Notation representation of an expression.

E.g. `1 + 2 * 3` becomes `1 2 3 * +`

--
# 2020-18 p1 solution
<div style="display: flex"><div style="flex: 1">
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
def parse(str):
    stack = deque()
    output = []
    for c in str:
        if c == ' ':
            continue
        if c in '\*+':
            while len(stack) > 0 and stack[-1] != '(':
                output.append(stack.pop())
            stack.append(c)
        if c == '(':
            stack.append(c)
        if c == ')':
            while True:
                tmp = stack.pop()
                if tmp == '(':
                    break
                output.append(tmp)
        if c.isdigit():
            output.append(int(c))
    while len(stack) > 0:
        output.append(stack.pop())
    return output
</code></pre>
</div><div style="flex: 1">
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
def calc(lst):
    stack = deque()
    for i in lst:
        if i == '\*':
            f2 = stack.pop()
            f1 = stack.pop()
            stack.append(f1\*f2)
        elif i == '+':
            t2 = stack.pop()
            t1 = stack.pop()
            stack.append(t1+t2)
        else:
            stack.append(i)
    return stack.pop()
</code></pre>
</div></div>

???
NB Assumes single digit numbers!

--
# 2020-18 p2
Now we "just" need to handle priorities, a slight alteration to the algorithm.
--
# 2020-18 p2 solution
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
def parse(str, plusprio = 1):
    stack = deque()
    output = []
    ops = { '*':1, '+': plusprio}
    for c in str:
        if c == ' ':
            continue
        if c in ops:
            while len(stack) > 0 and stack[-1] != '(' and ops[c] <= ops[stack[-1]]:
                output.append(stack.pop())
            stack.append(c)
        if c == '(':
            stack.append(c)
        if c == ')':
            while True:
                tmp = stack.pop()
                if tmp == '(':
                    break
                output.append(tmp)
        if c.isdigit():
            output.append(int(c))
    while len(stack) > 0:
        output.append(stack.pop())
    return output
</code></pre>
--
# 2020-18 trick
Python has a kind of operator overloading.

By defining our own class with overridden operators, making sure that numbers 
are made into this class and moving operators around we can just 
put the expressions into eval and let python figure everyting out.

--
# 2020-18 trick solution
<div style="display: flex"><div style="flex: 1">
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
def calcA(str):
    class n:
        def __init__(self,v):
            self.val = v
        def __add__(self, other):
            return n(self.val + other.val)
        def __sub__(self, other):
            return n(self.val \* other.val)
    str = str.replace('\*', '-')
    str = re.sub(r'([0-9]+)', r'n(\1)', str)
    return eval(str).val
</code></pre>
</div><div style="flex:1">
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
def calcB(str):
    class n:
        def __init__(self,v):
            self.val = v
        def __add__(self, other):
            return n(self.val \* other.val)
        def __mul__(self, other):
            return n(self.val + other.val)
    str = str.replace('\*', '-')
    str = str.replace('+', '\*')
    str = str.replace('-', '+')
    str = re.sub(r'([0-9]+)', r'n(\1)', str)
    return eval(str).val
</code></pre>
</div></div>
---
# Plants - and lots of them
<a href="https://adventofcode.com/2018/day/12" target="_blank">2018-12</a>
--
# 2018-12 p1
First we need to parse the somewhat tricky input into an initial state and replacement rules.

Then we need to apply the rules 20 times and calculate the score.

We do this by keeping track of the index of the first and last plant, and all pots between them.
In the beginning of each round we add four empty pots in the beginning and end and apply
all replacements as a sliding window and calculate the score.
--
# 2018-12 p1 solution
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
    offset = 0
    score = 0
    for iter in range(20):
        b = plants.find('#')
        e = plants.rfind('#')

        plants = '....' + plants[b:e+1] + '....'
        offset += b - 4 + 2
        next = ''
        for i in range(len(plants)-4):
            next += map[plants[i:i+5]]

        plants = next

        iter += 1

        score = 0
        for i in range(len(plants)):
            score += i+offset if plants[i] == '#' else 0
    ans1 = score
</code></pre>
--
# 2018-12 p2
Now we have to do this how many times? 50 billion?

We can quickly realize that this will take way too long time (and too much memory)

But if we print the difference between scores we will se a pattern occurring after 
a while... The difference becomes constant!

So we can run the algorithm until we get a stable difference, and then 
calculate what the end result would be

???
The tip here is that if encountering that you have to do something a ridiculous 
amount of times - there is a trick hiding.
--
# 2018-12 p2 solution
<div style="display: flex"><div style="flex: 1">
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
    (offset,iter,pscore,scorediff,diffrepeat) = (0,0,0,0,0)
    while True:
        b = plants.find('#')
        e = plants.rfind('#')
&nbsp;
        plants = '....' + plants[b:e+1] + '....'
        offset += b - 4 + 2
        next = ''
        for i in range(len(plants)-4):
            next += map[plants[i:i+5]]
&nbsp;
        plants = next
&nbsp;
        iter += 1
&nbsp;
        score = 0
        for i in range(len(plants)):
            score += i+offset if plants[i] == '#' else 0
</code></pre></div>
<div style="flex: 1">
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
        if iter == 20:
            ans1 = score
&nbsp;
        if scorediff == score - pscore:
            diffrepeat += 1
        else:
            scorediff = score - pscore
            diffrepeat = 1
&nbsp;
        if diffrepeat > 5:
            ans2 = score + scorediff \* (50000000000 - iter)
            break
&nbsp;
        pscore = score
</code></pre>
</div></div>

---
# Waiting for a bus
<a href="https://adventofcode.com/2020/day/13" target="_blank">2020-13</a>
--
# 2020-13 p1
This seems quite straight forward.

A bus number is a modulus, so taking the wanted time mod the bus number
we know how long time since the last bus left. By subtracting this from 
the bus number we know how long to the next bus.

Now we only need to find the bus with the least number of wait minutes 
and calculate the answer.
--
# 2020-13 p1 solution
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
    data = input.split(',')
    min = 10e100
    for d in data:
        if d != 'x':
            n = int(d)
            tmp = n - start % n
            if tmp < min:
                min = tmp
                ans1 = min * n

    print(ans1)

</code></pre>

--
# 2020-13 p2
Now this becomes tricky.

We cannot really find a pattern, but...

Let's look at the example - 
This looks like an equation system:
```
x = 7-0  mod 7  = 0  mod 7  
x = 13-1 mod 13 = 12 mod 13
x = 59-4 mod 59 = 55 mod 59
x = 31-6 mod 31 = 25 mod 31
x = 19-7 mod 19 = 12 mod 19
```

Chinese Reminder Theorem...

So we need to write a solver!

???
Google it! 

I might have known this 20 years ago when writing
an exam in discrete maths, but not now!

--
# 2020-13 p2 solution
<div style="display: flex"><div style="flex: 1">
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
    def inv(a, m):
        m0 = m
        x0 = 0
        x1 = 1
        if m == 1:
            return 0
        while a > 1:
            q = a // m
            t = m
            m = a % m
            a = t
            t = x0
            x0 = x1 - q \* x0
            x1 = t
        if x1 < 0:
            x1 += m0
        return x1
</code></pre></div>
<div style="flex: 1">
<pre class="python" style="font-size: x-small"><code data-trim data-noescape>
    co = []
    for i in range(len(data)):
        if data[i] != 'x':
            n = int(data[i])
            co.append(((n-i)%n,n))
    &nbsp;
    M = 1
    for a,m in co:
        M \*= m
    &nbsp;
    ans2 = 0
    for a,m in co:
        b = M // m
        ans2 += a \* b \* inv(b, m)
    &nbsp;
    ans2 = ans2 % M
    &nbsp;
    print(ans2)
</code></pre>
</div></div>

---
# Final words

It's less than 9 months til December 1!

There are 200 old problems on the Advent of Code page to solve.

Others:

https://projecteuler.net/

https://open.kattis.com/
---
<!-- .slide: class="center" data-state="hide-footer" -->
## Arno Lepisk
### <code style="background: inherit;">arno.lepisk@hiq.se</code>
### <span class="twitter"><code style="background: inherit;">@arno_l</code></span> &nbsp; <span class="github"><code style="background: inherit;">f00ale</code></span>

???
Thank you!

