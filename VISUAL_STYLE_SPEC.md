# Visual Style Specification

A comprehensive guide to recreating the retro-terminal aesthetic achieved in Ward-Watch NICU.

---

## 1. Art Direction Overview

**Theme:** Retro CRT terminal meets cozy pixel art
**Mood:** Warm, nostalgic, slightly clinical, playfully serious
**Era Reference:** Late 80s/early 90s hospital monitors with modern polish

### Core Aesthetic Pillars

1. **Scanline Reality** - CRT monitor effects ground the visuals in retro tech
2. **Neon Accents** - Strategic glow effects for interactivity and emphasis
3. **Pixel Warmth** - Despite clinical setting, warm skin tones and soft colors
4. **Depth Illusion** - Drop shadows and subtle 3D effects add weight to sprites

---

## 2. Color Palette

### Primary Sprite Colors

| Key | Hex Code | Usage |
|-----|----------|-------|
| S | `#f6d0b1` | Skin tone (faces, hands) |
| B | `#4299e1` | Blue (scrubs, eyes, UI accents) |
| G | `#48bb78` | Green (surgical wear, success states) |
| W | `#ffffff` | White (clothing, highlights) |
| H | `#2d3748` | Dark gray (hair, dark details) |
| R | `#f56565` | Red (emergencies, hazards, warnings) |
| P | `#ed64a6` | Pink (feminine accents, special states) |
| Y | `#ecc94b` | Yellow (gold accents, items) |
| O | `#ed8936` | Orange (warm accents, food items) |
| L | `#a0aec0` | Light gray (legs, neutral elements) |
| D | `#2c5282` | Dark blue (uniforms, depth) |
| K | `#000000` | Black (outlines, absolute dark) |
| C | `#0bc5ea` | Cyan (tech, medical equipment glow) |
| N | `#ccff00` | Neon yellow-green (highlights, alerts) |
| T | `#d69e2e` | Tan (skin variant, wood, warmth) |
| M | `#742a2a` | Maroon (hazards, dried blood) |
| J | `#63b3ed` | Johnny blue (visitor gowns, soft blue) |

### UI Color System

```css
/* Background Layers */
--bg-darkest:     #000000;   /* Pure black - monitor bg */
--bg-dark:        #111827;   /* CRT background */
--bg-medium:      #1e293b;   /* Canvas/floor base */
--bg-light:       #334155;   /* Alternate floor tiles */

/* Borders & Frames */
--border-dark:    #374151;   /* Monitor frame */
--border-medium:  #475569;   /* Panel borders */
--border-light:   #4b5563;   /* Inner borders */

/* Text */
--text-primary:   #e2e8f0;   /* Main text - off-white */
--text-muted:     #94a3b8;   /* Secondary text */
--text-dark:      #475569;   /* Timestamps, labels */

/* Accent Colors */
--accent-blue:    #60a5fa;   /* Primary UI, links */
--accent-green:   #4ade80;   /* Success, health, positive */
--accent-red:     #f87171;   /* Danger, critical, negative */
--accent-yellow:  #facc15;   /* Caution, warnings */
--accent-pink:    #f687b3;   /* Special, pause states */
--accent-cyan:    #22d3ee;   /* Tech, screens, equipment */

/* Button Specific */
--btn-bg:         #1e40af;   /* Button background */
--btn-border:     #60a5fa;   /* Button border */
--btn-text:       #bfdbfe;   /* Button text */
--btn-shadow:     #172554;   /* Button 3D shadow */
```

### Status Indicator Gradients

```css
/* Healthy (>70%) */
background: linear-gradient(90deg, #22c55e, #4ade80);
box-shadow: 0 0 10px rgba(74, 222, 128, 0.5);

/* Caution (30-70%) */
background: linear-gradient(90deg, #ca8a04, #facc15);
box-shadow: 0 0 10px rgba(250, 204, 21, 0.5);

/* Critical (<30%) */
background: linear-gradient(90deg, #991b1b, #ef4444);
box-shadow: 0 0 10px rgba(239, 68, 68, 0.5);
```

---

## 3. Sprite System

### Technical Specifications

```javascript
const PIXEL_SCALE = 12;  // Base multiplier - each "pixel" renders as 12x12 screen pixels
```

### Sprite Definition Format

Sprites are defined as multi-line strings where each character maps to a color:

```javascript
const SPRITES = {
    example: `
    ..WW..
    .WSSW.
    WSBSBW
    .WSSW.
    ..WW..
    `,
};
```

**Legend:**
- `.` = Transparent (skip pixel)
- Any letter = Color lookup from COLORS map

### Standard Sprite Sizes

| Category | Dimensions (chars) | Scaled Size (px @ 12) |
|----------|-------------------|----------------------|
| **Player Character** | 12 × 11 | 144 × 132 |
| **NPCs (standing)** | 8-10 × 9-10 | 96-120 × 108-120 |
| **NPCs (sitting)** | 10-12 × 10-12 | 120-144 × 120-144 |
| **Small Items** | 4-5 × 6-8 | 48-60 × 72-96 |
| **Medium Items** | 8-10 × 8-10 | 96-120 × 96-120 |
| **Equipment** | 8-9 × 8-10 | 96-108 × 96-120 |

### Scale Hierarchy

```javascript
// Player (largest, most detail)
playerScale = PIXEL_SCALE;           // 12

// NPCs and significant objects
npcScale = PIXEL_SCALE / 2;          // 6

// Equipment variants
equipmentSmall = PIXEL_SCALE / 2;    // 6
equipmentLarge = PIXEL_SCALE * 1.2;  // 14.4
```

### Character Design Guidelines

**Faces:**
- 2-3 pixels wide for eyes (color B for blue eyes default)
- Skin tone (S) fills face area
- Optional mouth: 1-2 pixels of darker tone or red

**Bodies:**
- Clear silhouette at small scale
- Uniform colors define role (white=nurse, green=surgeon, cyan=student)
- 1-pixel outline details for clothing edges

**Hair:**
- Dark gray (H) or black (K)
- 2-4 rows at top of head
- Simple shape, no intricate detail

---

## 4. Sprite Rendering

### Drop Shadow

Every sprite gets an elliptical shadow beneath:

```javascript
// Shadow ellipse at sprite base
ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
ctx.beginPath();
ctx.ellipse(
    x,                          // center X
    y + spriteHeight/2 + 4,     // below sprite
    spriteWidth / 3,            // horizontal radius
    spriteHeight / 8,           // vertical radius (flattened)
    0, 0, Math.PI * 2
);
ctx.fill();
```

### Pixel Shading (Volumetric Effect)

Add subtle depth to each pixel:

```javascript
// After drawing main pixel color:

// Top highlight (first 2 rows of sprite)
if (row < 2) {
    ctx.fillStyle = 'rgba(255, 255, 255, 0.1)';
    ctx.fillRect(pixelX, pixelY, scale, scale);
}

// Bottom shadow (last 2 rows of sprite)
if (row >= spriteHeight - 2) {
    ctx.fillStyle = 'rgba(0, 0, 0, 0.1)';
    ctx.fillRect(pixelX, pixelY, scale, scale);
}
```

### Glow Effects

Apply canvas shadow for emphasis:

```javascript
// Active/selected state
ctx.shadowBlur = 20;
ctx.shadowColor = '#4ade80';  // green glow

// Warning state
ctx.shadowBlur = 20;
ctx.shadowColor = '#ef4444';  // red glow

// Subtle tech glow
ctx.shadowBlur = 5;
ctx.shadowColor = '#22d3ee';  // cyan glow

// Remember to reset after drawing:
ctx.shadowBlur = 0;
```

### Depth Sorting

Sort entities by Y position before rendering:

```javascript
entities.sort((a, b) => a.y - b.y);
// Render in order: furthest (top of screen) first
```

---

## 5. Typography

### Font Stack

```css
/* Primary game font - retro pixel aesthetic */
@import url('https://fonts.googleapis.com/css2?family=VT323&display=swap');

font-family: 'VT323', monospace;
```

### Size Scale

| Context | Size | Weight |
|---------|------|--------|
| Giant titles (splash) | 60px | normal |
| Modal headers | 30px | normal |
| Game text/events | 20-24px | normal |
| UI labels | 16-18px | normal |
| Log entries | 0.9rem (~14px) | normal |
| Timestamps | 0.8rem (~13px) | normal |

### Text Styling

```css
/* Standard game text */
.game-text {
    font-family: 'VT323', monospace;
    color: var(--text-primary);
    text-transform: uppercase;
    letter-spacing: 1px;
}

/* Neon glow effect */
.neon-text-blue {
    text-shadow: 0 0 10px rgba(59, 130, 246, 0.8);
}

.neon-text-green {
    text-shadow: 0 0 10px rgba(74, 222, 128, 0.8);
}

.neon-text-red {
    text-shadow: 0 0 10px rgba(248, 113, 113, 0.8);
}
```

### Canvas Text

```javascript
// Set font before drawing
ctx.font = '24px "VT323"';
ctx.textAlign = 'center';
ctx.textBaseline = 'middle';
ctx.fillStyle = '#e2e8f0';

// With glow
ctx.shadowBlur = 5;
ctx.shadowColor = particle.color;
ctx.fillText(text, x, y);
ctx.shadowBlur = 0;
```

---

## 6. UI Components

### Panel Base

```css
.panel {
    background: rgba(17, 24, 39, 0.9);
    border: 2px solid #475569;
    border-radius: 2px;
    box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.5);
    position: relative;
}

/* Inner highlight border */
.panel::before {
    content: '';
    position: absolute;
    inset: 2px;
    border: 1px solid rgba(255, 255, 255, 0.05);
    border-radius: 1px;
    pointer-events: none;
}
```

### Buttons

```css
.btn {
    background: #1e40af;
    border: 2px solid #60a5fa;
    border-radius: 2px;
    padding: 12px 24px;

    font-family: 'VT323', monospace;
    font-size: 1.6rem;
    color: #bfdbfe;
    text-transform: uppercase;
    letter-spacing: 1px;

    /* 3D depth effect */
    box-shadow: 0 4px 0 #172554;

    cursor: pointer;
    transition: all 0.1s ease;
}

.btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 0 #172554;
}

.btn:active {
    transform: translateY(2px);
    box-shadow: 0 2px 0 #172554;
}
```

### Progress/Status Bars

```css
.status-bar-container {
    background: #1f2937;
    border: 2px solid #4b5563;
    border-radius: 2px;
    height: 24px;
    overflow: hidden;
    position: relative;
}

/* Scanline overlay on bar */
.status-bar-container::after {
    content: '';
    position: absolute;
    inset: 0;
    background: repeating-linear-gradient(
        0deg,
        transparent,
        transparent 2px,
        rgba(0, 0, 0, 0.1) 2px,
        rgba(0, 0, 0, 0.1) 4px
    );
    pointer-events: none;
}

.status-bar-fill {
    height: 100%;
    transition: width 0.3s cubic-bezier(0.4, 0, 0.2, 1),
                background 0.3s,
                box-shadow 0.3s;
}
```

### Modal Overlay

```css
.modal-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.8);
    display: flex;
    align-items: center;
    justify-content: center;
}

.modal-content {
    background: #111827;
    border: 2px solid #3b82f6;
    border-radius: 4px;
    max-width: 650px;
    padding: 24px;

    box-shadow:
        0 0 30px rgba(59, 130, 246, 0.3),
        inset 0 0 50px rgba(0, 0, 0, 0.8);
}

/* Inner border highlight */
.modal-content::before {
    content: '';
    position: absolute;
    inset: 4px;
    border: 1px solid rgba(59, 130, 246, 0.2);
    border-radius: 2px;
    pointer-events: none;
}
```

### Log/Message Panel

```css
.log-panel {
    position: fixed;
    right: 16px;
    top: 100px;
    bottom: 16px;
    width: 400px;

    background: rgba(17, 24, 39, 0.95);
    border: 2px solid #334155;
    border-radius: 2px;

    display: flex;
    flex-direction: column-reverse;  /* newest on top */
    overflow-y: auto;
    padding: 8px;
}

.log-entry {
    padding: 4px 8px;
    font-size: 0.9rem;
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.log-timestamp {
    font-family: monospace;
    color: #475569;
    margin-right: 8px;
}

.log-speaker {
    font-weight: bold;
    /* Color varies by character type */
}
```

---

## 7. Visual Effects

### CRT Scanlines

```css
.crt-overlay {
    position: fixed;
    inset: 0;
    pointer-events: none;
    z-index: 1000;

    background: repeating-linear-gradient(
        0deg,
        rgba(0, 0, 0, 0.1),
        rgba(0, 0, 0, 0.1) 1px,
        transparent 1px,
        transparent 2px
    );

    box-shadow: inset 0 0 80px rgba(0, 0, 0, 0.4);
}

/* Optional subtle flicker */
@keyframes flicker {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.98; }
}

.crt-overlay {
    animation: flicker 0.15s infinite;
}
```

### Vignette Effect

```javascript
// Daytime vignette
function drawVignette(ctx, width, height) {
    const gradient = ctx.createRadialGradient(
        width/2, height/2, height * 0.4,   // inner circle
        width/2, height/2, height * 0.9    // outer circle
    );
    gradient.addColorStop(0, 'transparent');
    gradient.addColorStop(1, 'rgba(0, 0, 0, 0.4)');

    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, width, height);
}

// Night mode vignette (darker)
function drawNightVignette(ctx, width, height) {
    // Full overlay
    ctx.fillStyle = 'rgba(0, 0, 20, 0.5)';
    ctx.fillRect(0, 0, width, height);

    // Stronger vignette
    const gradient = ctx.createRadialGradient(
        width/2, height/2, height * 0.4,
        width/2, height/2, height * 0.9
    );
    gradient.addColorStop(0, 'transparent');
    gradient.addColorStop(1, 'rgba(0, 0, 30, 0.7)');

    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, width, height);
}
```

### Floating Text Particles

```javascript
function drawParticle(ctx, particle) {
    ctx.font = 'bold 20px Arial';
    ctx.textAlign = 'center';
    ctx.fillStyle = particle.color;

    // Glow effect
    ctx.shadowBlur = 5;
    ctx.shadowColor = particle.color;

    ctx.fillText(particle.text, particle.x, particle.y);

    ctx.shadowBlur = 0;
}

// Particle moves upward and fades
particle.y -= 1;
particle.alpha -= 0.02;
```

### Floor Pattern

```javascript
function drawFloor(ctx, width, height, tileSize) {
    const cols = Math.ceil(width / tileSize);
    const rows = Math.ceil(height / tileSize);

    for (let row = 0; row < rows; row++) {
        for (let col = 0; col < cols; col++) {
            // Checkerboard pattern
            const isLight = (row + col) % 2 === 0;
            ctx.fillStyle = isLight ? '#1e293b' : '#334155';
            ctx.fillRect(col * tileSize, row * tileSize, tileSize, tileSize);
        }
    }
}
```

---

## 8. Stickers/Labels

For item labels, tutorial callouts, etc:

```javascript
function drawSticker(ctx, text, x, y, color) {
    ctx.font = '24px "VT323"';
    const padding = 8;
    const width = ctx.measureText(text).width + padding * 2;
    const height = 32;

    // Rounded rectangle background
    ctx.fillStyle = color;
    ctx.beginPath();
    ctx.roundRect(x - width/2, y - height/2, width, height, 5);
    ctx.fill();

    // White border
    ctx.strokeStyle = '#ffffff';
    ctx.lineWidth = 3;
    ctx.stroke();

    // Text
    ctx.fillStyle = '#000000';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(text, x, y);
}
```

---

## 9. Animation Timing

### Standard Transitions

```css
/* Quick feedback (buttons, hovers) */
transition: all 0.1s ease;

/* Smooth state changes (progress bars, colors) */
transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

/* Fade in/out */
transition: opacity 0.5s ease;
```

### Game Loop Timing

```javascript
// Target 60 FPS
const FRAME_TIME = 1000 / 60;  // ~16.67ms

// Use requestAnimationFrame for rendering
function gameLoop(timestamp) {
    update(timestamp);
    render();
    requestAnimationFrame(gameLoop);
}
```

---

## 10. Accessibility Considerations

1. **High Contrast** - Light text (#e2e8f0) on dark backgrounds (#111827)
2. **Color + Shape** - Don't rely on color alone; use shapes, icons, text
3. **Large Touch Targets** - Buttons minimum 44x44px
4. **Readable Font Sizes** - Minimum 14px for any text
5. **Status Indicators** - Combine color with position/animation for status bars

---

## 11. Implementation Checklist

### Canvas Setup
- [ ] Set canvas background to `#1e293b`
- [ ] Enable image smoothing: `ctx.imageSmoothingEnabled = false` for crisp pixels
- [ ] Implement floor tile pattern
- [ ] Add vignette overlay
- [ ] Add CRT scanline overlay (CSS or canvas)

### Sprite System
- [ ] Define COLORS lookup object
- [ ] Create SPRITES object with ASCII art
- [ ] Implement drawSprite() with shadow and shading
- [ ] Add glow effect support
- [ ] Implement depth sorting

### UI Layer
- [ ] Import VT323 font
- [ ] Create panel component style
- [ ] Create button component with 3D effect
- [ ] Create progress bar component
- [ ] Create modal overlay system
- [ ] Create log/message panel

### Polish
- [ ] Add particle system for floating text
- [ ] Implement night mode overlay
- [ ] Add CRT flicker animation
- [ ] Test all color states (healthy/caution/critical)
- [ ] Verify glow effects render correctly

---

## Quick Reference Card

```
COLORS: S(skin) B(blue) G(green) W(white) H(hair) R(red) P(pink) Y(yellow)
        O(orange) L(gray) D(darkblue) K(black) C(cyan) N(neon) T(tan) M(maroon)

PIXEL_SCALE: 12px per sprite pixel
FONT: VT323, monospace
BG: #1e293b (canvas), #111827 (UI panels)
ACCENT: #60a5fa (blue), #4ade80 (green), #f87171 (red)
BORDER: #475569 (panels), #3b82f6 (modals)

Drop shadow: rgba(0,0,0,0.5) ellipse
Highlight: rgba(255,255,255,0.1) top 2 rows
Shadow: rgba(0,0,0,0.1) bottom 2 rows
Glow: shadowBlur 15-20, color-matched
```

---

*This specification captures the visual language of Ward-Watch NICU as of February 2026. Use as a foundation and adapt for your specific game's needs.*
