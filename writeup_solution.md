## Project: 3D Motion Planning
![Quad Image](./misc/enroute.png)

---

[//]: # (Image References)

[image1]: ./images/path.png "Visualization"

# Required Steps for a Passing Submission:
1. Load the 2.5D map in the colliders.csv file describing the environment.
2. Discretize the environment into a grid or graph representation.
3. Define the start and goal locations.
4. Perform a search using A* or other search algorithm.
5. Use a collinearity test or ray tracing method (like Bresenham) to remove unnecessary waypoints.
6. Return waypoints in local ECEF coordinates (format for `self.all_waypoints` is [N, E, altitude, heading], where the drone’s start location corresponds to [0, 0, 0, 0].
7. Write it up.
8. Congratulations!  Your Done!

## [Rubric](https://review.udacity.com/#!/rubrics/1534/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it! Below I describe how I addressed each rubric point and where in my code each point is handled.

### Explain the Starter Code

#### 1. Explain the functionality of what's provided in `motion_planning.py` and `planning_utils.py`
These scripts contain a basic planning implementation that includes a state machine similar to the one used in the Backyard Flyer project. Once the Arming State is achieved, we enter a new state that did not exist in the Backyard Flyer called the planning state where a flight plan is created using the A-star algorithm.  In the Backyard Flyer we provided a static list of waypoints for the drone to follow, in this implementation a goal state is selected 10 meters north by 10 meters east of the drone's start state and the planning stage creates a waypoint list using the A-star algorithm to find the best path for the drone to follow to achieve it's goal state.  Once the planning is completed the state machine will transition to takeoff and function much like the Backyard Flyer code following it's path to achieve the goal state. motion_planning.py handles the state machine and transitions between states while the planning_utils.py provides the functions for the A-star algorithm, actions and valid actions check.

The A-star search algorithm uses a heuristic that is admissible and consistent, the heuristic used is the euclidean distance algorithm, implemented using numpy L2Norm of the vectors difference, for finding the path with the lowest cost from start to goal.  These waypoints are used to create the flight plan for the drone.  The A-star algorithm utilizes the actions and valid actions functions to determine the valid actions available for extending the partial plan and the costs associated with that action.

### Implementing Your Path Planning Algorithm

#### 1. Set your global home position
Here students should read the first line of the csv file, extract lat0 and lon0 as floating point values and use the self.set_home_position() method to set global home. Explain briefly how you accomplished this in your code.

The Drone API provides a set_home_position method to set the home position as noted above, whcih I use to set the global home position using a function get_global_home that takes in the csv file as an argument.  The function extracts lat0 and lon0 as floating points and returns a numpy array of the gloabl home position:

```
def get_global_home(filename):
    with open(filename) as f:
        lat, lon = [float(x.replace("lat0 ", "").replace("lon0 ", "")) for x in f.readline().split(',')]
        global_home = np.array([lon, lat, 0.0])
    return global_home
```

#### 2. Set your current local position
The Drone API provided set_home_position method is used to set the drones current local position.

```
self.set_home_position(global_home[0], global_home[1], global_home[2])
```

#### 3. Set grid start position from local position
The udacidrone.frame_utils contains a global_to_local method which I used to calculate the north-east-down coordinates from the global position using global_home position as a reference point.

```
current_local_pos = global_to_local(self.global_position, global_home)
```

#### 4. Set grid goal position from geodetic coords
Setting the GLOBAL_GOAL vector with longitude and latitude location will be convert to local location by applying the global_to_local function and then subtracting the north and east offsets.  The grid_goal will be set from the converted geodetic coords.

```
GLOBAL_GOAL = (-122.397336, 37.793836) 

goal_pos = global_to_local([GLOBAL_GOAL[0], GLOBAL_GOAL[1], self._altitude], self.global_home)
grid_goal = (int(goal_pos[0]-north_offset), int(goal_pos[1]-east_offset))
        
```

I added visualization plots of the grid with the start and end locations, as well as plotting the path once found.  The plotting was included in a function called plot_map which can be found in the motion_planning_solution notebook.

#### 5. Modify A* to include diagonal motion (or replace A* altogether)
I modified Actions in planning_utils.py to include diaganal movement, with their associated cost.  

```

    NORTH_WEST = (-1, -1, np.sqrt(2))
    NORTH_EAST = (-1, 1, np.sqrt(2))
    SOUTH_WEST = (1, -1, np.sqrt(2))
    SOUTH_EAST = (1, 1, np.sqrt(2))

```

To allow this to work I also needed to modify the valid_actions to allow diaganol movement.

```

    if (x - 1 < 0 or y - 1 < 0) or grid[x - 1, y - 1] == 1:
        valid_actions.remove(Action.NORTH_WEST)
    if (x - 1 < 0 or y + 1 > m) or grid[x - 1, y + 1] == 1:
        valid_actions.remove(Action.NORTH_EAST)
    if (x + 1 > n or y - 1 < 0) or grid[x + 1, y - 1] == 1:
        valid_actions.remove(Action.SOUTH_WEST)
    if (x + 1 > n or y + 1 > m) or grid[x + 1, y + 1] == 1:
        valid_actions.remove(Action.SOUTH_EAST)

```

With these two modifications the drone is able to move in diagonal directions when the A* algorithm searches for a path.

#### 6. Cull waypoints 
I used the collinearity method for culling the waypoints.  Three functions where added to planning_utils.py, importing prune_path function to perform the culling of the waypoints and returning a more optimized path for the drone to travel.


### Execute the flight
#### 1. Does it work?
It works!
Everything seems to work pretty good.

![alt text][image1]

### Double check that you've met specifications for each of the [rubric](https://review.udacity.com/#!/rubrics/1534/view) points.
  
# Extra Challenges: Real World Planning

For an extra challenge, consider implementing some of the techniques described in the "Real World Planning" lesson. You could try implementing a vehicle model to take dynamic constraints into account, or implement a replanning method to invoke if you get off course or encounter unexpected obstacles.


