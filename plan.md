Coding Plan: NICU Edition - Kitchen Sink Expansion
Overview
Expand the NICU nurse survival game with more authentic healthcare chaos, new enemy types, items, and game mechanics that capture the reality of working in a neonatal intensive care unit.

Phase 1: New Enemy Types
High Priority Enemies
1. Floor Cleaner Guy

Sprite: Create sprite with headphones, pushing large cleaning machine
Behavior:

Spawns rarely (0.001 probability)
Moves in straight line across screen, ignoring everything
Creates "NOISE FIELD" that drains sanity within radius (100px)
Can't be dismissed by talking - he has headphones in
Sound effect: Low rumbling bass beep (200hz)
Quote rotation: "🎧...", "[RUMBLE]", "oblivious"
Sanity drain: 0.2/frame when in range (VERY HIGH)
Players must avoid or close doors (new mechanic?)



2. "Medical" Family Member
javascript// The relative who's a dentist/vet/chiropractor
type: 'medicalfamily'
quotes: [
  "I'm a dentist, so I understand respiratory distress",
  "As a veterinarian, I think...",
  "My degree is in podiatry but...",
  "I took anatomy in college",
  "WebMD says..."
]

Behavior: Like mom/dad but MORE annoying
Stays longer (maxAnnoyance: 150 instead of 100)
Sanity drain: 0.08/frame
Score for dismissing: +250

3. The Too-Early Attending
javascripttype: 'attending'
spawnTime: 6:45am - 7:15am only
quotes: [
  "Quick bedside rounds before shift change",
  "Let's examine every baby real quick",
  "This'll only take 20 minutes",
  "Pull up the labs from last night"
]

Visits ALL isolettes in sequence
Blocks nurse from caring for babies during visit
Cannot be dismissed - must wait them out
Sanity drain: 0.06/frame

4. The Sick Visitor
javascripttype: 'sickvisi tor'
quotes: [
  "*cough cough* It's just allergies",
  "I'm not sick, just a little cold",
  "The baby needs to meet grandma!",
  "*sneezes on isolette*"
]

Has visual cough/sneeze particle effect
Moderate sanity drain: 0.07/frame
Can be dismissed but requires nurse to be at FULL sanity

5. The Photographer
javascripttype: 'photographer'
quotes: [
  "*FLASH* 📸",
  "Just one more picture!",
  "This is for Facebook",
  "Turn off the lights for better photos"
]

Creates bright flash visual effects
Wakes sleeping babies automatically
Fast sanity drain: 0.1/frame


Phase 2: New Items & Mechanics
Positive Items
6. Hand Sanitizer Station
javascripttype: 'sanitizer'
emoji: '🧴'
effect: +15 sanity, creates shield for 3 seconds (enemies avoid you)
spawnRate: 0.003
7. Functioning Equipment
javascripttype: 'workingpump'
emoji: '💉'
effect: Fixes one random isolette's needs automatically, +10 sanity
spawnRate: 0.002
8. Colleague Support
javascripttype: 'coworker'
emoji: '👩‍⚕️'
effect: Handles one random enemy for you, +25 sanity, "I got this one"
spawnRate: 0.001
Negative Items (Traps)
9. Mandatory Training Email
javascripttype: 'trainingemail'
emoji: '📧'
effect: -20 sanity, "DUE TODAY", freezes player for 1 second
spawnRate: 0.004
10. Missing Supplies
javascripttype: 'missingsupply'
emoji: '❌'
effect: -15 sanity, next baby task takes 2x longer
spawnRate: 0.005
11. Broken Equipment
javascripttype: 'brokenequip'
emoji: '⚠️'
effect: -10 sanity, creates urgent alarm sound
spawnRate: 0.003

Phase 3: Enhanced Baby Mechanics
New Baby States
12. Simultaneous Alarms
javascript// Multiple babies alarm at once
if (alarmedBabies > 3) {
  sanityDrainMultiplier = 2.0;
  playBeep(800, 300); // Louder, longer
  playSoundVisual(center, "ALARM SYMPHONY!");
}
13. Baby Needs New Types

'assessment' - Attending ordered assessment (+150 points)
'hold' - Parent wants to hold baby (takes longer, +75 points)
'procedure' - Needs procedure, parent wants to watch (-sanity)
'photo' - Parent wants photo taken (+50 points, quick)


Phase 4: New Environmental Elements
14. Phone/Pager System
javascriptclass Pager {
  constructor() {
    this.messages = [
      "Pharmacy: Med clarification needed",
      "Lab: Crit ical value call",
      "Family on line 3",
      "Charge nurse needs you",
      "RT wants to discuss vent settings"
    ];
  }
  // Appears as notification, must be cleared, drains sanity until acknowledged
}
15. Supply Cart
javascriptclass SupplyCart extends Entity {
  // Fixed location
  // Must visit to "restock" before certain baby tasks
  // Adds realism but also another step
}
16. Charting Computer
javascriptclass ChartingStation extends Entity {
  // Must visit periodically to "document"
  // If not visited every X minutes, -sanity and penalty
  // Log message: "Documentation overdue!"
}

Phase 5: New Game Modes & Difficulty
17. Night Shift Mode
javascript// Darker canvas, different enemy spawn rates
// More tired (sanity drains faster)
// Coffee is MORE effective
// Fewer staff around to help
18. Short-Staffed Modifier
javascript// Player handles 8 isolettes instead of 6
// Enemies spawn more frequently
// Sanity drain increased 1.5x
// Higher score multiplier
19. "Good Day" Boost
javascript// Random positive events:
// - Parent brings cookies for staff (+20 sanity to all)
// - All babies sleeping at once (+50 sanity)
// - Attending cancels rounds (respite period)
// - Extra break nurse shows up (auto-handles one baby)

Phase 6: Polish & Feedback
20. Enhanced Log Messages
javascript// More variety in responses:
nurseResponses = {
  dismissEnemy: [
    "Read the chart!",
    "I'll get back to you",
    "Ask the charge nurse",
    "Not right now",
    "Let me finish this first"
  ],
  feedBaby: [
    "Baby fed",
    "+15g weight gain",
    "Tolerating feeds well",
    "No residuals!"
  ],
  changeDiaper: [
    "Diaper changed",
    "Output documented",
    "That was... substantial",
    "Code brown managed"
  ],
  calmBaby: [
    "Baby calmed",
    "Shhh...",
    "Back to sleep",
    "Crisis averted"
  ]
}
21. Visual Effects
javascript// Add more particle effects:
// - Steam from coffee
// - Sparkles from successful dismissal
// - Storm cloud over head when sanity low
// - Speed lines when coffee-boosted
22. Sound Design
javascript// More varied beeps:
// - Different pitch per isolette (harmony of chaos)
// - Distinct sounds for different actions
// - Ambient hospital sounds (muffled)

Phase 7: Sticker Expansion
23. New Sticker Phrases
javascriptadditionalStickers = [
  "Hand Hygiene Queen",
  "Alarm Silencer",
  "NPO Enforcer",
  "No Flash Photos",
  "I Need Backup",
  "Intake/Output",
  "15 Minute Chart",
  "Family Education",
  "Code Brown Survivor",
  "Tandem Nursing",
  "Skin-to-Skin",
  "Beep Beep Beep",
  "Where's RT?",
  "Call the Fellow",
  "Not My Patient",
  "I Forgot to Chart",
  "Pizza Party?",
  "Mandatory Training",
  "Float Pool",
  "Short Staffed"
]

Implementation Order
Sprint 1: Core Enemies (Quick Win)

Floor Cleaner Guy sprite & behavior
Medical Family Member enemy
Sick Visitor enemy
Enhanced log message variety

Sprint 2: Items & Balance

New positive items (sanitizer, coworker)
New negative items (training email, broken equipment)
Balancing pass on sanity drain rates
Sound effect additions

Sprint 3: Baby Mechanics

New baby need types
Simultaneous alarm system
Charting station requirement
Supply cart mechanic

Sprint 4: Advanced Features

Pager/phone system
Photographer enemy with flash effects
Too-Early Attending enemy
Enhanced visual effects

Sprint 5: Game Modes

Difficulty modifiers
Random positive events
Night shift mode
New sticker phrases


Technical Considerations
Performance

Limit max enemies on screen (8-10)
Particle system pooling for efficiency
Optimize sprite rendering for mobile

Mobile Optimization

Touch target sizes for dismissing enemies
Better touch handling for dragging stickers
Responsive canvas sizing maintained

Accessibility

Visual indicators for sound cues
Colorblind-friendly status bars
Pause menu improvements

Code Quality

Create enemy factory pattern
Centralize spawn rate configuration
Create dialogue/quote management system
Separate game balance constants into config object


Testing Checklist

 All new enemies spawn and behave correctly
 Sanity drain feels balanced
 Mobile touch works for all new elements
 No console errors
 Log messages display properly
 Sticker system works with new phrases
 Sound effects don't overlap cacophonously
 Game remains fun (playtest!)
 Performance on mobile devices
 Game can still be won/lost appropriately


Config File Structure
javascriptconst GAME_CONFIG = {
  enemies: {
    resident: {
      speed: 1.5,
      sanityDrain: 0.05,
      maxAnnoyance: 100,
      quotes: [...]
    },
    floorcleaner: {
      speed: 1.0,
      sanityDrain: 0.2,
      noiseRadius: 100,
      canDismiss: false,
      quotes: [...]
    },
    // ... etc
  },
  items: {
    coffee: { sanityBoost: 5, effect: 'speed', duration: 5000 },
    // ... etc
  },
  spawnRates: {
    enemy: 0.005,
    item: 0.004,
    floorcleaner: 0.001
  }
};