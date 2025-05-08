# Program a robot using Blockly to build the program

Demo: https://antonrd.github.io/program-the-robot/

For some time I had this idea to create a simple HTML page for young people learning to code. The page would allow them ot use Blockly blocks to construct a program, which is used to control a robot moving on a fields of square cells. It has always been a very low priority project and I never found the time for it. But nowadays when AI models can generate the code for such projects, I decided to give it a go. With a few iterations, the code that does something meaningful was ready. This required no coding on my side, just prompting AI models, looking for solutions to errors and reading the model explanations.

Used Google AI Studio with Gemini 2.5 Pro Preview 05-06 to generate the source code for a spec that is described in the spec.txt file. It also contains follow-up requirements and a few comments about what happened while iterating on the code with Gemini.

## Issues while generating the code

### Old Blockly library API expectations

The model generated code initially, which was causing a JS error. The problem seems to be that earlier versions of Blockly worked with code like that:

```js
  Blockly.JavaScript['robot_move_forward'] = function(block) {
    return 'await robotAPI.moveForward();\n';
  };
```

but with the latest version this leads to an error: `Uncaught (in promise) Error: JavaScript generator does not know how to generate code for block type "robot_move_forward".`

Initially Gemini could not suggest a fix, so I checked with ChatGPT, which initially also suggested a number of other possible issues, until it finally pointed out the real problem - that the code needs to be written as follows:

```js
  javascriptGenerator = Blockly.JavaScript;

  // Other code may be found here...

  javascriptGenerator.forBlock['robot_move_forward'] = function(block) {
    return 'await robotAPI.moveForward();\n';
  };
```

## Follow up question to Gemini

Finally, asked Gemini the following question: "given this game, is it possible to write a program for the robot, which guarantees that the final goal cell will always be reached regardless of the board?"

Its answer goes like that:

No, it is **not possible** to write a single, fixed Blockly program that guarantees the robot will *always* reach the goal cell regardless of the randomly generated board, given the current set of commands and sensors.

Here's why:

1.  **Lack of Global Pathfinding/Memory:**
    *   The robot's commands (`move_forward`, `turn_left`, `turn_right`) are purely local.
    *   The sensors (`is_wall_ahead`, `is_on_goal`) are also local, only providing information about the immediate environment or current state.
    *   The robot has no way to "see" the entire map, store a map, or remember complex paths it has tried.

2.  **The Nature of Mazes:**
    *   Randomly generated mazes can have many dead ends, loops, and complex structures.
    *   Without a more sophisticated pathfinding algorithm (like A\*, Dijkstra's, or even a simple wall-follower with memory to avoid infinite loops in certain maze types), the robot can easily get stuck.

3.  **Limitations of Simple Loop Structures:**
    *   **`for` loop:** Executes a fixed number of times, which is not adaptable to varying maze sizes or path lengths.
    *   **`while (is_wall_ahead)` / `while not (is_wall_ahead)`:** Useful for navigating along corridors or around simple obstacles, but can lead to infinite loops if the robot enters a concave area or a specific type of loop without additional logic.
    *   **`while not (is_on_goal)`:** This is the most promising for a general solution, but the actions *inside* this loop need to be intelligent enough to explore effectively.

**What kind of algorithms *could* solve any maze (but are not implementable with current blocks)?**

*   **Wall Follower (Right-Hand or Left-Hand Rule):** This simple algorithm can solve *simply connected* mazes (mazes where all walls are connected, or equivalently, where the goal is reachable by always keeping one hand on a wall). However, it can fail in mazes with detached wall sections or islands if the start/goal are on different "islands" relative to the wall being followed. It also often takes a very long path.
    *   *Blockly limitation:* Implementing a robust wall-follower that can also detect it's back at the start or in an infinite loop is tricky without more state or advanced logic blocks.

*   **Random Mouse Algorithm:** The robot moves randomly until it finds the goal. Inefficient but will eventually find it if a path exists and the maze is finite.
    *   *Blockly limitation:* No "move random direction" block, and even if there were, it's highly inefficient.

*   **Pledge Algorithm:** An improvement on the wall follower that can escape some traps.
    *   *Blockly limitation:* Requires keeping track of net turns, which is beyond the current sensors.

*   **Trémaux's Algorithm / Depth-First Search (DFS) with Marking:** This is a guaranteed method. It involves marking paths as visited (e.g., drawing a line on the floor). When at a junction, it tries an unmarked path. If all paths are marked, it backtracks.
    *   *Blockly limitation:* No way to "mark" cells as visited or to systematically backtrack based on such markings.

*   **Breadth-First Search (BFS) / A\*:** These algorithms explore the maze layer by layer or by a heuristic, keeping track of visited nodes and paths. They are guaranteed to find the shortest path if one exists.
    *   *Blockly limitation:* Requires data structures (queues, priority queues) and memory of the grid, which are not available.

**What kind of programs *can* be built with the current blocks?**

*   Programs that solve specific, known board layouts.
*   Programs that use simple strategies like "try to go forward, if wall turn right" which might work on very open mazes but will fail on complex ones.
*   Programs that attempt a limited form of wall-following (e.g., "while not on goal: if not wall ahead move, else turn right"). This will get stuck in many common maze patterns.

**Conclusion:**

To guarantee reaching the goal on *any* solvable board, the robot (or its programming environment) would need:
*   The ability to **remember** visited cells or paths.
*   More sophisticated **decision-making logic** than simple conditional checks based on immediate surroundings.
*   Potentially, a way to **backtrack** systematically.

The current set of Blockly blocks provides a good introduction to basic robot control and sequential logic, but it's not powerful enough for universal maze-solving algorithms. It's more about the user devising a specific plan for a given (or simple type of) maze.

### Expectations that Blockly would detect async JS code inside user-defined functions

Gemini explicitly expressed its reliance on the ability of Blockly to make the JS code for user-defined functions blocks be using async functions. But this does not seem to be the case with the version of Blockly used. Another prompt was needed to ask Gemini to generate code, which can handle that.
