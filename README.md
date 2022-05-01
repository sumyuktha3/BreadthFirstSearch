# Breadth First Search
## AIM

To develop an algorithm to find the route from the source to the destination point using breadth-first search.

## THEORY
Explain the problem statement

## DESIGN STEPS

### STEP 1:
Identify a location in the google map:

### STEP 2:
Select a specific number of nodes with distance

### STEP -> Write your own steps:


## ROUTE MAP
#### Include your own map
#### Example map
![IMG_20220501_000913](https://user-images.githubusercontent.com/75235818/166118500-fb6e31b4-236f-40b4-aa1c-6a6993a47c49.jpg)

## PROGRAM
```
%matplotlib inline
import matplotlib.pyplot as plt
import random
import math
import sys
from collections import defaultdict, deque, Counter
from itertools import combinations

class Problem(object):
    """The abstract class for a formal problem. A new domain subclasses this,
    overriding `actions` and `results`, and perhaps other methods.
    The default heuristic is 0 and the default action cost is 1 for all states.
    When yiou create an instance of a subclass, specify `initial`, and `goal` states 
    (or give an `is_goal` method) and perhaps other keyword args for the subclass."""

    def __init__(self, initial=None, goal=None, **kwds): 
        self.__dict__.update(initial=initial, goal=goal, **kwds) 
        
    def actions(self, state):        
        raise NotImplementedError
    def result(self, state, action): 
        raise NotImplementedError
    def is_goal(self, state):        
        return state == self.goal
    def action_cost(self, s, a, s1): 
        return 1
    
    def __str__(self):
        return '{0}({1}, {2})'.format(
            type(self).__name__, self.initial, self.goal)
            
class Node:
    "A Node in a search tree."
    def __init__(self, state, parent=None, action=None, path_cost=0):
        self.__dict__.update(state=state, parent=parent, action=action, path_cost=path_cost)

    def __str__(self): 
        return '<{0}>'.format(self.state)
    def __len__(self): 
        return 0 if self.parent is None else (1 + len(self.parent))
    def __lt__(self, other): 
        return self.path_cost < other.path_cost
        
 failure = Node('failure', path_cost=math.inf) # Indicates an algorithm couldn't find a solution.
cutoff  = Node('cutoff',  path_cost=math.inf) # Indicates iterative deepening search was cut off.

def expand(problem, node):
    "Expand a node, generating the children nodes."
    s = node.state
    for action in problem.actions(s):
        s1 = problem.result(s, action)
        cost = node.path_cost + problem.action_cost(s, action, s1)
        yield Node(s1, node, action, cost)
        

def path_actions(node):
    "The sequence of actions to get to this node."
    if node.parent is None:
        return []  
    return path_actions(node.parent) + [node.action]


def path_states(node):
    "The sequence of states to get to this node."
    if node in (cutoff, failure, None): 
        return []
    return path_states(node.parent) + [node.state]
    
FIFOQueue = deque

def breadth_first_search(problem):
    "Search shallowest nodes in the search tree first."
    node = Node(problem.initial)
    if problem.is_goal(problem.initial):
        return node
    # Remove the following comments to initialize the data structure
    frontier = FIFOQueue([node])
    reached = {problem.initial}
    while frontier:
        node = frontier.pop()
        for child in expand(problem, node):
            s = child.state
            if problem.is_goal(s):
                return child
            if s not in reached:
                reached.add(s)
                frontier.appendleft(child)
    return failure
   
class RouteProblem(Problem):
    """A problem to find a route between locations on a `Map`.
    Create a problem with RouteProblem(start, goal, map=Map(...)}).
    States are the vertexes in the Map graph; actions are destination states."""
    
    def actions(self, state): 
        """The places neighboring `state`."""
        return self.map.neighbors[state]
    
    def result(self, state, action):
        """Go to the `action` place, if the map says that is possible."""
        return action if action in self.map.neighbors[state] else state
    
    def action_cost(self, s, action, s1):
        """The distance (cost) to go from s to s1."""
        return self.map.distances[s, s1]
    
    def h(self, node):
        "Straight-line distance between state and the goal."
        locs = self.map.locations
        return straight_line_distance(locs[node.state], locs[self.goal])
        
class Map:
    """A map of places in a 2D world: a graph with vertexes and links between them. 
    In `Map(links, locations)`, `links` can be either [(v1, v2)...] pairs, 
    or a {(v1, v2): distance...} dict. Optional `locations` can be {v1: (x, y)} 
    If `directed=False` then for every (v1, v2) link, we add a (v2, v1) link."""

    def __init__(self, links, locations=None, directed=False):
        if not hasattr(links, 'items'): # Distances are 1 by default
            links = {link: 1 for link in links}
        if not directed:
            for (v1, v2) in list(links):
                links[v2, v1] = links[v1, v2]
        self.distances = links
        self.neighbors = multimap(links)
        self.locations = locations or defaultdict(lambda: (0, 0))

        
def multimap(pairs) -> dict:
    "Given (key, val) pairs, make a dict of {key: [val,...]}."
    result = defaultdict(list)
    for key, val in pairs:
        result[key].append(val)
    return result
    
House_nearby_locations = Map(
    {('Ayapakkam', 'Athipet'):  6,('Athipeti', 'Ambattur'):  2,('Ambattur' , 'Korattur'): 3.5,('Korattur' , 'Padi'): 2,('Padi' , 'Villivakkam'): 3,
     ('Villivakkam' , 'ICF'): 2,('ICF' , 'MMM'): 6,('MMM' , 'VR Mall'): 4,('VR Mall' , 'Rohini theater'): 3,('Rohini theater' , 'Mogappair East'): 3,
     ('Mogappair East' , 'Mogappair West'): 3,('Mogappair West' , 'Decathlon'): 1,('Decathlon' , 'Ayanambakkam'): 2,('Ayanambakkam' , 'Appolo Hospital'): 1,('Mogappair West' , 'Nolambur'): 1,('Nolambur' , 'MGR University'): 2,('MGR University' , 'AGS Cinemas'): 2,('AGS Cinemas' , 'SPP Gardens'): 3,('MGR University' , 'Koyambedu'): 5,('Koyambedu' , 'Rohini theater'): 1,('MMM' , 'Mogappair East'): 1,('Padi' , 'ICF'): 4,('Mogappair West' , 'Ambattur'): 7,('Ambattur' , 'Decathlon'): 4})


r0 = RouteProblem('Ayapakkam', 'Villivakkam', map=House_nearby_locations)
r1 = RouteProblem('MMM', 'Appollo Hospital', map=House_nearby_locations)
r2 = RouteProblem('Koyambedu', 'ICF', map=House_nearby_locations)
r3 = RouteProblem('VR Mall', 'Decathlon', map=House_nearby_locations)
r4 = RouteProblem('Ambattur', 'SPP Gardens', map=House_nearby_locations)

goal_state_path=breadth_first_search(r2)
print("GoalStateWithPath:{0}".format(goal_state_path))
path_states(goal_state_path) 
print("Total Distance={0} Kilometers".format(goal_state_path.path_cost))
```

## OUTPUT:

![ai3](https://user-images.githubusercontent.com/75235818/166142438-3c87e553-eba0-4b31-b840-7007f159c670.jpg)

## SOLUTION JUSTIFICATION:
Route follow the minimum distance between locations using breadth-first search.
## RESULT:
Thus the program developed for finding route with drawn map and finding its distance covered.
