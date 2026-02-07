# Design Document: Flappy Bird Game

## Overview

This design describes a browser-based Flappy Bird game built with Next.js and TypeScript. The game uses HTML5 Canvas for rendering and implements a classic game loop pattern with physics simulation. The architecture separates concerns into distinct modules: game state management, physics engine, rendering, collision detection, and input handling.

The game runs entirely client-side and uses React hooks for lifecycle management and state synchronization with the UI. The core game loop runs at 60 FPS using `requestAnimationFrame`, while the physics engine applies gravity and velocity updates on each frame.

## Architecture

The system follows a modular architecture with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────┐
│                    Next.js App Router                    │
│                   (app/page.tsx)                         │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              GameComponent (Client Component)            │
│  - Canvas initialization                                 │
│  - Event listener setup                                  │
│  - Game loop orchestration                               │
└────┬────────────┬────────────┬────────────┬─────────────┘
     │            │            │            │
     ▼            ▼            ▼            ▼
┌─────────┐ ┌─────────┐ ┌──────────┐ ┌──────────────┐
│  Game   │ │ Physics │ │Collision │ │   Renderer   │
│  State  │ │ Engine  │ │ Detector │ │              │
└─────────┘ └─────────┘ └──────────┘ └──────────────┘
     │            │            │            │
     └────────────┴────────────┴────────────┘
                     │
                     ▼
              ┌──────────────┐
              │ Input Handler│
              └──────────────┘
```

**Component Responsibilities:**

- **GameComponent**: React component that manages the canvas, game loop, and coordinates all subsystems
- **GameState**: Manages game state (ready, playing, game_over), bird position/velocity, pipes, and score
- **PhysicsEngine**: Applies gravity, velocity, and movement calculations
- **CollisionDetector**: Checks for intersections between bird and pipes/boundaries
- **Renderer**: Draws all game elements to the canvas
- **InputHandler**: Processes keyboard and mouse events

## Components and Interfaces

### GameState

The central state management module that holds all game data.

```typescript
enum GameStatus {
  READY = 'ready',
  PLAYING = 'playing',
  GAME_OVER = 'game_over'
}

interface Bird {
  x: number;           // Horizontal position (constant during gameplay)
  y: number;           // Vertical position
  velocity: number;    // Vertical velocity
  width: number;       // Bird hitbox width
  height: number;      // Bird hitbox height
}

interface Pipe {
  x: number;           // Horizontal position
  gapY: number;        // Y position of gap center
  gapHeight: number;   // Height of the gap
  width: number;       // Pipe width
  passed: boolean;     // Whether bird has passed this pipe (for scoring)
}

interface GameState {
  status: GameStatus;
  bird: Bird;
  pipes: Pipe[];
  score: number;
  canvasWidth: number;
  canvasHeight: number;
}

function createInitialState(canvasWidth: number, canvasHeight: number): GameState;
function resetGame(state: GameState): GameState;
```

### PhysicsEngine

Handles all physics calculations including gravity, velocity, and movement.

```typescript
interface PhysicsConfig {
  gravity: number;        // Downward acceleration (pixels/frame²)
  jumpVelocity: number;   // Upward velocity on jump (pixels/frame)
  birdSpeed: number;      // Horizontal speed (not used - bird stays fixed)
  pipeSpeed: number;      // Horizontal pipe movement speed (pixels/frame)
  terminalVelocity: number; // Maximum downward velocity
}

function applyGravity(bird: Bird, config: PhysicsConfig): Bird;
function applyJump(bird: Bird, config: PhysicsConfig): Bird;
function updateBirdPosition(bird: Bird): Bird;
function updatePipePositions(pipes: Pipe[], config: PhysicsConfig): Pipe[];
```

### CollisionDetector

Detects collisions between the bird and pipes or canvas boundaries.

```typescript
interface Rectangle {
  x: number;
  y: number;
  width: number;
  height: number;
}

function checkBirdPipeCollision(bird: Bird, pipe: Pipe): boolean;
function checkBirdBoundaryCollision(bird: Bird, canvasHeight: number): boolean;
function rectanglesIntersect(rect1: Rectangle, rect2: Rectangle): boolean;
```

### PipeGenerator

Manages pipe creation and removal.

```typescript
interface PipeGeneratorConfig {
  pipeWidth: number;
  gapHeight: number;
  minGapY: number;      // Minimum Y position for gap center
  maxGapY: number;      // Maximum Y position for gap center
  spawnInterval: number; // Frames between pipe spawns
}

function shouldSpawnPipe(frameCount: number, config: PipeGeneratorConfig): boolean;
function generatePipe(canvasWidth: number, canvasHeight: number, config: PipeGeneratorConfig): Pipe;
function removeOffscreenPipes(pipes: Pipe[]): Pipe[];
```

### Renderer

Draws all game elements to the canvas.

```typescript
interface RenderConfig {
  birdColor: string;
  pipeColor: string;
  backgroundColor: string;
  textColor: string;
}

function clearCanvas(ctx: CanvasRenderingContext2D, width: number, height: number): void;
function drawBird(ctx: CanvasRenderingContext2D, bird: Bird, config: RenderConfig): void;
function drawPipe(ctx: CanvasRenderingContext2D, pipe: Pipe, canvasHeight: number, config: RenderConfig): void;
function drawScore(ctx: CanvasRenderingContext2D, score: number, canvasWidth: number, config: RenderConfig): void;
function drawGameOver(ctx: CanvasRenderingContext2D, score: number, canvasWidth: number, canvasHeight: number, config: RenderConfig): void;
```

### InputHandler

Processes user input events.

```typescript
type JumpCallback = () => void;
type RestartCallback = () => void;

function setupInputListeners(
  canvas: HTMLCanvasElement,
  onJump: JumpCallback,
  onRestart: RestartCallback,
  gameState: GameState
): () => void; // Returns cleanup function
```

## Data Models

### Bird Model

The bird is represented as a rectangle with position and velocity:

- **Position**: `(x, y)` where x is fixed at approximately 1/4 of canvas width, y varies based on physics
- **Velocity**: Vertical velocity in pixels per frame (positive = downward, negative = upward)
- **Dimensions**: Fixed width and height for collision detection (e.g., 34x24 pixels)

### Pipe Model

Each pipe consists of two segments (top and bottom) with a gap:

- **Position**: `x` coordinate (horizontal position on canvas)
- **Gap**: Defined by `gapY` (center of gap) and `gapHeight` (size of gap)
- **Dimensions**: Fixed width (e.g., 52 pixels), height extends to canvas edges
- **Passed flag**: Boolean to track if bird has passed for scoring

The pipe rendering calculates:
- Top segment: from y=0 to y=(gapY - gapHeight/2)
- Bottom segment: from y=(gapY + gapHeight/2) to y=canvasHeight

### Score Model

Simple integer counter that increments when:
1. Bird's x position exceeds pipe's (x + width)
2. Pipe's `passed` flag is false
3. Set `passed` flag to true to prevent double-counting

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*


### Physics Properties

**Property 1: Bird horizontal position remains constant**
*For any* bird state during gameplay, the bird's x position should remain at the initial fixed position (not change during physics updates).
**Validates: Requirements 1.1**

**Property 2: Gravity increases downward velocity**
*For any* bird state, applying gravity should increase the velocity value (making it more positive/downward) or decrease upward velocity, bounded by terminal velocity.
**Validates: Requirements 1.2**

**Property 3: Position updates reflect velocity**
*For any* bird with a given velocity, updating the bird's position should change the y coordinate by exactly the velocity amount.
**Validates: Requirements 1.3**

**Property 4: Jump sets upward velocity**
*For any* bird state, applying jump should set the bird's velocity to the configured jump velocity (negative value for upward movement), regardless of the current velocity.
**Validates: Requirements 2.1, 2.2**

**Property 5: Pipe movement is consistent**
*For any* pipe, updating its position should decrease the x coordinate by exactly the configured pipe speed.
**Validates: Requirements 3.3**

### Collision Detection Properties

**Property 6: Bottom boundary collision detection**
*For any* bird with y position greater than or equal to (canvasHeight - bird.height), the boundary collision detector should return true.
**Validates: Requirements 1.4, 4.3**

**Property 7: Top boundary collision detection**
*For any* bird with y position less than or equal to 0, the boundary collision detector should return true.
**Validates: Requirements 4.2**

**Property 8: Bird-pipe collision detection**
*For any* bird and **valid** pipe (where gapY is within canvas bounds and gapHeight > 0) where the bird's bounding box intersects with either the top or bottom pipe segment, the collision detector should return true. A valid pipe must have its gap positioned such that both top and bottom pipe segments have non-negative heights.
**Validates: Requirements 4.1**

### Pipe Generation Properties

**Property 9: Pipe spawn timing**
*For any* frame count that is a multiple of the spawn interval, the spawn check function should return true.
**Validates: Requirements 3.1**

**Property 10: Pipe gap bounds**
*For any* generated pipe, the gapY value should be greater than or equal to minGapY and less than or equal to maxGapY, and the gapHeight should be greater than zero. Additionally, the gap must be positioned such that gapY - gapHeight/2 >= 0 (top pipe has non-negative height) and gapY + gapHeight/2 <= canvasHeight (bottom pipe segment starts within canvas).
**Validates: Requirements 3.2, 3.3, 3.4**

**Property 11: Offscreen pipe removal**
*For any* list of pipes, after filtering offscreen pipes, all remaining pipes should have x + width >= 0.
**Validates: Requirements 3.4**

### Score Properties

**Property 12: Score increments on pipe pass**
*For any* bird and pipe where bird.x > pipe.x + pipe.width and pipe.passed is false, the scoring logic should increment the score by 1 and set pipe.passed to true.
**Validates: Requirements 5.1**

**Property 13: Passed pipes don't score again (idempotence)**
*For any* pipe where passed is true, attempting to score again should not increment the score.
**Validates: Requirements 5.3**

**Property 14: Score preserved on game over**
*For any* game state transitioning to game_over status, the score value should remain unchanged.
**Validates: Requirements 5.4**

### State Management Properties

**Property 15: Ready to playing transition on first jump**
*For any* game state with status READY, processing a jump input should transition the status to PLAYING.
**Validates: Requirements 6.2**

**Property 16: Playing to game over transition on collision**
*For any* game state with status PLAYING where a collision is detected, the state should transition to GAME_OVER.
**Validates: Requirements 6.3**

**Property 17: Reset restores initial state**
*For any* game state, calling reset should restore the bird to initial position, clear all pipes, set score to 0, and set status to READY.
**Validates: Requirements 7.2, 7.3, 7.4, 7.5**

## Error Handling

The game handles several error conditions:

1. **Canvas initialization failure**: If canvas context cannot be obtained, log error and display message to user
2. **Invalid dimensions**: If canvas width or height is 0 or negative, use default dimensions (800x600)
3. **Invalid physics values**: Validate configuration values on initialization, use defaults if invalid
4. **Event listener errors**: Wrap event handlers in try-catch to prevent game crashes
5. **Animation frame errors**: If requestAnimationFrame fails, fall back to setTimeout with 16ms delay

Error handling strategy:
- Use TypeScript's type system to prevent invalid states at compile time
- Validate configuration values on initialization
- Gracefully degrade functionality rather than crashing
- Log errors to console for debugging
- Display user-friendly messages for critical failures

## Testing Strategy

The testing approach combines unit tests for specific examples and edge cases with property-based tests for universal correctness properties.

### Unit Testing

Unit tests focus on:
- **Specific examples**: Test concrete scenarios like "bird at y=100 with velocity=5 moves to y=105"
- **Edge cases**: Test boundary conditions like bird at exactly y=0 or y=canvasHeight
- **Error conditions**: Test invalid inputs like negative dimensions or null values
- **Integration points**: Test that components work together correctly

Example unit tests:
- Initial state has status READY and score 0
- Bird at bottom boundary triggers collision
- Empty pipe array after filtering when all pipes are offscreen
- Game over state ignores jump input

### Property-Based Testing

Property-based tests validate universal properties across many randomly generated inputs. Each test runs a minimum of 100 iterations with randomized data.

**Testing Library**: Use `fast-check` for TypeScript property-based testing

**Test Configuration**:
- Minimum 100 iterations per property test
- Each test tagged with: `Feature: flappy-bird-game, Property N: [property description]`
- Generate random but valid game states, bird positions, pipe configurations
- Use shrinking to find minimal failing cases

**Property Test Coverage**:
- All 17 correctness properties listed above
- Each property maps to one or more acceptance criteria
- Properties cover physics, collision detection, pipe generation, scoring, and state management

**Generator Strategy**:
- Generate random bird positions within canvas bounds
- Generate random velocities within reasonable ranges (-20 to +20)
- Generate random pipe positions and gap positions within valid bounds
- Generate random game states with various status values
- Include edge cases in generators (boundary positions, zero velocity, etc.)

**Example Property Test Structure**:
```typescript
// Feature: flappy-bird-game, Property 3: Position updates reflect velocity
test('bird position updates by velocity amount', () => {
  fc.assert(
    fc.property(
      fc.record({
        x: fc.integer(0, 800),
        y: fc.integer(0, 600),
        velocity: fc.integer(-20, 20),
        width: fc.constant(34),
        height: fc.constant(24)
      }),
      (bird) => {
        const initialY = bird.y;
        const updated = updateBirdPosition(bird);
        return updated.y === initialY + bird.velocity;
      }
    ),
    { numRuns: 100 }
  );
});
```

### Testing Balance

- **Unit tests**: Focus on specific examples, integration, and edge cases (approximately 20-30 tests)
- **Property tests**: Focus on universal properties across all inputs (17 property tests)
- Together they provide comprehensive coverage: unit tests catch concrete bugs, property tests verify general correctness
- Avoid writing too many unit tests for cases already covered by property tests
