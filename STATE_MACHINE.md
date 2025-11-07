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
| Start | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Pause | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Resume | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Skip | ❌ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Restart | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Exercise List | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

## State Transitions

### From `idle`
- **Start** → `ready` (first exercise)
- **Exercise click** → `ready` (jump to exercise)

### From `ready`
- **Timer reaches 0** → `exercise`
- **Pause** → `paused` (previousPhase=ready)
- **Skip** → `ready` (next exercise) OR `completed` (if last)
- **Restart** → `paused` (shows dialog) → `idle` or resume
- **Exercise click** → `ready` (restart current or jump to another)

### From `exercise`
- **Timer reaches 0** → `rest` (if more exercises) OR `completed` (if last)
- **Pause** → `paused` (previousPhase=exercise)
- **Skip** → `ready` (next exercise) OR `completed` (if last)
- **Restart** → `paused` (shows dialog) → `idle` or resume
- **Exercise click** → `ready` (jump to exercise)

### From `rest`
- **Timer reaches 0** → `exercise` (next exercise)
- **Pause** → `paused` (previousPhase=rest)
- **Skip** → `ready` (next exercise) OR `completed` (if last)
- **Restart** → `paused` (shows dialog) → `idle` or resume
- **Exercise click** → `ready` (jump to exercise)

### From `paused`
- **Resume** → `previousPhase` (ready/exercise/rest)
- **Resume (timer=0)** → advance from previousPhase
- **Skip** → `ready` (next exercise) OR `completed` (if last)
- **Restart** → shows dialog → `idle` or resume paused
- **Exercise click** → `ready` (jump to exercise)

### From `completed`
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

