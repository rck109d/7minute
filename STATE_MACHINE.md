# 7-Minute Workout State Machine

## States
1. **idle** - Initial state, no workout started
2. **ready** - 10-second countdown before an exercise
3. **exercise** - 30-second exercise period
4. **rest** - 10-second rest between exercises
5. **paused** - Workout is paused (stores previousPhase)
6. **completed** - All exercises finished

## Button Availability by State

| Button | idle | ready | exercise | rest | paused | completed |
|--------|------|-------|----------|------|--------|-----------|
| Play/Pause | ▶️ Start (green) | ⏸️ Pause (yellow) | ⏸️ Pause (yellow) | ⏸️ Pause (yellow) | ▶️ Resume (blue) | ▶️ Start (green) |
| Skip | ❌ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Restart | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Sound | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Exercise List | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

**Note:** The Play/Pause button is a single unified button that changes label, color, and behavior based on state.

## State Transitions

### From `idle`
- **Play/Pause (Start)** → `ready` (first exercise)
- **Exercise click** → `ready` (jump to exercise)

### From `ready`
- **Timer reaches 0** → `exercise`
- **Play/Pause (Pause)** → `paused` (previousPhase=ready)
- **Skip** → `ready` (next exercise) OR `completed` (if last)
- **Restart** → `paused` (shows dialog) → `idle` or resume
- **Exercise click** → `ready` (restart current or jump to another)

### From `exercise`
- **Timer reaches 0** → `rest` (if more exercises) OR `completed` (if last)
- **Play/Pause (Pause)** → `paused` (previousPhase=exercise)
- **Skip** → `ready` (next exercise) OR `completed` (if last)
- **Restart** → `paused` (shows dialog) → `idle` or resume
- **Exercise click** → `ready` (jump to exercise)

### From `rest`
- **Timer reaches 0** → `exercise` (next exercise)
- **Play/Pause (Pause)** → `paused` (previousPhase=rest)
- **Skip** → `ready` (next exercise) OR `completed` (if last)
- **Restart** → `paused` (shows dialog) → `idle` or resume
- **Exercise click** → `ready` (jump to exercise)

### From `paused`
- **Play/Pause (Resume)** → `previousPhase` (ready/exercise/rest)
- **Play/Pause (Resume, timer=0)** → advance from previousPhase
- **Skip** → `ready` (next exercise) OR `completed` (if last)
- **Restart** → shows dialog → `idle` or resume paused
- **Exercise click** → `ready` (jump to exercise)

### From `completed`
- **Play/Pause (Start)** → `ready` (starts new workout immediately)
- **Restart** → `ready` (starts new workout immediately, no dialog)
- **Exercise click** → `ready` (jump to exercise)

## State Invariants

1. **previousPhase** is only set when entering `paused` state
2. **previousPhase** is cleared when entering any active state (ready/exercise/rest)
3. From `paused`, you can always return to a valid active state via Resume or Skip
4. From `completed`, you can restart or jump to any exercise
5. Exercise list clicks always go to `ready` state regardless of current state
6. Timer only runs in `ready`, `exercise`, and `rest` states
7. All state resets (Start Fresh, restart from completed) clear previousPhase

## Dead End Prevention

- No state is unreachable from any other state
- `completed` can exit via Restart or Exercise click
- `paused` can exit via Resume, Skip, Restart, or Exercise click
- All active states can be paused and resumed
- Exercise list provides universal navigation to any exercise from any state

## UI Layout

1. **Audio Controls** (Voice & Volume) - Collapsible on mobile via Sound button
2. **Control Buttons** - Play/Pause, Skip, Restart, Sound (always visible)
3. **Workout Status** - Timer and status badge ("GET READY", "EXERCISE", "REST", "PAUSED")
4. **Exercise List** - Full-width buttons for all 13 exercises

## Button Colors

- **Green** - Start button (idle/completed states)
- **Blue** - Resume button (paused state)
- **Yellow** - Pause button (active workout), Skip button
- **Red** - Restart button
- **Gray** - Sound button (secondary), disabled buttons

## Exercise Button Colors

- **Gray** (`--bg-secondary`) - Idle/not started exercises
- **Green** (`--color-exercise`) - Completed exercises
- **Blue** (`--color-ready`) - Current exercise during exercise phase
- **Orange** (`--color-rest`) - Current exercise during ready/rest phases

## Audio Features

- **Speech Synthesis** - Announces exercises, countdowns (3-2-1), "halfway" at 15s
- **Voice Selection** - Dropdown with system voices (prefers en-US by default)
- **Volume Control** - 0-100% slider
- **Sound Effects** - Whistle at countdown 0, completion sound
- **Settings Persistence** - Voice and volume saved to localStorage

