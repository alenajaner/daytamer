# DayTamer — Claude Code Project Instructions

## Project Overview

**DayTamer** is a personal task and goal management PWA (Progressive Web App) with a focus on simplicity and cross-device sync. It's a single-file React application deployed to GitHub Pages, designed to help users structure their day with time-based task blocks and goal tracking.

**Key features:**
- Time-based task scheduling with customizable time slots
- Goal tracking with today/this-week/all-time views
- Multiple color themes (Sunrise, Teal, Ocean, Slate, Midnight, Forest, Rose)
- Dark/light mode
- Cross-device sync via Supabase (optional, sync-key based)
- PWA support (installable on mobile/desktop)
- localStorage for local persistence

## Architecture

### Single File, No Build Step
- **Source:** `daytamer-deploy/index.html`
- **React:** Loaded from CDN (react 18.2.0, react-dom 18.2.0)
- **Babel:** Inline transpilation via CDN babel-standalone 7.23.5
- **Fonts:** Google Fonts (Nunito, Nunito Sans)
- **Deployment:** GitHub Pages (`/daytamer-deploy/` directory)

### Data Persistence
- **Primary:** `localStorage` with keys: `dt_tasks`, `dt_goals`, `dt_categories`, `dt_deleted`, `dt_sorts`, `dt_theme`, `dt_dark`, `dt_synckey`
- **Sync:** Optional Supabase REST API (`https://cbiqdequtjojsaocktfk.supabase.co`) with user-provided sync keys
- **Sync logic:** Last-write-wins; manual pull, automatic push on changes

### State Management
- React hooks: `useState`, `useRef`, `useContext`, `useCallback`, `useMemo`, `useEffect`
- Theme context (`ThemeCtx`)
- No external state library (all state in main App component)

### Styling
- Inline `<style>` tag in `<head>` for critical CSS
- Inline styles via objects for component theming
- CSS variables not used (colors baked into theme objects)
- Tailwind-like approach: utility classes for positioning/spacing via inline styles

### Theme system
- A theme's `accent` may be a single hex string **or** a `{light, dark}` object; `useTheme()` resolves it to one `accent` for the current mode.
- `useTheme()` also returns `accentText` — a contrast-safe foreground (dark on light accents, white on dark ones). Use `accentText` for text/icons on an accent-filled background; **never hardcode `#fff` there**. A theme's light/dark palette may set its own `accentText` to override.
- Each theme's light/dark palette has these keys: `bg, surface, card, border, text, muted, subtle, navBg`.

### Established feature conventions
- The task completion toggle is a **rounded-square checkbox** (solid green fill + white check when done). Goal-milestone toggles stay **circles**.
- **Plan My Day** is a 5-step flow (tasks → top 3 → planning mode → preview → schedule). Breaks are **manual, not automatic**: `buildDaySchedule` places tasks back-to-back around locked tasks and scheduled breaks; the user adds breaks in the schedule step (push later, or trim from the task). No cadence/auto-break logic.
- localStorage key `dt_custombreaks` holds scheduled (fixed-time) breaks and is included in Supabase sync. `dt_breakcadence` is legacy/unused.

## Development Rules

### Minimal Changes Philosophy
- **Make the smallest change that solves the problem.** No refactoring for its own sake.
- No unnecessary abstractions—three similar lines is acceptable.
- Avoid adding features beyond what's requested.
- Don't add error handling for impossible scenarios (trust framework guarantees).

### Preserve Architecture
- Keep the single-file structure; do not split into separate files.
- Do not introduce a build step or bundler.
- React and dependencies must remain CDN-based.
- localStorage keys and Supabase table schema must not change without careful migration planning.

### Code Inspection
- Only read/modify the relevant section of `index.html`.
- Use grep to locate functions or state keys before editing.
- Check git history or search for related code when understanding context.
- Do not refactor surrounding code unless it directly affects the fix.

## Coding Conventions

### Component Structure
Components are defined as regular JavaScript functions using React hooks. Example:
```javascript
function TaskBlock({ task, onEdit, isDark }) {
  const theme = useTheme();
  return (
    <div style={{ /* inline styles */ }}>
      {/* JSX */}
    </div>
  );
}
```

### Hooks Usage
- Prefer `useState` for component-level state
- Use `useCallback` for event handlers that reference external state
- Use `useMemo` for expensive computations (e.g., sorting/filtering)
- Use `useEffect` sparingly; most data is persisted to localStorage synchronously

### Inline Styles
All styling is via inline objects; no CSS classes (except utility classes in `<style>`). Colors come from the `THEMES` object:
```javascript
const colors = isDark ? THEMES[themeId].dark : THEMES[themeId].light;
const styles = {
  container: { backgroundColor: colors.bg, color: colors.text, ... },
};
```

### Naming
- Components: PascalCase (`TaskBlock`, `SettingsPanel`, `DayView`)
- Functions: camelCase (`formatTime`, `syncToSupabase`)
- State variables: descriptive, snake_case for localStorage keys (`dt_tasks`, `dt_synckey`)
- Constants: UPPER_CASE (`THEMES`, `ICONS`, `SB_URL`)

### Persistence Pattern
When modifying state that should persist:
1. Update React state via `setState`
2. Immediately save to localStorage: `LS.set("dt_key", value)`
3. If syncing is enabled, `pushSync()` will pick up the change on next opportunity

Example:
```javascript
const [tasks, setTasks] = useState(/* ... */);
const addTask = (task) => {
  const updated = [...tasks, task];
  setTasks(updated);
  LS.set("dt_tasks", updated);
};
```

### Comments
- No comments unless the WHY is non-obvious (hidden constraint, workaround, subtle invariant)
- Never comment what the code does; the code should be clear
- One line max; no multi-line comment blocks

## Browser Compatibility
- Target: Modern browsers (Chrome, Safari, Firefox, Edge)
- PWA features: Service Workers via manifest.json
- Mobile: Viewport meta tags, touch-friendly tap targets (min 44px)

## When Making Changes
1. Read only the relevant section of `index.html` (use grep to find it)
2. Make the minimal change to fix the issue
3. Verify the change in a browser (use the preview tools to run locally or against GitHub Pages)
4. Do not commit style/lint cleanup unless it's part of the fix
5. Test on both light and dark modes, and verify sync still works if touching sync logic

## Committing & Publishing
- When a task is finished, prompt the user to commit using an **interactive button** (AskUserQuestion), not a plain-text question — so it stands out. Ask before committing; never commit automatically.
- Commit **and push** together: the site is served from GitHub Pages (origin: github.com/alenajaner/daytamer, branch `master`) and only updates on push. A local commit alone does not reach the live site. After committing, `git push origin master` so changes go live.
- Do not call a change "done"/"shipped" until it is pushed.
- Use a concise, descriptive commit message summarizing the change.
- Note: this `CLAUDE.md` lives at the repo root (`DayTamer/`) and is version-controlled.

## Testing Approach
- Manual testing in browser (light/dark mode, desktop/mobile viewport, with/without sync key)
- Check localStorage directly in DevTools to verify persistence
- For sync changes, test with a sync key on two browsers/devices if possible
- No automated tests; rely on manual verification before shipping

## Deployment
- Source: `/daytamer-deploy/` directory
- Deployed to: GitHub Pages (automatic on push)
- Manifest & icons: `/daytamer-deploy/manifest.json`, `/daytamer-deploy/icon-*.png`, etc.
- No build step; deployed as-is
