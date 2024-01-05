---
layout: assignment
title: "Individual Project 1"
permalink: /assignments/ip1
parent: Assignments
nav_order: 1
due_date: "Wednesday January 24, 11:00am EDT"
submission_notes: Submit via Autograder.io at neu.autograder.io
---

Welcome aboard to the Covey.Town team! We're glad that you're here and ready to join our development team as a new software engineer.
We're building an open source virtual meeting application, and are very happy to see that we have so many new developers who can help make this application a reality.
By the end of the semester, you'll be able to propose, design, implement and test a new feature for our project.
We understand that some of you may have some web development experience, but don't expect that most of you do, and hence, have created an individual project to help you get up to speed with our existing codebase and development environment.

Covey.Town is a web application that consists of some code that runs in each client's web browser, and also code that runs on a server.
Users join the application in a "town": a 2D arcade-style map with different rooms to explore.
Each town is also a video call: when two players get close to each other, they can see and hear each other; there is also a text chat available within the town.
In Winter of 2021, our lead software engineer, Avery, developed a prototype for Covey.Town, and since then, hundreds of students in this class have built on that codebase.
The most recent class-wide effort, in Fall 2023, added a concept called [Game Areas](https://neu-se.github.io/CS4530-Fall-2023/assignments/ip1), allowing players to play games within a special part of the town. 
Students implemented a single game, [Tic-Tac-Toe](https://en.wikipedia.org/wiki/Tic-tac-toe), as a proof-of-concept for this feature.

The objective for this semester's individual project is to extend this new `GameArea` abstraction, implementing [Connect Four](https://en.wikipedia.org/wiki/Connect_Four) as a new game that can be played within a `GameArea`.
This implementation effort will be split across two deliverables. In this first deliverable, you will implement and test the core backend components for this feature, and in the second deliverable, you will implement and test the frontend components. 

## Objectives of this assignment
The objectives of this assignment are to:
*  Get you familiar with the basics of TypeScript, VSCode, and the project codebase
*  Learn how to read and write code in TypeScript
*  Translate high-level requirements into code
*  Learn how to write unit tests with Jest

## Getting started with this assignment

Before you begin, be sure to check that you have NodeJS 18.x installed, along with VSCode. We have provided a [tutorial on setting up a development environment for this class]({{site.baseurl}}% link tutorials/week1-getting-started.md % **FIX LINK**) 
Start by [downloading the starter code]({{site.baseurl}}{% link /Assignments/ip1/ip1-handout.zip %}). Extract the archive and run `npm install` to fetch the dependencies. Avery has provided you with some very basic sanity tests that you can extend for testing your implementation as you go. You can run these tests with the command `npm test ConnectFour` (note that many tests are *expected* to fail until you have begun to implement the assignment).

## Grading
This submission will be scored out of 100 points, 90 of which will be automatically awarded by the grading script, with the remaining 10 manually awarded by the course staff.

Your code will automatically be evaluated for linter errors and warnings. Submissions that have *any* linter errors will automatically receive a grade of 0. **Do not wait to run the linter until the last minute**. To check for linter errors, run the command `npm run lint` from the terminal. The handout contains the same eslint configuration that is used by our grading script.

Your code will be automatically evaluated for functional correctness by a test suite that expands on the core tests that are distributed in the handout. 
Your tests will be automatically evaluated for functional correctness by a process that will inject bugs into our reference solution: to receive full marks your tests must detect a minimum number of injected bugs. 
Each submission will be graded against the same set of injected bugs (repeated submissions will not receive new/different injected bugs).
You will __not__ receive detailed feedback on which injected bugs you do or do not find.

The autograding script will impose a strict rate limit of 5 submissions per 24 hours.
Submissions that fail to grade will not count against the quota.
This limit exists to encourage you to start early on this assignment: students generally report that assignments like this take between 3-20 hours.
If you start early, you will be able to take full advantage of the resources that we provide to help you succeed: office hours, discussion on Piazza --- and the ability to have a greater total number of submission attempts.

Your code will be manually evaluated for conformance to our course [style guide]({{ site.baseurl }}{% link style.md %}). This manual evaluation will account for 10% of your total grade on this assignment. We will manually evaluate your code for style on the following rubric:

To receive all 10 points:
* All new names (e.g. for local variables, methods, and properties) follow the naming conventions defined in our style guide
* There are no unused local variables
* All public properties and methods (other than getters, setters, and constructors) are documented with JSDoc-style comments that describes what the property/method does, as defined in our style guide
* The code and tests that you write generally follows the design principles discussed in week one. In particular, your design does not have duplicated code that could have been refactored into a shared method.

We will review your code and note each violation of this rubric. We will deduct two points for each violation, up to a maximum of deducting all 10 style points.

## Overview of Relevant Classes

<script src="{{site.baseurl}}/assets/js/mermaid.min.js"></script>
<div class="mermaid">
%%{init: { 'theme':'forest', } }%%
classDiagram
    class Game {
      +GameState state
      +GameInstanceID id
      ~Player[] _players
      ~GameResult _result
    + join(player: Player)
    + leave(player: Player)
    ~ _join(player: Player)
    ~ _leave(player: Player)
    + applyMove(move: GameMove)

    }
    class  GameArea {
        ~Game _game
        ~GameResult _history
    }
    class ConnectFourGame {

    }
    class ConnectFourGameArea {
    }
    class GameResult {
        +GameInstanceID gameID
        +Map scores
    }
    GameArea o-- GameResult
    ConnectFourGame ..|> Game
    ConnectFourGameArea ..|> GameArea
    ConnectFourGameArea o-- ConnectFourGame
    GameArea o-- Game
</div>

State diagram for game status:
<div class="mermaid">
%%{init: { 'theme':'forest', } }%%
flowchart TD

    A[WAITING_FOR_PLAYERS] --> |1 player joins| A
    A --> |2nd player joins| B
    B --> |A player leaves| A
    B[WAITING_TO_START] --> |1 player clicks ready|B
    B --> |Other player clicks ready| C
    C[IN_PROGRESS] --> |Game Ends or a player leaves| D
    D[OVER] --> |1 player clicks 'new game'| B
</div>

## Implementation Tasks
This deliverable has four parts; each part will be graded on its own rubric. You should complete the assignment one part at a time, in the order presented here.

**General Requirements**: Implement your code *only* in the files specified: `src/town/games/ConnectFourGame.ts`, `src/town/games/ConnectFourGame.test.ts` and `src/town/games/ConnectFourGame.ts`. You should not install any additional dependencies. The autograder will ignore any other files that you modify, and will not install any dependencies that you add to the project.

### Task 1: Joining and Leaving the ConnectFourGame (10 points)
The `ConnectFourGame` class extends the base `Game` class, implementing the semantics of the game Connect Four. Avery has provided a definition for the types that will be used to represent the state of a `ConnectFourGame` - `ConnectFourGameState`. That type definition is reproduced below:

{% highlight typescript %}
/**
 * Type for the state of a ConnectFour game.
 * The state of the game is represented as a list of moves, and the playerIDs of the players (red and yellow)
 */
export interface ConnectFourGameState extends WinnableGameState {
  // The moves in this game
  moves: ReadonlyArray<ConnectFourMove>;
  // The playerID of the red player, if any
  red?: PlayerID;
  // The playerID of the yellow player, if any
  yellow?: PlayerID;
  // Whether the red player is ready to start the game
  redReady?: boolean;
  // Whether the yellow player is ready to start the game
  yellowReady?: boolean;
  // The color of the player who will make the first move
  firstPlayer: ConnectFourColor;
}

/**
 * Type for a move in ConnectFour
 * Columns are numbered 0-6, with 0 being the leftmost column
 * Rows are numbered 0-5, with 0 being the top row
 */
export interface ConnectFourMove {
  gamePiece: ConnectFourColor;
  col: ConnectFourColIndex;
  row: ConnectFourRowIndex;
}

/**
 * Row indices in ConnectFour start at the top of the board and go down
 */
export type ConnectFourRowIndex = 0 | 1 | 2 | 3 | 4 | 5;
/**
 * Column indices in ConnectFour start at the left of the board and go right
 */
export type ConnectFourColIndex = 0 | 1 | 2 | 3 | 4 | 5 | 6;

export type ConnectFourColor = 'Red' | 'Yellow';
{% endhighlight %}

Your first task is to implement the `_join`, `startGame` and `_leave` methods of `ConnectFourGame`. To implement these methods, you should not need to read any other parts of the codebase besides `Game.ts` and `ConnectFourGame.ts`. You might find it useful or necessary to modify the constructor of `ConnectFourGame` to initialize the state of the game, and to add private instance variables and/or helper methods.

{::options parse_block_html="true" /}
<details><summary markdown="span">View the specification for these methods</summary>
{% highlight typescript %}
  /**
   * Joins a player to the game.
   * - Assigns the player to a color (red or yellow). If the player was in the prior game, then attempts
   * to reuse the same color if it is not in use. Otherwise, assigns the player to the first
   * available color (red, then yellow).
   * - If both players are now assigned, updates the game status to WAITING_TO_START.
   *
   * @throws InvalidParametersError if the player is already in the game (PLAYER_ALREADY_IN_GAME_MESSAGE)
   * @throws InvalidParametersError if the game is full (GAME_FULL_MESSAGE)
   *
   * @param player the player to join the game
   */
  protected _join(player: Player): void

  /**
   * Indicates that a player is ready to start the game.
   *
   * Updates the game state to indicate that the player is ready to start the game.
   *
   * If both players are ready, the game will start. If there was a prior game, and at least
   * one of the players from the prior game is in this game, then the first player will be
   * the other color. Otherwise, the first player will be red.
   *
   * @throws InvalidParametersError if the player is not in the game (PLAYER_NOT_IN_GAME_MESSAGE)
   * @throws InvalidParametersError if the game is not in the WAITING_TO_START state (GAME_NOT_STARTABLE_MESSAGE)
   *
   * @param player The player who is ready to start the game
   */
  public startGame(player: Player): void 

  /**
   * Removes a player from the game.
   * Updates the game's state to reflect the player leaving.
   *
   * If the game state is currently "IN_PROGRESS", updates the game's status to OVER and sets the winner to the other player.
   *
   * If the game state is currently "WAITING_TO_START", updates the game's status to WAITING_FOR_PLAYERS.
   *
   * If the game state is currently "WAITING_FOR_PLAYERS" or "OVER", the game state is unchanged.
   *
   * @param player The player to remove from the game
   * @throws InvalidParametersError if the player is not in the game (PLAYER_NOT_IN_GAME_MESSAGE)
   */
  protected _leave(player: Player): void

{% endhighlight %}
</details>

*Testing*: Avery has provided you with some very simple (and incomplete) tests for `_join`. You can run these tests by running the command `npx jest --watch TicTacToeGame.test`, which will automatically re-run the tests as you update the file (note that tests for `applyMove` will also run - but you can ignore those at this point). As you read and understand the specification, you should extend the existing test suite, adding tests to cover the entire specification. Please implement these additional tests in the file `src/town/games/TicTacToeGame.test.ts`.

Grading for implementation tasks:
* `_join`: 2 points
* `startGame`: 1 points
* `_leave`: 1 points

Grading for testing tasks:
* `_join`: 2 points
* `startGame`: 2 points
* `_leave`: 2 points 

### Task 2: Connect Four Game Semantics (70 points total)
The next (and largest) task for this deliverable is to implement the method `ConnectFourGame.applyMove`, which applies a player's move to the game. This method is responsible for validating the move, and updating the game state to reflect the move. Given the complexity of this method, you should anticipate creating (at least one) private, helper method to implement its logic.

<details><summary markdown="span">View the specification for this method</summary>
{% highlight typescript %}
  /**
   * Applies a move to the game.
   * Uses the player's ID to determine which color they are playing as (ignores move.gamePiece).
   *
   * Validates the move, and if it is valid, applies it to the game state.
   *
   * If the move ends the game, updates the game state to reflect the end of the game,
   * setting the status to "OVER" and the winner to the player who won (or "undefined" if it was a tie)
   *
   * @param move The move to attempt to apply
   *
   * @throws InvalidParametersError if the game is not in progress (GAME_NOT_IN_PROGRESS_MESSAGE)
   * @throws InvalidParametersError if the player is not in the game (PLAYER_NOT_IN_GAME_MESSAGE)
   * @throws INvalidParametersError if the move is not the player's turn (MOVE_NOT_YOUR_TURN_MESSAGE)
   * @throws InvalidParametersError if the move is invalid per the rules of Connect Four (BOARD_POSITION_NOT_VALID_MESSAGE)
   *
   */
  public applyMove(move: GameMove<ConnectFourMove>): void
{% endhighlight %}
</details>

Grading for implementation tasks:
* Correctly validating who is the first player: 5 points
* Applying moves: 7 points
* Checking for invalid moves: 12 points
* Checking for game-ending moves: 10 points

Grading for testing tasks:
* Correctly validating who is the first player: 5 points
* Permitting valid moves: 8 points
* Checking for invalid moves: 13 points
* Handling game-ending moves: 10 points

### Task 3: Implement the ConnectFourGameArea (10 points total)
The `ConnectFourGameArea` receives `InteractableCommand`s from players who enter the area on their client. The main responsibility of this class is to interpet those commands, dispatching them as appropriate to the `ConnectFourGame` instance that it manages. Your final task is to implement the `handleCommand` method of `TicTacToeGameArea`.

There are four types of commands that the `ConnectFourGameArea` will receive, which map directly to the three methods of `ConnectFourGame` that you implemented in the previous task. 

Avery has provided a complete test suite for `handleCommand` - you do not need to write any additional tests.

<details><summary markdown="span">View the specification for this methods</summary>
{% highlight typescript %}
  /**
   * Handle a command from a player in this game area.
   * Supported commands:
   * - JoinGame (joins the game `this._game`, or creates a new one if none is in progress)
   * - StartGame (indicates that the player is ready to start the game)
   * - GameMove (applies a move to the game)
   * - LeaveGame (leaves the game)
   *
   * If the command ended the game, records the outcome in this._history
   * If the command is successful (does not throw an error), calls this._emitAreaChanged (necessary
   * to notify any listeners of a state update, including any change to history)
   * If the command is unsuccessful (throws an error), the error is propagated to the caller
   *
   * @see InteractableCommand
   *
   * @param command command to handle
   * @param player player making the request
   * @returns response to the command, @see InteractableCommandResponse
   * @throws InvalidParametersError if the command is not supported or is invalid.
   * Invalid commands:
   * - GameMove, StartGame and LeaveGame: if the game is not in progress (GAME_NOT_IN_PROGRESS_MESSAGE) or if the game ID does not match the game in progress (GAME_ID_MISSMATCH_MESSAGE)
   * - Any command besides JoinGame, GameMove, StartGame and LeaveGame: INVALID_COMMAND_MESSAGE
   */
  public handleCommand<CommandType extends InteractableCommand>(
    command: CommandType,
    player: Player,
  ): InteractableCommandReturnType<CommandType>
{% endhighlight %}
</details>

Grading for implementation tasks:
* Handling JoinGame: 2 points
* Handling GameMove: 2 points
* Handling LeaveGame: 2 points
* Handling StartGame: 2 points
* Handling invalid commands: 2 point

## Submission Instructions
Submit your assignment to the instance of Autograder.io running at [neu.autograder.io](https://neu.autograder.io).
Navigate to [neu.autograder.io](https://neu.autograder.io) in your web browser, click the "Sign in" button, and log in with your Northeastern account.
You should then see the course listed on the neu.autograder.io home page.
Please contact the instructors immediately if you have difficulty accessing the course on Autograder.io.
If you haven't been added to the course roster yet, you can access the assignment page at [this direct link](https://neu.autograder.io/web/project/12).

To submit your assignment: run the command `npm run zip` in the top-level directory of the handout. This will produce a file called `covey-town-townService.zip`. Submit that zip file on Autograder.io.

Autograder.io will provide you with feedback on your submission, but note that it will *not* include any marks that will be assigned after we manually grade your submission for code style (these marks will remain hidden until it is graded). It may take several minutes for the grading script to complete.

Autograder.io is configured to only provide feedback on at most 5 submissions per-24-hours per-student (submissions that fail to run or receive a grade of 0 are not counted in that limit). We strongly encourage you to lint and test your submission on your local development machine, and *not* rely on Autograder.io for providing grading feedback - relying on Autograder.io is a very slow feedback loop.
To check for linter errors, run the command `npm run lint` from the terminal. The handout contains the same eslint configuration that is used by our grading script.
Submission limit resets at 11am EST.