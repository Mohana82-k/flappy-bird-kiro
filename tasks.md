# Implementation Plan: Flappy Bird Game

## Overview

This implementation plan breaks down the Flappy Bird game into incremental coding steps. The approach starts with project scaffolding, then builds core game logic (physics, collision detection, state management), adds rendering, integrates everything into a React component, and finally adds property-based tests to validate correctness properties.

## Tasks

- [x] 1. Set up Next.js project and install dependencies
  - Create new Next.js project with TypeScript and App Router
  - Install fast-check for property-based testing
  - Install Jest and React Testing Library for unit tests
  - Configure TypeScript with strict mode
  - Set up basic project structure with lib/ and app/ directories
  - _Requirements: 9.1, 9.3_

- [x] 2. Implement core data models and types
  - [x] 2.1 Create types.ts with GameStatus enum, Bird, Pipe, GameState interfaces
    - Define all TypeScript interfaces and enums from design
    - Include PhysicsConfig, PipeGeneratorConfig, RenderConfig types
    - _Requirements: 1.1, 1.2, 3.1, 3.2, 6.1_
  
  - [x] 2.2 Create gameState.ts with state initialization and reset functions
    - Implement createInitialState function
    - Implement resetGame function
    - _Requirements: 6.1, 7.2, 7.3, 7.4, 7.5_
  
  - [x]  2.3 Write unit tests for state initialization

    - Test initial state has READY status and score 0
    - Test reset clears pipes and resets score
    - _Requirements: 6.1, 7.2, 7.3, 7.4, 7.5_

- [x] 3. Implement physics engine
  - [x] 3.1 Create physics.ts with gravity, jump, and position update functions
    - Implement applyGravity function with terminal velocity
    - Implement applyJump function
    - Implement updateBirdPosition function
    - Implement updatePipePositions function
    - _Requirements: 1.2, 1.3, 2.1, 2.2, 3.3_
  
  - [x]  3.2 Write property test for gravity application

    - **Property 2: Gravity increases downward velocity**
    - **Validates: Requirements 1.2**
  
  - [x]  3.3 Write property test for position updates

    - **Property 3: Position updates reflect velocity**
    - **Validates: Requirements 1.3**
  
  - [x]  3.4 Write property test for jump velocity

    - **Property 4: Jump sets upward velocity**
    - **Validates: Requirements 2.1, 2.2**
  
  - [x]  3.5 Write property test for pipe movement

    - **Property 5: Pipe movement is consistent**
    - **Validates: Requirements 3.3**

- [x] 4. Implement collision detection
  - [x] 4.1 Create collision.ts with collision detection functions
    - Implement rectanglesIntersect helper function
    - Implement checkBirdPipeCollision function
    - Implement checkBirdBoundaryCollision function (top and bottom)
    - _Requirements: 1.4, 4.1, 4.2, 4.3_
  
  - [x]  4.2 Write property test for bottom boundary collision

    - **Property 6: Bottom boundary collision detection**
    - **Validates: Requirements 1.4, 4.3**
  
  - [x]  4.3 Write property test for top boundary collision

    - **Property 7: Top boundary collision detection**
    - **Validates: Requirements 4.2**
  
  - [x]  4.4 Write property test for bird-pipe collision

    - **Property 8: Bird-pipe collision detection**
    - **Validates: Requirements 4.1**
  
  - [x]  4.5 Write unit tests for edge cases

    - Test bird exactly at boundary
    - Test bird just inside safe zone
    - _Requirements: 1.4, 4.1, 4.2_

- [x] 5. Implement pipe generation and management
  - [x] 5.1 Create pipeGenerator.ts with pipe spawn and cleanup functions
    - Implement shouldSpawnPipe function
    - Implement generatePipe function with random gap positioning
    - Implement removeOffscreenPipes function
    - _Requirements: 3.1, 3.2, 3.4_
  
  - [x]  5.2 Write property test for pipe spawn timing

    - **Property 9: Pipe spawn timing**
    - **Validates: Requirements 3.1**
  
  - [x]  5.3 Write property test for pipe gap bounds

    - **Property 10: Pipe gap bounds**
    - **Validates: Requirements 3.2**
  
  - [x]  5.4 Write property test for offscreen pipe removal

    - **Property 11: Offscreen pipe removal**
    - **Validates: Requirements 3.4**

- [x] 6. Implement scoring logic
  - [x] 6.1 Create scoring.ts with score update functions
    - Implement checkPipePassed function
    - Implement updateScore function that marks pipes as passed
    - _Requirements: 5.1, 5.3, 5.4_
  
  - [x]  6.2 Write property test for score increment

    - **Property 12: Score increments on pipe pass**
    - **Validates: Requirements 5.1**
  
  - [x]  6.3 Write property test for idempotent scoring

    - **Property 13: Passed pipes don't score again**
    - **Validates: Requirements 5.3**
  
  - [x]  6.4 Write property test for score preservation

    - **Property 14: Score preserved on game over**
    - **Validates: Requirements 5.4**

- [x] 7. Checkpoint - Ensure core game logic tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 8. Implement rendering functions
  - [x] 8.1 Create renderer.ts with canvas drawing functions
    - Implement clearCanvas function
    - Implement drawBird function
    - Implement drawPipe function (top and bottom segments)
    - Implement drawScore function
    - Implement drawGameOver function
    - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5_
  
  - [x]  8.2 Write unit tests for rendering calculations

    - Test pipe segment position calculations
    - Test score positioning in top right
    - _Requirements: 8.3, 8.4_

- [x] 9. Implement state management logic
  - [x] 9.1 Create stateManager.ts with state transition functions
    - Implement handleJumpInput function (ready -> playing transition)
    - Implement handleCollision function (playing -> game_over transition)
    - Implement handleGameUpdate function (orchestrates physics, collision, scoring)
    - _Requirements: 6.2, 6.3, 6.4_
  
  - [x]  9.2 Write property test for ready to playing transition

    - **Property 15: Ready to playing transition on first jump**
    - **Validates: Requirements 6.2**
  
  - [x]  9.3 Write property test for collision transition

    - **Property 16: Playing to game over transition on collision**
    - **Validates: Requirements 6.3**
  
  - [x]  9.4 Write property test for reset function

    - **Property 17: Reset restores initial state**
    - **Validates: Requirements 7.2, 7.3, 7.4, 7.5**

- [x] 10. Create React game component
  - [x] 10.1 Create components/Game.tsx as client component
    - Add 'use client' directive
    - Set up canvas ref and useEffect for initialization
    - Implement game loop with requestAnimationFrame
    - Set up event listeners for keyboard and mouse input
    - Implement cleanup on unmount
    - _Requirements: 9.1, 9.2, 9.3_
  
  - [x] 10.2 Integrate all game logic modules into game loop
    - Call physics updates on each frame
    - Call collision detection on each frame
    - Call pipe generation and cleanup
    - Call scoring logic
    - Call state transitions based on game status
    - Call renderer to draw all elements
    - _Requirements: 1.1, 1.2, 1.3, 2.1, 3.1, 3.3, 4.1, 4.4, 5.1, 6.2_
  
  - [x] 10.3 Add restart button UI for game over state
    - Render button when status is GAME_OVER
    - Wire button click to resetGame function
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5_

- [x] 11. Create main page and wire up game component
  - [x] 11.1 Update app/page.tsx to render Game component
    - Import and render Game component
    - Add basic page styling
    - _Requirements: 9.1, 9.3_
  
  - [x] 11.2 Add global styles for canvas and layout
    - Style canvas to be centered
    - Add responsive container
    - Style restart button
    - _Requirements: 8.1, 8.2_

- [x] 12. Final checkpoint - Run all tests and verify gameplay
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Property tests validate universal correctness properties with 100+ iterations
- Unit tests validate specific examples and edge cases
- The game uses HTML5 Canvas for rendering and requestAnimationFrame for smooth animation
- All game logic is separated into pure functions for testability
