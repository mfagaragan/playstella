# CLAUDE.md - PlayStella Codebase Guide

## Project Overview

**PlayStella** is a free, browser-based daily game where players interact with a virtual cat named Stella. The goal is to keep Stella happy as long as possible by making the right choices before she "swipes" at you. It's a Wordle-style daily challenge game with seeded randomness ensuring all players worldwide experience the same cat behavior each day.

### Key Characteristics
- **Type**: Single-page web application
- **Platform**: Browser-based (no installation required)
- **Gameplay**: One game per day with consistent behavior across all players
- **Data Storage**: Client-side only (localStorage)
- **Backend**: None - fully static site
- **Deployment**: Hosted on Netlify

## Repository Structure

```
playstella/
├── index.html       # Main game application (SPA with embedded React)
├── privacy.html     # Privacy policy page
├── terms.html       # Terms of service page
└── .git/           # Git repository
```

### File Descriptions

#### `index.html` (Main Game - 854 lines)
The entire game is contained in a single HTML file with:
- **Lines 1-74**: HTML head with meta tags, CDN imports, and CSS styles
- **Lines 76**: Root div for React mounting
- **Lines 78-851**: Embedded React application written in JSX (via Babel)
- **Lines 82-114**: Inline SVG icon components (Lucide-style icons)
- **Lines 116-847**: Main `StellaCatGame` React component
- **Lines 849-850**: React app initialization

#### `privacy.html` (225 lines)
Static legal document with:
- Comprehensive privacy policy with changelog
- Explains data collection practices (minimal)
- Details localStorage usage and third-party services

#### `terms.html` (350 lines)
Static legal document with:
- Terms of service with changelog
- Usage rules, warranties, liability limitations
- Arbitration agreement and governing law

## Technology Stack

### Core Technologies
- **React 18**: UI framework (loaded via unpkg CDN)
- **React DOM 18**: DOM rendering (loaded via unpkg CDN)
- **Babel Standalone**: In-browser JSX transformation (loaded via unpkg CDN)
- **Tailwind CSS**: Utility-first CSS framework (loaded via CDN)

### Browser APIs Used
- **localStorage**: Persistent game state storage
- **Web Share API**: Native sharing on mobile devices
- **Clipboard API**: Copy-to-clipboard fallback for desktop
- **Date API**: Daily seed generation and streak tracking

### Hosting & Infrastructure
- **Netlify**: Static site hosting with server-side analytics
- **CDNs**: unpkg.com, cdn.tailwindcss.com

## Game Architecture

### State Management

The game uses React hooks for state management with the following key state variables:

```javascript
// Game flow
gameState: 'start' | 'playing' | 'gameOver'

// Gameplay state
timer: number                    // Current game time in seconds
catMood: number                  // 0-100 (not displayed but tracked)
patience: number                 // 0-100 (determines cat expression)
timeLeft: number                 // 0-3 seconds until timeout
catAnimation: 'idle' | 'react' | 'swipe'

// Action tracking
actionCounts: { pet, leave, treat, end }
actionHistory: Array<{ action: string, outcome: string }>

// Daily/persistent state
dailySeed: number                // Seeded from current date
hasPlayedToday: boolean
bestTime: number | null
streak: number
lastGameTime: number
lastGameActions: number
lastGameHistory: Array

// UI state
ringColor: string                // Progress ring color
showCopied: boolean             // Share button feedback
```

### localStorage Schema

```javascript
// Keys used for persistent storage
{
  stellaLastPlayDate: "Tue Nov 19 2025",        // Date string
  stellaBestTime: "15.7",                       // Float as string
  stellaStreak: "5",                            // Integer as string
  stellaLastGameTime: "12.3",                   // Float as string
  stellaLastGameActions: "8",                   // Integer as string
  stellaLastGameHistory: '[{...}]'              // JSON array
}
```

### Seeded Randomness System

**Critical Feature**: All players worldwide experience the same cat behavior on the same day.

```javascript
// Daily seed generation (lines 138-141)
const today = new Date().toDateString();
const seed = today.split('').reduce((acc, char) => acc + char.charCodeAt(0), 0);

// Seeded random number generator (lines 202-207)
const getSeededRandom = (actionName, count) => {
  const actionSeed = actionName.split('').reduce((acc, char) => acc + char.charCodeAt(0), 0);
  const combinedSeed = dailySeed + actionSeed + count;
  const x = Math.sin(combinedSeed) * 10000;
  return x - Math.floor(x);
};
```

**How it works**:
1. Date string generates a daily seed (same for everyone globally)
2. Each action (pet, leave, treat) + occurrence count generates unique random value
3. Same action sequence always produces same outcomes on same day
4. Different players making same choices get same results

### Outcome Tier System

Actions have 7 possible outcomes with specific probabilities (lines 350-390):

| Outcome | Probability | Patience Change | Mood Change |
|---------|-------------|-----------------|-------------|
| Terrible | 10% | -40 | -25 |
| Bad | 15% | -25 | -15 |
| Okay | 20% | -12 | -8 |
| Neutral | 20% | -5 | 0 |
| Good | 17% | +10 | +8 |
| Great | 13% | +20 | +15 |
| Excellent | 5% | +35 | +25 |

### Timer & Inactivity System

- Players have 3 seconds to make each action
- Timer ring changes color: green (3-2s) → amber (2-1s) → red (1-0s)
- Circular progress ring updates every 20ms for smooth animation (line 234)
- Game ends when timer reaches 0 or patience drops to 0

### Cat Expression System

Cat emoji changes based on patience level (lines 402-411):

```javascript
if (patience >= 70) return '😺';  // Happy (70-100)
if (patience >= 40) return '😼';  // Neutral (40-69)
if (patience >= 20) return '😾';  // Annoyed (20-39)
return '🙀';                       // Scared/angry (0-19)
// Game over: '😾'
```

### Share Feature

The share system generates a Wordle-style grid (lines 482-554):

```javascript
// Action outcomes mapped to colored squares
'excellent'/'great' → 🟩 (green)
'good'/'neutral'    → 🟨 (yellow)
'okay'              → 🟧 (orange)
'bad'/'terrible'    → 🟥 (red)
Final square        → ⬛ (black - game ended)
```

**Share text format**:
```
PlayStella 😺
Nov 19, 2025

12.3s ⏱️ | 8 actions
5 day streak 🔥

🟩🟨🟥🟧🟨🟩🟨🟥⬛

playstella.com
```

## Development Workflows

### Making Changes to the Game

1. **Edit `index.html`** - All game logic is in this single file
2. **Test locally** - Open file directly in browser or use local server
3. **Verify game mechanics**:
   - Daily seed consistency (check same date produces same behavior)
   - localStorage persistence (check data survives page refresh)
   - Timer accuracy (3-second countdown)
   - Share feature (check grid generation)
4. **Test responsive design** - Mobile and desktop viewports
5. **Commit and push** to deploy (Netlify auto-deploys)

### Testing Checklist

When making changes, verify:

- [ ] Game starts correctly
- [ ] Timer counts accurately
- [ ] Actions produce consistent outcomes (same seed = same results)
- [ ] Patience/mood tracking works
- [ ] Cat expressions update appropriately
- [ ] Timer ring animates smoothly
- [ ] Game ends on timeout or patience = 0
- [ ] Daily play limit works (can't play twice same day)
- [ ] Streak tracking increments correctly
- [ ] Best time saves and displays
- [ ] Share feature generates correct grid
- [ ] Responsive design works on mobile
- [ ] No console errors
- [ ] localStorage data persists

### Modifying Legal Pages

`privacy.html` and `terms.html` follow a consistent structure:

1. **Update changelog** section at top (lines 34-53)
2. **Update "Last Updated" date** (line 31)
3. **Modify relevant sections** while maintaining structure
4. **Keep footer consistent** with other pages

## Code Conventions

### JavaScript/React Patterns

1. **Functional components with hooks** - No class components
2. **Inline event handlers** - Direct function calls in JSX
3. **Tailwind utility classes** - Minimal custom CSS
4. **No PropTypes or TypeScript** - Plain JavaScript
5. **CDN dependencies** - No npm/build process

### CSS Organization

1. **Tailwind utilities** for most styling
2. **Custom CSS in `<style>` tag** for:
   - Fluid responsive scaling (clamp functions)
   - Progress ring animations
   - Touch device hover fixes
3. **CSS class naming**: kebab-case (e.g., `timer-display`, `cat-container`)

### Component Structure

The app follows a single-component architecture with conditional rendering:

```jsx
<StellaCatGame>
  {gameState === 'start' && <StartScreen />}
  {gameState === 'playing' && <GameScreen />}
  {gameState === 'gameOver' && <GameOverScreen />}
  {gameState !== 'playing' && <Footer />}
</StellaCatGame>
```

### State Updates

**Pattern**: Most state updates are relative, not absolute

```javascript
// Good: Relative updates
setPatience(prev => Math.max(0, prev - 40));
setCatMood(prev => Math.min(100, prev + 15));

// Avoid: Absolute updates (can cause race conditions)
setPatience(patience - 40);  // Don't do this
```

## Important Implementation Details

### Streak Calculation Critical Section

**CRITICAL BUG RISK**: Streak calculation must read old lastPlayDate BEFORE updating it (lines 292-316)

```javascript
// ❌ WRONG - Will always reset streak
localStorage.setItem('stellaLastPlayDate', today);
const lastPlayDateStr = localStorage.getItem('stellaLastPlayDate'); // Always today!

// ✅ CORRECT - Read first, then update
const lastPlayDateStr = localStorage.getItem('stellaLastPlayDate');
// ... calculate streak based on OLD date ...
localStorage.setItem('stellaLastPlayDate', today);
```

### Daily Seed Consistency

The daily seed MUST be consistent across:
- All players worldwide
- All timezones (uses toDateString())
- Starting stats generation
- Action outcome determination
- Daily stats calculation (average, percentile)

**Do not** use:
- `Date.now()` (changes every millisecond)
- `toISOString()` (includes time)
- Random Math.random() (different per user)

### Timer Ring Animation

The progress ring uses stroke-dashoffset for smooth animation:

```javascript
strokeDasharray="283"  // Full circle circumference (2πr where r=45)
strokeDashoffset={283 - (timeLeft / 3) * 283}  // Remaining percentage
```

Updates every 20ms (line 234) for 50fps smoothness.

### Mobile Touch Fixes

Hover states are disabled on touch devices (lines 18-29):

```css
@media (hover: none) {
  .action-btn:hover {
    background-color: white !important;
  }
}
```

Buttons also call `.blur()` to prevent stuck hover states (lines 339-341).

## Deployment Notes

### Netlify Configuration

- Auto-deploys from main branch
- No build process required (static files)
- Server-side analytics enabled (no cookies)
- Custom domain: playstella.com

### Browser Compatibility

**Required features**:
- localStorage
- ES6+ JavaScript
- React 18 compatibility
- Flexbox/Grid CSS
- Web Share API (optional, has fallback)
- Clipboard API (optional, graceful degradation)

**Tested browsers**:
- Chrome/Edge (Chromium)
- Safari (iOS/macOS)
- Firefox

## AI Assistant Guidelines

### When Making Changes

1. **Understand seeded randomness** - Don't break daily consistency
2. **Preserve localStorage schema** - Changing keys breaks existing users' data
3. **Test streak logic carefully** - It's complex and easy to break
4. **Maintain responsive design** - Use clamp() for fluid scaling
5. **Keep single-file architecture** - Don't split into multiple files
6. **No build process** - Must work with CDN-loaded dependencies
7. **Test on mobile** - Touch interactions differ from desktop

### Common Modification Patterns

**Adding a new action button**:
1. Add icon component (lines 82-114)
2. Add to actionCounts state initialization (line 124)
3. Add button to grid (lines 710-746)
4. Outcome logic already handles any action name

**Changing game difficulty**:
1. Adjust outcome probabilities (lines 350-390)
2. Modify patience/mood change amounts
3. Change timer duration (currently 3 seconds)

**Adding stats/achievements**:
1. Add to localStorage schema
2. Initialize in useEffect (lines 138-199)
3. Update on game end (lines 285-333)
4. Display in game over screen

### Security Considerations

1. **No XSS risk** - No user-generated content or innerHTML
2. **No CSRF risk** - No forms or server communication
3. **localStorage only** - Data stays on device
4. **CDN integrity** - Consider adding SRI hashes for production
5. **No secrets** - Everything is client-side and visible

### Performance Considerations

1. **Minimal re-renders** - State is well-scoped
2. **Timer optimization** - Uses intervals, not recursive setTimeout
3. **No API calls** - Everything is local
4. **Lightweight** - ~37KB HTML file, CDN resources cached
5. **No images** - Uses emoji and SVG only

## Legal & Compliance

### Privacy Requirements

- COPPA compliant (13+ age requirement)
- No personal data collection
- Anonymous analytics only (Netlify server-side)
- localStorage disclosure in privacy policy

### Terms Requirements

- "As-is" warranty disclaimer
- Liability limitations
- Arbitration agreement (with 30-day opt-out)
- Intellectual property protection
- Anti-cheating provisions

### Changelog Maintenance

When updating legal docs:
1. Update "Last Updated" date
2. Add entry to changelog section
3. Summarize what changed
4. Keep old changelog entries for transparency

## Troubleshooting Common Issues

### Game won't start after playing once
- **Cause**: hasPlayedToday flag is set
- **Fix**: Clear localStorage or wait until next day (midnight local time)
- **Debug**: Check localStorage key 'stellaLastPlayDate'

### Streak not incrementing
- **Cause**: Streak calculation logic in endGame() (lines 291-316)
- **Debug**: Check if lastPlayDate was yesterday (not today, not 2+ days ago)

### Different behavior on same day
- **Cause**: Seed generation issue or non-seeded randomness
- **Fix**: Ensure using getSeededRandom(), not Math.random()
- **Debug**: Log dailySeed value, should be same for all players

### Timer ring not animating smoothly
- **Cause**: timeLeft not updating frequently enough
- **Fix**: Ensure interval runs at 20ms (line 234)
- **Debug**: Check browser dev tools performance

### Share feature not working
- **Cause**: Web Share API not supported or clipboard permissions
- **Fix**: Already has clipboard fallback (line 546)
- **Debug**: Check browser console for errors

## Future Enhancement Ideas

### Potential Features
- Sound effects (purr, meow, hiss)
- Multiple cat characters with different personalities
- Achievements system
- Leaderboard (would require backend)
- Customizable cat appearance
- More action types
- Difficulty modes
- Accessibility improvements (screen reader support)

### Technical Improvements
- Add SRI hashes for CDN resources
- Implement service worker for offline play
- Add unit tests (would require build process)
- Migrate to TypeScript (would require build process)
- Bundle dependencies locally instead of CDN
- Add analytics events (e.g., which actions are most popular)

## Version History

### Current Version (November 2025)
- Initial release with core gameplay
- Daily play limit with streak tracking
- Seeded randomness for consistency
- Share feature with Wordle-style grid
- Privacy policy and terms of service

## Contact & Support

- **Website**: playstella.com
- **Privacy inquiries**: privacy@playstella.com
- **Legal inquiries**: legal@playstella.com

---

**Last Updated**: November 19, 2025
**Document Version**: 1.0
**Codebase Version**: Initial Release
