# MyAgent Design System

## Direction

Mood: a quiet engineering workspace in soft daylight — neutral surfaces, graphite text, and just enough cool color to mark the active thread.

The interface uses a restrained product strategy. Neutral surfaces carry the workspace; the chosen theme color appears only on active navigation, focus rings, contextual highlights, and the primary send action.

## Color

All implementation colors are CSS custom properties in OKLCH. The default accent is a restrained blue (`oklch(0.59 0.14 252)`) that supports a calm Codex-like work surface. Users can switch among moss, blue, violet, amber, and rose accents. Light, dark, and system appearance modes share the same semantic token names.

Semantic roles:

- `--canvas`: main conversation surface.
- `--sidebar`: navigation surface, one neutral step away from the canvas.
- `--surface`: controls and elevated working surfaces.
- `--text`: primary ink with at least 7:1 contrast against the canvas.
- `--muted`: secondary text with at least 4.5:1 contrast where used for prose.
- `--accent`: active and primary-action color only.
- `--accent-soft`: selection and context highlight background.
- `--border`: subtle structural separators.
- `--danger`, `--success`, `--warning`: state-only colors.

## Typography

Use the native system sans stack so the workbench feels at home on the user's desktop and loads without font shift. Code and paths use the native monospace stack. Base size is 14px, body line-height is 1.65, and conversation prose is capped near 72 characters.

## Layout

- Desktop: 264px navigation rail plus a flexible conversation canvas.
- Composer: centered at the bottom of the conversation, maximum width 880px.
- Conversation content: maximum width 820px with generous vertical rhythm.
- Tablet/mobile: sidebar becomes an overlay; secondary labels simplify before primary actions disappear.

## Components

- Buttons use an 8px radius; text inputs and compact controls use 8–10px.
- The composer is the only visually elevated element and uses a restrained shadow without a decorative border-shadow combination.
- Conversation content is continuous, not a stack of cards.
- Tool activity is represented as compact timeline rows with status icons and explicit labels.
- Theme controls expose both appearance and accent; the current choice is conveyed by shape and label, not color alone.
- Selecting conversation text reveals a compact dark action bar anchored above the selection. “添加到对话” attaches the quote to the main composer; “在侧边聊天中提问” opens a 384px contextual panel.
- The side chat uses a continuous reading surface rather than consumer-style message bubbles. On narrow screens it becomes a full-width overlay.
- In the main conversation, user messages are compact right-aligned accent capsules. Assistant responses remain a document-like flow: run disclosure, prose, change summary, then low-emphasis icon actions.
- Tool execution stays collapsed under the run-duration disclosure unless the user explicitly opens it. The answer, not the internal process, owns the page hierarchy.

## Motion

Use 160–220ms ease-out transitions for menus, selection, sidebar collapse, and theme changes. Motion communicates state only. Under `prefers-reduced-motion: reduce`, transitions become effectively instant.

## Voice

Concise, specific, and operational. Labels should say what will happen: “新建会话”, “打开项目”, “发送”. Avoid promotional copy inside the work surface.
