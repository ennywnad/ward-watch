# NICU Game Enhancement Implementation Guide

## For AI Coding Agent

**Project:** Shift Goals: NICU Edition
**File:** `index.html` (single-file vanilla JS game)
**Repository:** `ennywnad/ward-watch`

---

## Codebase Overview

### Architecture
- Single HTML file with embedded `<script>` and `<style>` tags
- Canvas-based rendering with HTML overlay for HUD
- No build system, no frameworks, no npm
- Uses Tailwind via CDN for UI styling
- Font: VT323 (pixel/retro style)

### Key Global Variables
```javascript
gameState = {
    mode: 'MENU' | 'GAME' | 'MUG',
    running: boolean,
    paused: boolean,
    time: number,        // Minutes since midnight (420 = 7am)
    sanity: 0-100,
    score: number,
    coffeeBoost: number, // ms remaining
    shortStaffed: boolean,
    nightShift: boolean,
    // ... more
}

entities = []        // All game objects
particles = []       // Floating text effects
isolettes = []       // The 6 baby stations
nurse = Nurse        // Player character
chair = Chair        // Healing station
```

### Entity Class Hierarchy
```
Entity (base)
├── Nurse (player)
├── Isolette (babies)
├── Enemy (resident, surgeon, mom, dad, floorcleaner, etc.)
├── Item (coffee, pizza, sanitizer, etc.)
├── Machine (decorative equipment)
├── Chair
├── ChartingStation
├── SupplyCart
└── Pager
```

### Sprite System
- `SPRITES` object contains pixel art as string arrays
- `COLORS` maps characters to hex colors
- `drawSprite(ctx, spriteKey, x, y, scale)` renders sprites

### Key Functions
- `initGame()` - Reset and setup
- `update(dt)` - Game logic tick
- `draw()` - Render frame
- `loop(timestamp)` - Main game loop via requestAnimationFrame
- `spawnEnemy()` / `spawnItem()` - Entity spawning
- `addLogEntry(speaker, message)` - Chat log
- `playSoundVisual(x, y, text)` - Floating text particle
- `playBeep(freq, duration)` - Audio feedback

---

## Feature 1: WASD Movement

### Goal
Add keyboard controls for nurse movement alongside existing click-to-move.

### Implementation Steps

1. **Add keyboard state tracking** at the top of the script:
```javascript
const keys = {
    w: false, a: false, s: false, d: false,
    up: false, down: false, left: false, right: false
};
```

2. **Add event listeners** after the existing `window.addEventListener` calls:
```javascript
window.addEventListener('keydown', (e) => {
    const key = e.key.toLowerCase();
    if (key === 'w' || key === 'arrowup') keys.w = true;
    if (key === 'a' || key === 'arrowleft') keys.a = true;
    if (key === 's' || key === 'arrowdown') keys.s = true;
    if (key === 'd' || key === 'arrowright') keys.d = true;
});

window.addEventListener('keyup', (e) => {
    const key = e.key.toLowerCase();
    if (key === 'w' || key === 'arrowup') keys.w = false;
    if (key === 'a' || key === 'arrowleft') keys.a = false;
    if (key === 's' || key === 'arrowdown') keys.s = false;
    if (key === 'd' || key === 'arrowright') keys.d = false;
});
```

3. **Modify `Nurse.update()`** to check keyboard input:
```javascript
// At the START of Nurse.update(), before existing logic:
if (keys.w || keys.a || keys.s || keys.d) {
    // Cancel sitting state
    if (this.state === 'sitting') {
        this.state = 'idle';
    }
    
    // Cancel click-to-move target
    this.target = null;
    this.state = 'idle';
    
    // Calculate movement vector
    let dx = 0, dy = 0;
    if (keys.w) dy -= 1;
    if (keys.s) dy += 1;
    if (keys.a) dx -= 1;
    if (keys.d) dx += 1;
    
    // Normalize diagonal movement
    if (dx !== 0 && dy !== 0) {
        dx *= 0.707;
        dy *= 0.707;
    }
    
    // Apply movement
    if (dx !== 0 || dy !== 0) {
        this.pos.x += dx * this.speed;
        this.pos.y += dy * this.speed;
        
        // Clamp to canvas bounds
        this.pos.x = Math.max(30, Math.min(canvas.width - 30, this.pos.x));
        this.pos.y = Math.max(30, Math.min(canvas.height - 30, this.pos.y));
        
        this.state = 'walking'; // For animation system
    }
}
```

4. **Add 'walking' state handling** - ensure it doesn't break existing idle checks.

### Testing
- WASD moves nurse in 8 directions
- Arrow keys also work
- Diagonal movement isn't faster than cardinal
- Nurse stops at canvas edges
- Click-to-move still works
- Keyboard interrupts click-to-move path

---

## Feature 2: Night Shift Visuals

### Goal
When `gameState.nightShift = true`, apply dark atmosphere with vignette and flickering.

### Implementation Steps

1. **Add flicker state** to gameState:
```javascript
gameState.flickerTimer = 0;
gameState.flickerIntensity = 0;
```

2. **Create overlay rendering function** (call at END of `draw()`):
```javascript
function drawNightShiftOverlay() {
    if (!gameState.nightShift) return;
    
    // Darken the entire screen
    ctx.save();
    ctx.fillStyle = 'rgba(0, 0, 20, 0.3)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    // Vignette effect
    const gradient = ctx.createRadialGradient(
        canvas.width / 2, canvas.height / 2, canvas.height * 0.3,
        canvas.width / 2, canvas.height / 2, canvas.height * 0.8
    );
    gradient.addColorStop(0, 'rgba(0, 0, 0, 0)');
    gradient.addColorStop(1, 'rgba(0, 0, 30, 0.5)');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    // Flicker effect (random light fluctuation)
    if (gameState.flickerIntensity > 0) {
        ctx.fillStyle = `rgba(255, 255, 200, ${gameState.flickerIntensity * 0.1})`;
        ctx.fillRect(0, 0, canvas.width, canvas.height);
    }
    
    ctx.restore();
}
```

3. **Update flicker in `update()`**:
```javascript
// Add to update() function
if (gameState.nightShift) {
    gameState.flickerTimer -= dt;
    if (gameState.flickerTimer <= 0) {
        // Random chance of flicker
        if (Math.random() < 0.1) {
            gameState.flickerIntensity = Math.random() * 3;
        } else {
            gameState.flickerIntensity = 0;
        }
        gameState.flickerTimer = 100 + Math.random() * 200;
    }
}
```

4. **Modify floor color** in `draw()`:
```javascript
// Change background based on night shift
ctx.fillStyle = gameState.nightShift ? '#1a1a2e' : '#2d3748';
```

5. **Add blue-ish tint to grid lines**:
```javascript
ctx.strokeStyle = gameState.nightShift ? '#2a2a4a' : '#3a4659';
```

6. **Call overlay at end of draw()**:
```javascript
// After all entities drawn, before pause overlay
drawNightShiftOverlay();
```

### Visual Notes
- Keep UI elements (HUD) fully visible - don't darken those
- Isolette glow from monitors should be more prominent at night
- Consider adding small "window" rectangles that show moonlight

---

## Feature 3: Settings Menu

### Goal
Add a settings modal accessible from pause screen with volume and difficulty toggles.

### Implementation Steps

1. **Add settings state**:
```javascript
const settings = {
    volume: 0.5,
    nightShift: false,
    shortStaffed: false,
    tutorialSeen: false
};

// Load from localStorage on page load
function loadSettings() {
    const saved = localStorage.getItem('nicuSettings');
    if (saved) {
        Object.assign(settings, JSON.parse(saved));
        gameState.nightShift = settings.nightShift;
        gameState.shortStaffed = settings.shortStaffed;
    }
}

function saveSettings() {
    localStorage.setItem('nicuSettings', JSON.stringify(settings));
}
```

2. **Add settings modal HTML** (inside `#game-container`, after modal-overlay):
```html
<div id="settings-overlay" style="display: none;">
    <div class="modal-content" style="max-width: 400px;">
        <h2 class="text-3xl mb-4 text-blue-300">SETTINGS</h2>
        
        <div class="text-left space-y-4">
            <div class="flex justify-between items-center">
                <label>Volume</label>
                <input type="range" id="volume-slider" min="0" max="100" value="50" 
                       class="w-32" onchange="updateVolume(this.value)">
            </div>
            
            <div class="flex justify-between items-center">
                <label>Night Shift Mode</label>
                <button id="night-toggle" class="btn text-sm py-1" onclick="toggleNightShift()">
                    OFF
                </button>
            </div>
            
            <div class="flex justify-between items-center">
                <label>Short Staffed Mode</label>
                <button id="staff-toggle" class="btn text-sm py-1" onclick="toggleShortStaffed()">
                    OFF
                </button>
            </div>
            
            <div class="flex justify-between items-center">
                <label>Reset Tutorial</label>
                <button class="btn text-sm py-1" onclick="resetTutorial()">
                    RESET
                </button>
            </div>
        </div>
        
        <button class="btn mt-6" onclick="closeSettings()">DONE</button>
    </div>
</div>
```

3. **Add CSS for settings overlay**:
```css
#settings-overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.9);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 150;
    pointer-events: auto;
}

#settings-overlay input[type="range"] {
    cursor: pointer;
}
```

4. **Add settings functions**:
```javascript
function openSettings() {
    document.getElementById('settings-overlay').style.display = 'flex';
    updateSettingsUI();
}

function closeSettings() {
    document.getElementById('settings-overlay').style.display = 'none';
    saveSettings();
}

function updateSettingsUI() {
    document.getElementById('volume-slider').value = settings.volume * 100;
    document.getElementById('night-toggle').innerText = settings.nightShift ? 'ON' : 'OFF';
    document.getElementById('night-toggle').style.background = settings.nightShift ? '#48bb78' : '#e53e3e';
    document.getElementById('staff-toggle').innerText = settings.shortStaffed ? 'ON' : 'OFF';
    document.getElementById('staff-toggle').style.background = settings.shortStaffed ? '#48bb78' : '#e53e3e';
}

function updateVolume(val) {
    settings.volume = val / 100;
    // Apply to audio context gain if implemented
}

function toggleNightShift() {
    settings.nightShift = !settings.nightShift;
    gameState.nightShift = settings.nightShift;
    updateSettingsUI();
}

function toggleShortStaffed() {
    settings.shortStaffed = !settings.shortStaffed;
    gameState.shortStaffed = settings.shortStaffed;
    updateSettingsUI();
}

function resetTutorial() {
    settings.tutorialSeen = false;
    saveSettings();
    alert('Tutorial will show on next shift!');
}
```

5. **Add settings button to pause screen or main menu**:
```html
<!-- Add to modal-content in start menu -->
<button class="btn" onclick="openSettings()" style="margin-top: 0.5rem; background: #718096;">
    ⚙ SETTINGS
</button>
```

6. **Modify `playBeep()`** to respect volume:
```javascript
function playBeep(frequency = 440, duration = 100) {
    if (!audioCtx || settings.volume === 0) return;
    try {
        const oscillator = audioCtx.createOscillator();
        const gainNode = audioCtx.createGain();
        oscillator.connect(gainNode);
        gainNode.connect(audioCtx.destination);
        oscillator.frequency.value = frequency;
        gainNode.gain.setValueAtTime(0.3 * settings.volume, audioCtx.currentTime);
        // ... rest unchanged
    }
}
```

7. **Call `loadSettings()`** at page load.

---

## Feature 4: Shift Summary Stats

### Goal
Track and display end-of-shift statistics before the mug screen.

### Implementation Steps

1. **Add stats tracking object**:
```javascript
const shiftStats = {
    babiesFed: 0,
    diapersChanged: 0,
    babiesCalmed: 0,
    enemiesDismissed: 0,
    coffeeConsumed: 0,
    bathroomBreaks: 0,
    stepsWalked: 0,
    chartingCompleted: 0,
    peakSanity: 100,
    lowestSanity: 100,
    timesCritical: 0  // Sanity below 20
};

function resetShiftStats() {
    Object.keys(shiftStats).forEach(k => shiftStats[k] = 0);
    shiftStats.peakSanity = 100;
    shiftStats.lowestSanity = 100;
}
```

2. **Track stats throughout gameplay** - add to relevant locations:

```javascript
// In isolette need handling:
if (need === 'feed') {
    shiftStats.babiesFed++;
    // ... existing code
}
if (need === 'diaper') {
    shiftStats.diapersChanged++;
    // ...
}
if (need === 'calm') {
    shiftStats.babiesCalmed++;
    // ...
}

// In enemy dismissal:
enemy.leave();
shiftStats.enemiesDismissed++;

// In item pickup:
if (item.type === 'coffee') {
    shiftStats.coffeeConsumed++;
    // ...
}
if (item.type === 'bathroom') {
    shiftStats.bathroomBreaks++;
    // ...
}

// In Nurse.update() when moving (rough step counter):
if (this.state === 'moving' || this.state === 'walking') {
    shiftStats.stepsWalked += this.speed * 0.01;
}

// In charting station completion:
shiftStats.chartingCompleted++;

// In main update(), track sanity extremes:
shiftStats.peakSanity = Math.max(shiftStats.peakSanity, gameState.sanity);
shiftStats.lowestSanity = Math.min(shiftStats.lowestSanity, gameState.sanity);
if (gameState.sanity < 20) shiftStats.timesCritical++;
```

3. **Add summary screen mode**:
```javascript
// Modify endGame() to show summary first
function endGame(victory) {
    gameState.mode = 'SUMMARY';
    // ... existing setup
}
```

4. **Add summary drawing function**:
```javascript
function drawSummaryScreen() {
    ctx.fillStyle = '#1a202c';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    ctx.fillStyle = 'white';
    ctx.font = '48px "Vt323"';
    ctx.textAlign = 'center';
    ctx.fillText('SHIFT COMPLETE', canvas.width / 2, 80);
    
    ctx.font = '28px "Vt323"';
    ctx.textAlign = 'left';
    const startX = canvas.width / 2 - 200;
    let y = 150;
    const lineHeight = 40;
    
    const stats = [
        ['Babies Fed', shiftStats.babiesFed],
        ['Diapers Changed', shiftStats.diapersChanged],
        ['Babies Calmed', shiftStats.babiesCalmed],
        ['People Handled', shiftStats.enemiesDismissed],
        ['Coffee Consumed', shiftStats.coffeeConsumed],
        ['Bathroom Breaks', shiftStats.bathroomBreaks],
        ['Steps Walked', Math.floor(shiftStats.stepsWalked)],
        ['Charts Completed', shiftStats.chartingCompleted],
        ['', ''],  // Spacer
        ['Peak Sanity', Math.floor(shiftStats.peakSanity) + '%'],
        ['Lowest Sanity', Math.floor(shiftStats.lowestSanity) + '%'],
        ['Times Critical', shiftStats.timesCritical],
    ];
    
    stats.forEach(([label, value]) => {
        if (label === '') { y += 20; return; }
        ctx.fillStyle = '#a0aec0';
        ctx.fillText(label + ':', startX, y);
        ctx.fillStyle = '#68d391';
        ctx.textAlign = 'right';
        ctx.fillText(String(value), startX + 400, y);
        ctx.textAlign = 'left';
        y += lineHeight;
    });
    
    // Final score
    ctx.font = '36px "Vt323"';
    ctx.fillStyle = '#ecc94b';
    ctx.textAlign = 'center';
    ctx.fillText(`FINAL SCORE: ${gameState.score}`, canvas.width / 2, y + 60);
    
    // Continue prompt
    ctx.font = '24px "Vt323"';
    ctx.fillStyle = '#718096';
    ctx.fillText('Click anywhere to continue...', canvas.width / 2, canvas.height - 50);
}
```

5. **Modify draw() and input handling**:
```javascript
function draw() {
    if (gameState.mode === 'SUMMARY') {
        drawSummaryScreen();
        return;
    }
    if (gameState.mode === 'MUG') {
        // ... existing
    }
    // ... rest of draw
}

// In handlePointerDown:
if (gameState.mode === 'SUMMARY') {
    gameState.mode = 'MUG';
    // Initialize mug screen (move existing endGame mug setup here)
    initMugScreen();
    return;
}
```

6. **Extract mug initialization** from endGame into separate function.

7. **Call `resetShiftStats()`** in `initGame()`.

---

## Feature 5: Report-Off / Incoming Nurse

### Goal
At shift end, an incoming nurse walks in with a big mug. Brief visual before mug screen.

### Implementation Steps

1. **Add report-off state**:
```javascript
gameState.reportOffPhase = false;
gameState.incomingNurse = null;
gameState.reportOffTimer = 0;
```

2. **Create incoming nurse sprite** (add to SPRITES):
```javascript
incoming_nurse: [
    "___PPPP___",  // Pink scrub cap
    "___PPPP___",
    "___SSSS___",
    "___SRSS___",
    "__PPPPPP__",  // Pink scrubs
    "__PPPPPP__",
    "_PPWWPPPP_",
    "_PPPPPPPP_",
    "__PP__PP__",
    "__NW__NW__"
],
big_mug: [
    "___WWW____",
    "__OOOOO_W_",
    "__OOOOO_W_",
    "__OOOOO_W_",
    "__OOOOO___",
    "___OOO____"
]
```

3. **Modify endGame()**:
```javascript
function endGame(victory) {
    if (victory) {
        // Start report-off sequence
        gameState.mode = 'REPORTOFF';
        gameState.reportOffPhase = true;
        gameState.reportOffTimer = 0;
        
        // Create incoming nurse at door
        gameState.incomingNurse = {
            x: -100,
            y: canvas.height / 2,
            targetX: canvas.width / 2 - 100,
            arrived: false,
            bubbleText: null
        };
        
        addLogEntry('admin', "Night shift arriving...");
    } else {
        // Burnout - skip to mug
        gameState.mode = 'SUMMARY';
    }
}
```

4. **Add report-off update logic**:
```javascript
function updateReportOff(dt) {
    const inc = gameState.incomingNurse;
    
    // Walk in
    if (inc.x < inc.targetX) {
        inc.x += 3;
    } else if (!inc.arrived) {
        inc.arrived = true;
        inc.bubbleText = "How were they?";
        addLogEntry('nurse', "Incoming: How were they tonight?");
        gameState.reportOffTimer = 0;
    }
    
    // After arriving, show dialogue
    if (inc.arrived) {
        gameState.reportOffTimer += dt;
        
        if (gameState.reportOffTimer > 2000 && !inc.responseGiven) {
            inc.responseGiven = true;
            
            // Generate response based on shift
            const responses = [];
            if (shiftStats.babiesFed > 10) responses.push("Fed everyone on time.");
            if (shiftStats.timesCritical > 5) responses.push("It was rough.");
            if (shiftStats.coffeeConsumed > 3) responses.push("Needed a lot of coffee.");
            if (gameState.sanity > 50) responses.push("Not bad, actually.");
            else responses.push("You'll need that coffee.");
            
            const response = responses[Math.floor(Math.random() * responses.length)] || "It was a shift.";
            nurse.say(response, 'nurse');
            inc.bubbleText = null;
        }
        
        if (gameState.reportOffTimer > 4500) {
            inc.bubbleText = "I got it from here.";
            addLogEntry('nurse', "Incoming: I got it from here. Go home!");
        }
        
        if (gameState.reportOffTimer > 6000) {
            gameState.mode = 'SUMMARY';
        }
    }
}
```

5. **Add report-off drawing**:
```javascript
function drawReportOff() {
    // Draw normal game background (frozen)
    ctx.fillStyle = '#2d3748';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    // Draw grid, entities frozen...
    entities.forEach(e => e.draw(ctx));
    
    const inc = gameState.incomingNurse;
    
    // Draw incoming nurse
    drawSprite(ctx, 'incoming_nurse', inc.x, inc.y);
    
    // Draw their big mug
    drawSprite(ctx, 'big_mug', inc.x + 40, inc.y + 10, PIXEL_SCALE * 1.5);
    
    // Speech bubble
    if (inc.bubbleText) {
        ctx.fillStyle = 'white';
        ctx.strokeStyle = 'black';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.roundRect(inc.x - 60, inc.y - 100, 120, 50, 5);
        ctx.fill();
        ctx.stroke();
        
        ctx.fillStyle = 'black';
        ctx.font = '16px "Vt323"';
        ctx.textAlign = 'center';
        ctx.fillText(inc.bubbleText, inc.x, inc.y - 70);
    }
    
    // "SHIFT CHANGE" banner
    ctx.fillStyle = 'rgba(0,0,0,0.7)';
    ctx.fillRect(0, 50, canvas.width, 60);
    ctx.fillStyle = '#ecc94b';
    ctx.font = '48px "Vt323"';
    ctx.textAlign = 'center';
    ctx.fillText('SHIFT CHANGE', canvas.width / 2, 95);
}
```

6. **Update main loop**:
```javascript
function update(dt) {
    if (gameState.mode === 'REPORTOFF') {
        updateReportOff(dt);
        return;
    }
    // ... existing
}

function draw() {
    if (gameState.mode === 'REPORTOFF') {
        drawReportOff();
        return;
    }
    // ... existing
}
```

---

## Feature 6: Tutorial Overlay

### Goal
First-time players see contextual hints that fade as they learn.

### Implementation Steps

1. **Add tutorial state**:
```javascript
const tutorial = {
    active: false,
    step: 0,
    seen: new Set(),
    hintTimer: 0,
    currentHint: null
};

const tutorialHints = [
    { trigger: 'start', text: 'Click or use WASD to move your nurse', x: 0.5, y: 0.7 },
    { trigger: 'baby_need', text: 'Walk to babies to help them!', x: 0.3, y: 0.4 },
    { trigger: 'enemy_spawn', text: 'Walk up to people to dismiss them', x: 0.7, y: 0.5 },
    { trigger: 'sanity_low', text: 'Find coffee or take a break!', x: 0.5, y: 0.3 },
    { trigger: 'coffee_spawn', text: 'Grab the coffee for a speed boost!', x: 0.5, y: 0.5 },
    { trigger: 'charting', text: 'Visit the computer to chart', x: 0.85, y: 0.15 },
    { trigger: 'chair', text: 'Sit in the chair to recover sanity', x: 0.6, y: 0.85 }
];
```

2. **Show tutorial on first play**:
```javascript
function initGame() {
    // ... existing init
    
    if (!settings.tutorialSeen) {
        tutorial.active = true;
        tutorial.step = 0;
        tutorial.seen = new Set();
        showTutorialHint('start');
    }
}
```

3. **Add hint trigger function**:
```javascript
function showTutorialHint(trigger) {
    if (!tutorial.active) return;
    if (tutorial.seen.has(trigger)) return;
    
    const hint = tutorialHints.find(h => h.trigger === trigger);
    if (hint) {
        tutorial.currentHint = hint;
        tutorial.hintTimer = 5000; // Show for 5 seconds
        tutorial.seen.add(trigger);
    }
    
    // End tutorial after all hints seen
    if (tutorial.seen.size >= tutorialHints.length) {
        settings.tutorialSeen = true;
        saveSettings();
    }
}
```

4. **Trigger hints at appropriate moments** in update():
```javascript
// In spawnEnemy():
if (/* enemy spawned */) {
    showTutorialHint('enemy_spawn');
}

// When isolette gets a need:
showTutorialHint('baby_need');

// When coffee spawns:
if (type === 'coffee') showTutorialHint('coffee_spawn');

// When sanity drops below 40:
if (gameState.sanity < 40) showTutorialHint('sanity_low');

// First time near charting station:
if (nurse.pos.dist(chartingStation.pos) < 150) showTutorialHint('charting');

// First time near chair:
if (nurse.pos.dist(chair.pos) < 150) showTutorialHint('chair');
```

5. **Update tutorial timer**:
```javascript
// In update():
if (tutorial.currentHint) {
    tutorial.hintTimer -= dt;
    if (tutorial.hintTimer <= 0) {
        tutorial.currentHint = null;
    }
}
```

6. **Draw tutorial overlay** (call after everything else in draw()):
```javascript
function drawTutorialHint() {
    if (!tutorial.currentHint) return;
    
    const hint = tutorial.currentHint;
    const x = canvas.width * hint.x;
    const y = canvas.height * hint.y;
    
    // Pulsing glow
    const pulse = Math.sin(Date.now() / 200) * 0.2 + 0.8;
    
    // Background box
    ctx.save();
    ctx.globalAlpha = pulse;
    ctx.fillStyle = 'rgba(0, 0, 0, 0.85)';
    ctx.strokeStyle = '#ecc94b';
    ctx.lineWidth = 3;
    
    const textWidth = ctx.measureText(hint.text).width + 40;
    ctx.beginPath();
    ctx.roundRect(x - textWidth / 2, y - 25, textWidth, 50, 10);
    ctx.fill();
    ctx.stroke();
    
    // Text
    ctx.globalAlpha = 1;
    ctx.fillStyle = '#fff';
    ctx.font = '24px "Vt323"';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(hint.text, x, y);
    
    // Skip prompt
    ctx.font = '16px "Vt323"';
    ctx.fillStyle = '#718096';
    ctx.fillText('(Click anywhere to dismiss)', x, y + 35);
    
    ctx.restore();
}
```

7. **Allow dismissing hints with click**:
```javascript
// In handlePointerDown:
if (tutorial.currentHint) {
    tutorial.currentHint = null;
    return; // Don't move nurse
}
```

---

## Feature 7: Sprite Animations

### Goal
Add walking animation for nurse, wiggle for crying babies.

### Implementation Steps

1. **Create animation frame sprites**:
```javascript
// Add walking frames to SPRITES
nurse_walk1: [
    "___WWWW___",
    "___WWWW___",
    "___SSSS___",
    "___SRSS___",
    "__BBBBBB__",
    "__BBBBBB__",
    "_BBWWBBBB_",
    "_BBBBBBBB_",
    "_BB____BB_",  // Legs apart
    "_NW____NW_"
],
nurse_walk2: [
    "___WWWW___",
    "___WWWW___",
    "___SSSS___",
    "___SRSS___",
    "__BBBBBB__",
    "__BBBBBB__",
    "_BBWWBBBB_",
    "_BBBBBBBB_",
    "__BBBB____",  // Legs together, shifted
    "__NWNW____"
],
```

2. **Add animation state to Nurse**:
```javascript
class Nurse extends Entity {
    constructor(x, y) {
        // ... existing
        this.animFrame = 0;
        this.animTimer = 0;
        this.facing = 1; // 1 = right, -1 = left
    }
    
    update(dt) {
        // Track facing direction
        if (this.target && this.state === 'moving') {
            if (this.target.x < this.pos.x) this.facing = -1;
            else if (this.target.x > this.pos.x) this.facing = 1;
        }
        
        // Update animation
        if (this.state === 'moving' || this.state === 'walking') {
            this.animTimer += dt;
            if (this.animTimer > 150) { // 150ms per frame
                this.animTimer = 0;
                this.animFrame = (this.animFrame + 1) % 2;
            }
        } else {
            this.animFrame = 0;
        }
        
        // ... rest of existing update
    }
    
    draw(ctx) {
        let sprite = 'nurse';
        if (this.state === 'sitting') sprite = 'nurse_sit';
        else if (this.state === 'moving' || this.state === 'walking') {
            sprite = this.animFrame === 0 ? 'nurse_walk1' : 'nurse_walk2';
        }
        
        ctx.save();
        ctx.translate(this.pos.x, this.pos.y);
        ctx.scale(this.facing, 1); // Flip based on direction
        drawSprite(ctx, sprite, 0, 0);
        ctx.restore();
        
        // ... rest of existing draw (coffee icon, shield, etc.)
    }
}
```

3. **Add baby wiggle animation to Isolette**:
```javascript
class Isolette extends Entity {
    constructor(x, y, id) {
        // ... existing
        this.wiggleOffset = 0;
        this.wiggleTimer = 0;
    }
    
    update(dt) {
        // ... existing
        
        // Wiggle when needs exist (baby crying)
        if (this.needs.length > 0) {
            this.wiggleTimer += dt;
            this.wiggleOffset = Math.sin(this.wiggleTimer / 50) * 3;
        } else {
            this.wiggleOffset = 0;
        }
    }
    
    drawBody(ctx) {
        const scaleFactor = PIXEL_SCALE / 6;
        
        // Apply wiggle to baby position
        ctx.save();
        ctx.translate(this.wiggleOffset, 0);
        
        // ... existing isolette drawing code
        // The baby sprite will now shake left/right
        
        ctx.restore();
    }
}
```

4. **Optional: Add frame timing utility**:
```javascript
class AnimationController {
    constructor(frameCount, frameDuration) {
        this.frameCount = frameCount;
        this.frameDuration = frameDuration;
        this.currentFrame = 0;
        this.timer = 0;
    }
    
    update(dt) {
        this.timer += dt;
        if (this.timer >= this.frameDuration) {
            this.timer = 0;
            this.currentFrame = (this.currentFrame + 1) % this.frameCount;
        }
        return this.currentFrame;
    }
    
    reset() {
        this.currentFrame = 0;
        this.timer = 0;
    }
}
```

---

## Feature 8: Parent Bonding Animation

### Goal
Visual feedback when parent holds baby - hearts, glow effect.

### Implementation Steps

1. **Add bonding state to Isolette**:
```javascript
class Isolette extends Entity {
    constructor(x, y, id) {
        // ... existing
        this.bondingActive = false;
        this.bondingTimer = 0;
        this.bondingParticles = [];
    }
}
```

2. **Modify hold need handling**:
```javascript
// In the isolette need handling section:
else if (need === 'hold') {
    iso.bondingActive = true;
    iso.bondingTimer = 3000; // 3 second animation
    
    playSoundVisual(iso.pos.x, iso.pos.y, "Bonding...");
    gameState.score += 75;
    addLogEntry('nurse', "Parent holding baby. ❤️");
    nurse.freezeTimer = 2000;
    playBeep(400, 300);
}
```

3. **Update bonding animation**:
```javascript
// In Isolette.update():
if (this.bondingActive) {
    this.bondingTimer -= dt;
    
    // Spawn heart particles
    if (Math.random() < 0.1) {
        this.bondingParticles.push({
            x: this.pos.x + (Math.random() - 0.5) * 40,
            y: this.pos.y - 20,
            vy: -1 - Math.random(),
            life: 60,
            size: 10 + Math.random() * 10
        });
    }
    
    // Update particles
    this.bondingParticles = this.bondingParticles.filter(p => {
        p.y += p.vy;
        p.life--;
        return p.life > 0;
    });
    
    if (this.bondingTimer <= 0) {
        this.bondingActive = false;
        this.bondingParticles = [];
        
        // BONUS: Reduce future need frequency for this baby
        // (ties into baby personality system)
    }
}
```

4. **Draw bonding effects**:
```javascript
// In Isolette.drawBody(), after main isolette:
if (this.bondingActive) {
    // Warm glow around isolette
    ctx.save();
    const glowIntensity = 0.3 + Math.sin(Date.now() / 200) * 0.1;
    ctx.globalAlpha = glowIntensity;
    ctx.fillStyle = '#fbd38d'; // Warm yellow
    ctx.beginPath();
    ctx.arc(0, 0, 60, 0, Math.PI * 2);
    ctx.fill();
    ctx.restore();
    
    // Parent silhouette (simplified)
    ctx.fillStyle = '#ed64a6';
    ctx.beginPath();
    ctx.arc(-25, 0, 15, 0, Math.PI * 2); // Head
    ctx.fill();
    ctx.fillRect(-35, 15, 20, 30); // Body
}

// Draw heart particles
this.bondingParticles.forEach(p => {
    ctx.save();
    ctx.globalAlpha = p.life / 60;
    ctx.fillStyle = '#fc8181';
    ctx.font = `${p.size}px Arial`;
    ctx.fillText('❤️', p.x - this.pos.x, p.y - this.pos.y);
    ctx.restore();
});
```

---

## Feature 9: Improved Floor Cleaning Machine

### Goal
Make the floor cleaner more visually distinct, add water/soap trail, better noise visualization.

### Implementation Steps

1. **Improve floor cleaner sprite**:
```javascript
floorcleaner: [
    "__LLLLLL__",  // Grey machine top
    "_LLLLLLLL_",
    "_LL_CC_LL_",  // Control panel (cyan)
    "_LLLLLLLL_",
    "LLLLLLLLLL",  // Wide body
    "LLLLLLLLLL",
    "_KKKKKKKK_",  // Black brushes
    "__K_KK_K__",  // Wheels
],
floorcleaner_operator: [
    "___HHH____",
    "___SSS____",
    "__YYYYY___",  // Yellow vest
    "__YYYYY___",
    "___YYY____",
    "___LL_____",
    "___LL_____"
]
```

2. **Add trail effect**:
```javascript
class Enemy extends Entity {
    constructor(x, y, type, targetIsolette) {
        // ... existing
        if (type === 'floorcleaner') {
            this.trail = [];
            this.trailTimer = 0;
        }
    }
    
    update(dt) {
        // Floor cleaner trail
        if (this.type === 'floorcleaner') {
            this.trailTimer += dt;
            if (this.trailTimer > 100) {
                this.trailTimer = 0;
                this.trail.push({
                    x: this.pos.x,
                    y: this.pos.y,
                    life: 120
                });
            }
            
            // Fade trail
            this.trail = this.trail.filter(t => {
                t.life--;
                return t.life > 0;
            });
            
            // Limit trail length
            if (this.trail.length > 50) {
                this.trail.shift();
            }
        }
        
        // ... existing update
    }
}
```

3. **Enhanced floor cleaner drawing**:
```javascript
// In Enemy.draw(), special case for floorcleaner:
if (this.type === 'floorcleaner') {
    // Draw wet floor trail
    this.trail.forEach(t => {
        ctx.save();
        ctx.globalAlpha = t.life / 120 * 0.3;
        ctx.fillStyle = '#90cdf4'; // Light blue
        ctx.beginPath();
        ctx.ellipse(t.x, t.y, 30, 15, 0, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();
    });
    
    // Draw machine
    drawSprite(ctx, 'floorcleaner', this.pos.x, this.pos.y);
    
    // Draw operator behind machine
    drawSprite(ctx, 'floorcleaner_operator', this.pos.x, this.pos.y - 50, PIXEL_SCALE * 0.8);
    
    // Animated noise rings
    const ringCount = 3;
    for (let i = 0; i < ringCount; i++) {
        const offset = (Date.now() / 500 + i / ringCount) % 1;
        ctx.save();
        ctx.globalAlpha = (1 - offset) * 0.3;
        ctx.strokeStyle = '#a0aec0';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.arc(this.pos.x, this.pos.y, 50 + offset * 100, 0, Math.PI * 2);
        ctx.stroke();
        ctx.restore();
    }
    
    // "LOUD" text that pulses
    if (Math.sin(Date.now() / 100) > 0) {
        ctx.fillStyle = '#f56565';
        ctx.font = 'bold 16px "Vt323"';
        ctx.textAlign = 'center';
        ctx.fillText('LOUD!', this.pos.x, this.pos.y - 80);
    }
    
    return; // Skip normal enemy drawing
}
```

4. **Add "Wet Floor" warning sign** that appears after cleaner passes:
```javascript
// In floor cleaner update, when leaving screen:
if (this.pos.x > canvas.width + 50) {
    // Spawn wet floor sign item
    entities.push(new WetFloorSign(canvas.width / 2, this.pos.y));
    this.markedForDeletion = true;
}

// New entity class
class WetFloorSign extends Entity {
    constructor(x, y) {
        super(x, y, 'wetfloorsign');
        this.timer = 10000; // Lasts 10 seconds
    }
    
    update(dt) {
        this.timer -= dt;
        if (this.timer <= 0) this.markedForDeletion = true;
        
        // Slip hazard if nurse runs through
        if (nurse.pos.dist(this.pos) < 40 && gameState.coffeeBoost > 0) {
            // Slip!
            playSoundVisual(nurse.pos.x, nurse.pos.y, "SLIP!");
            nurse.freezeTimer = 500;
            gameState.sanity -= 5;
            addLogEntry('nurse', "Slipped on wet floor!");
            this.markedForDeletion = true;
        }
    }
    
    draw(ctx) {
        ctx.save();
        ctx.translate(this.pos.x, this.pos.y);
        ctx.fillStyle = '#ecc94b';
        ctx.beginPath();
        ctx.moveTo(0, -30);
        ctx.lineTo(20, 10);
        ctx.lineTo(-20, 10);
        ctx.closePath();
        ctx.fill();
        ctx.fillStyle = 'black';
        ctx.font = '12px Arial';
        ctx.textAlign = 'center';
        ctx.fillText('!', 0, 5);
        ctx.restore();
    }
}
```

---

## Feature 10: Charge Nurse Power-Up

### Goal
Rare spawn that follows player for 30s and auto-handles one enemy type.

### Implementation Steps

1. **Add charge nurse sprite**:
```javascript
charge_nurse: [
    "___RRRR___",  // Red cap (charge color)
    "___RRRR___",
    "___SSSS___",
    "___SRSS___",
    "__RRRRRR__",  // Red scrubs
    "__RRRRRR__",
    "_RRWWRRRR_",
    "_RRRRRRRR_",
    "__RR__RR__",
    "__NW__NW__"
]
```

2. **Create ChargeNurse class**:
```javascript
class ChargeNurse extends Entity {
    constructor(x, y) {
        super(x, y, 'charge_nurse');
        this.speed = 3.5 * (PIXEL_SCALE / 4);
        this.timer = 30000; // 30 seconds
        this.cooldown = 0;
        this.targetEnemy = null;
        this.handledTypes = new Set();
    }
    
    update(dt) {
        this.timer -= dt;
        this.cooldown -= dt;
        
        // Follow nurse with offset
        const followDist = 80;
        const dx = nurse.pos.x - followDist - this.pos.x;
        const dy = nurse.pos.y - this.pos.y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        
        if (dist > 20) {
            this.pos.x += (dx / dist) * this.speed;
            this.pos.y += (dy / dist) * this.speed;
        }
        
        // Look for enemies to handle
        if (this.cooldown <= 0) {
            const enemies = entities.filter(e => 
                e instanceof Enemy && 
                e.active && 
                e.state === 'annoying' &&
                e.type !== 'floorcleaner'
            );
            
            if (enemies.length > 0) {
                // Find closest
                let closest = null;
                let closestDist = Infinity;
                enemies.forEach(e => {
                    const d = this.pos.dist(e.pos);
                    if (d < closestDist && d < 200) {
                        closest = e;
                        closestDist = d;
                    }
                });
                
                if (closest) {
                    this.targetEnemy = closest;
                    this.handleEnemy();
                }
            }
        }
        
        // Despawn when timer expires
        if (this.timer <= 0) {
            this.say("Page me if you need anything!", 'nurse');
            addLogEntry('nurse', "Charge: I'll be at the desk.");
            this.markedForDeletion = true;
        }
    }
    
    handleEnemy() {
        if (!this.targetEnemy) return;
        
        const quotes = {
            'resident': "I'll handle the resident. You focus on your babies.",
            'surgeon': "Let me talk to the surgeon.",
            'mom': "I've got this family.",
            'dad': "I'll answer their questions.",
            'medicalfamily': "I'll educate them on policy.",
            'sickvisitor': "Let me escort them out."
        };
        
        const quote = quotes[this.targetEnemy.type] || "I've got this.";
        this.say(quote, 'nurse');
        addLogEntry('nurse', `Charge: ${quote}`);
        
        this.targetEnemy.leave();
        gameState.score += 100;
        playSoundVisual(this.targetEnemy.pos.x, this.targetEnemy.pos.y, "HANDLED!");
        
        this.targetEnemy = null;
        this.cooldown = 5000; // 5 second cooldown between auto-handles
    }
    
    draw(ctx) {
        drawSprite(ctx, 'charge_nurse', this.pos.x, this.pos.y);
        
        // Timer indicator above head
        const pct = this.timer / 30000;
        ctx.fillStyle = '#f56565';
        ctx.fillRect(this.pos.x - 20, this.pos.y - 70, 40, 5);
        ctx.fillStyle = '#48bb78';
        ctx.fillRect(this.pos.x - 20, this.pos.y - 70, 40 * pct, 5);
        
        // "CHARGE" label
        ctx.fillStyle = '#ecc94b';
        ctx.font = '12px "Vt323"';
        ctx.textAlign = 'center';
        ctx.fillText('CHARGE', this.pos.x, this.pos.y - 75);
        
        if (this.bubble) {
            // Draw speech bubble
            ctx.fillStyle = 'white';
            ctx.strokeStyle = 'black';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.roundRect(this.pos.x - 60, this.pos.y - 120, 120, 40, 5);
            ctx.fill();
            ctx.stroke();
            ctx.fillStyle = 'black';
            ctx.font = '14px "Vt323"';
            ctx.fillText("...", this.pos.x, this.pos.y - 95);
        }
    }
}
```

3. **Add rare spawn logic**:
```javascript
// In spawnItem() or create separate function:
function spawnChargeNurse() {
    // Only spawn if none exists and rare chance
    const existingCharge = entities.find(e => e instanceof ChargeNurse);
    if (existingCharge) return;
    
    if (Math.random() < 0.0005) { // Very rare
        const cn = new ChargeNurse(-50, canvas.height / 2);
        entities.push(cn);
        playSoundVisual(canvas.width / 2, 100, "CHARGE NURSE ARRIVED!");
        addLogEntry('admin', "Charge nurse checking in!");
        playBeep(1000, 200);
    }
}

// Call in update():
spawnChargeNurse();
```

4. **Optional: Add item to summon charge nurse**:
```javascript
// New item type
else if (item.type === 'chargebell') {
    playSoundVisual(item.pos.x, item.pos.y, "DING!");
    const cn = new ChargeNurse(item.pos.x, item.pos.y);
    entities.push(cn);
    addLogEntry('nurse', "Called the charge nurse!");
}
```

---

## Feature 11: Baby Personalities

### Goal
Each baby has a name and temperament affecting gameplay.

### Implementation Steps

1. **Define personality types and names**:
```javascript
const babyPersonalities = {
    fussy: {
        names: ['Fiona', 'Felix', 'Frankie', 'Finnegan'],
        needMultiplier: 1.5,      // Needs appear faster
        calmDifficulty: 1.3,      // Harder to calm
        description: 'High maintenance',
        color: '#fc8181'          // Red-ish
    },
    hungry: {
        names: ['Hugo', 'Hazel', 'Henry', 'Harper'],
        needMultiplier: 1.2,
        feedFrequency: 2.0,       // Needs feeding more often
        description: 'Always hungry',
        color: '#fbd38d'          // Orange
    },
    sleepy: {
        names: ['Stella', 'Simon', 'Sage', 'Sloane'],
        needMultiplier: 0.7,      // Fewer needs
        wakeChance: 0.8,          // Wakes up easier from noise
        description: 'Light sleeper',
        color: '#90cdf4'          // Blue
    },
    chill: {
        names: ['Charlie', 'Cleo', 'Camden', 'Casey'],
        needMultiplier: 0.5,      // Rare needs
        bonusPoints: 1.2,         // More points when helped
        description: 'Easy baby',
        color: '#9ae6b4'          // Green
    },
    dramatic: {
        names: ['Daphne', 'Diego', 'Delilah', 'Dexter'],
        needMultiplier: 1.0,
        alarmVolume: 1.5,         // Louder when crying
        dramaParticles: true,     // Extra visual effects
        description: 'Drama queen',
        color: '#d6bcfa'          // Purple
    },
    fighter: {
        names: ['Faith', 'Finn', 'Freya', 'Fletcher'],
        needMultiplier: 1.3,
        recoveryBonus: true,      // Occasionally self-resolves
        description: 'Little fighter',
        color: '#f687b3'          // Pink
    }
};
```

2. **Modify Isolette class**:
```javascript
class Isolette extends Entity {
    constructor(x, y, id) {
        super(x, y, 'isolette');
        this.id = id;
        this.needs = [];
        
        // Assign personality
        const types = Object.keys(babyPersonalities);
        this.personalityType = types[Math.floor(Math.random() * types.length)];
        this.personality = babyPersonalities[this.personalityType];
        
        // Assign name
        const names = this.personality.names;
        this.babyName = names[Math.floor(Math.random() * names.length)];
        
        // Adjust need timer based on personality
        const baseTimer = 10000;
        this.needTimer = (baseTimer / this.personality.needMultiplier) * (0.8 + Math.random() * 0.4);
        this.baseNeedInterval = this.needTimer;
        
        this.size = 15 * (PIXEL_SCALE / 4);
        this.wiggleOffset = 0;
        this.wiggleTimer = 0;
        this.bondingActive = false;
        this.bondingTimer = 0;
        this.bondingParticles = [];
    }
    
    update(dt) {
        this.needTimer -= dt;
        
        if (this.needTimer <= 0 && this.needs.length < 3) {
            // Personality affects which needs appear
            let newNeed = this.getPersonalityNeed();
            
            if (!this.needs.includes(newNeed)) {
                this.needs.push(newNeed);
                
                // Dramatic babies make more noise
                if (this.personality.dramaParticles) {
                    for (let i = 0; i < 5; i++) {
                        particles.push({
                            x: this.pos.x + (Math.random() - 0.5) * 30,
                            y: this.pos.y - 30,
                            text: '!',
                            color: this.personality.color,
                            life: 40
                        });
                    }
                }
                
                playSoundVisual(this.pos.x, this.pos.y, `${this.babyName}!`);
                playBeep(600, 150);
            }
            
            // Reset timer with personality modifier
            this.needTimer = this.baseNeedInterval * (0.8 + Math.random() * 0.4);
        }
        
        // Fighter babies occasionally self-resolve
        if (this.personality.recoveryBonus && this.needs.length > 0) {
            if (Math.random() < 0.0001) {
                this.needs.shift();
                playSoundVisual(this.pos.x, this.pos.y, "Self-soothed!");
                addLogEntry('baby', `${this.babyName} calmed down on their own!`);
            }
        }
        
        // Sanity drain based on needs
        if (this.needs.length > 0) {
            let drain = 0.01 * this.personality.needMultiplier;
            if (gameState.shortStaffed) drain *= 1.5;
            if (gameState.nightShift) drain *= 1.2;
            gameState.sanity -= drain;
        }
        
        // Wiggle animation
        if (this.needs.length > 0) {
            this.wiggleTimer += dt;
            const wiggleSpeed = this.personality.dramaParticles ? 30 : 50;
            this.wiggleOffset = Math.sin(this.wiggleTimer / wiggleSpeed) * 3;
        } else {
            this.wiggleOffset = 0;
        }
        
        // Bonding update
        if (this.bondingActive) {
            // ... existing bonding code
        }
    }
    
    getPersonalityNeed() {
        const roll = Math.random();
        
        // Hungry babies need feeding more
        if (this.personalityType === 'hungry') {
            if (roll < 0.5) return 'feed';
        }
        
        // Fussy babies need calming more
        if (this.personalityType === 'fussy') {
            if (roll < 0.3) return 'calm';
        }
        
        // Default distribution
        if (roll < 0.3) return 'feed';
        if (roll < 0.5) return 'diaper';
        if (roll < 0.65) return 'assessment';
        if (roll < 0.8) return 'hold';
        if (roll < 0.9) return 'procedure';
        return 'photo';
    }
    
    drawBody(ctx) {
        const scaleFactor = PIXEL_SCALE / 6;
        
        // Apply wiggle
        ctx.save();
        ctx.translate(this.wiggleOffset, 0);
        
        // Isolette frame
        ctx.fillStyle = '#a0aec0';
        ctx.fillRect(-15 * scaleFactor, -8 * scaleFactor, 30 * scaleFactor, 25 * scaleFactor);
        ctx.fillStyle = '#718096';
        ctx.fillRect(-15 * scaleFactor, 17 * scaleFactor, 30 * scaleFactor, 4 * scaleFactor);
        ctx.fillStyle = '#edf2f7';
        ctx.beginPath();
        ctx.arc(0, -8 * scaleFactor, 18 * scaleFactor, Math.PI, 0);
        ctx.fill();
        
        // Baby sprite with personality tint
        const babyData = SPRITES.baby;
        const babyPixelScale = PIXEL_SCALE / 3;
        const startX = -(babyData[0].length * babyPixelScale) / 2;
        const startY = -12 * scaleFactor;
        
        for (let r = 0; r < babyData.length; r++) {
            for (let c = 0; c < babyData[0].length; c++) {
                if (COLORS[babyData[r][c]]) {
                    ctx.fillStyle = COLORS[babyData[r][c]];
                    ctx.fillRect(startX + c * babyPixelScale, startY + r * babyPixelScale, babyPixelScale, babyPixelScale);
                }
            }
        }
        
        // Glass dome
        ctx.fillStyle = 'rgba(190, 240, 255, 0.4)';
        ctx.beginPath();
        ctx.arc(0, -8 * scaleFactor, 18 * scaleFactor, Math.PI, 0);
        ctx.fill();
        ctx.strokeStyle = 'rgba(255,255,255,0.5)';
        ctx.lineWidth = 1 * scaleFactor;
        ctx.stroke();
        
        ctx.restore();
        
        // NAME TAG with personality color
        ctx.fillStyle = this.personality.color;
        ctx.fillRect(-25 * scaleFactor, 22 * scaleFactor, 50 * scaleFactor, 14 * scaleFactor);
        ctx.fillStyle = 'black';
        ctx.font = `${10 * scaleFactor}px "Vt323"`;
        ctx.textAlign = 'center';
        ctx.fillText(this.babyName, 0, 32 * scaleFactor);
        
        // Personality indicator (small icon)
        ctx.font = `${12 * scaleFactor}px Arial`;
        const icons = {
            fussy: '😤', hungry: '🍼', sleepy: '😴',
            chill: '😊', dramatic: '🎭', fighter: '💪'
        };
        ctx.fillText(icons[this.personalityType], 22 * scaleFactor, -20 * scaleFactor);
        
        // Need icons
        ctx.font = `${20 * scaleFactor}px Arial`;
        if (this.needs.includes('feed')) ctx.fillText('🍼', -10 * scaleFactor, -25 * scaleFactor);
        if (this.needs.includes('diaper')) ctx.fillText('💩', 10 * scaleFactor, -25 * scaleFactor);
        if (this.needs.includes('assessment')) ctx.fillText('📋', -20 * scaleFactor, -10 * scaleFactor);
        if (this.needs.includes('hold')) ctx.fillText('🤱', 20 * scaleFactor, -10 * scaleFactor);
        if (this.needs.includes('procedure')) ctx.fillText('💉', -10 * scaleFactor, 10 * scaleFactor);
        if (this.needs.includes('photo')) ctx.fillText('📸', 10 * scaleFactor, 10 * scaleFactor);
        if (this.needs.includes('calm')) ctx.fillText('😭', 0, -35 * scaleFactor);
        
        // Bonding effects
        // ... existing bonding draw code
    }
}
```

3. **Modify scoring** to account for personality:
```javascript
// When completing a need:
let points = basePoints;
if (iso.personality.bonusPoints) {
    points *= iso.personality.bonusPoints;
}
gameState.score += Math.floor(points);
```

4. **Add personality to log entries**:
```javascript
// Instead of "Baby fed", say "Hugo fed" 
addLogEntry('nurse', `${iso.babyName} fed.`);
```

5. **Add personality summary to shift stats**:
```javascript
// Track which babies were most/least demanding
shiftStats.mostDemandingBaby = null;
shiftStats.needsPerBaby = {};

// In isolette need completion:
shiftStats.needsPerBaby[iso.babyName] = (shiftStats.needsPerBaby[iso.babyName] || 0) + 1;
```

---

## General Implementation Notes

### Code Style
- Use `const` and `let`, not `var`
- Keep functions under 50 lines when possible
- Add comments for non-obvious logic
- Follow existing naming conventions (camelCase)

### Testing Checklist
For each feature:
1. Feature works in isolation
2. Feature works with other features enabled
3. No console errors
4. No performance degradation
5. Works on mobile (touch)
6. Works with both difficulty modes

### Performance Considerations
- Particle arrays should be capped (max 100-200)
- Trails should be limited in length
- Avoid creating objects in draw loops
- Use object pooling for frequently created/destroyed entities

### Save/Load
When modifying `settings` or adding persistent data:
```javascript
// Always save after changes
saveSettings();

// Always load on init
loadSettings();
```

### Debug Helpers
Add these for testing:
```javascript
// Console commands for testing
window.debugSpawnCharge = () => entities.push(new ChargeNurse(200, 200));
window.debugMaxSanity = () => gameState.sanity = 100;
window.debugShowTutorial = () => { tutorial.active = true; showTutorialHint('start'); };
window.debugListBabies = () => isolettes.forEach(i => console.log(i.babyName, i.personalityType));
```

---

## Priority Order

If implementing incrementally, suggested order:

1. **WASD Movement** - Foundation, quick win
2. **Baby Personalities** - Core gameplay enhancement  
3. **Tutorial Overlay** - Helps new players
4. **Night Shift Visuals** - Atmosphere
5. **Settings Menu** - QoL
6. **Sprite Animations** - Polish
7. **Shift Summary Stats** - Feedback loop
8. **Floor Cleaner Improvements** - Polish
9. **Parent Bonding Animation** - Polish
10. **Report-Off Sequence** - End-game polish
11. **Charge Nurse Power-Up** - Advanced feature

---

## File Output

All changes go into the single `index.html` file. The AI agent should:

1. Read the current file
2. Make targeted edits using search/replace or line insertions
3. Test in browser
4. Commit with descriptive message per feature

Example commit messages:
- `feat: add WASD keyboard movement controls`
- `feat: implement baby personalities with names and traits`
- `feat: add night shift visual effects with vignette`
- `fix: prevent tutorial hints from blocking gameplay`