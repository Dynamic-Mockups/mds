# Design System Workflow: Figma → Claude Code → Storybook

## 1. Overview

Building design system components manually from Figma specs is slow, inconsistent, and error-prone. Developers eyeball spacing, misinterpret token values, and skip edge-case variants.

This workflow connects Claude Code directly to your Figma files and Storybook instance via MCP (Model Context Protocol). Claude Code reads exact design specs from Figma, checks your existing component library in Storybook, and generates production-ready components that follow your architecture — not generic boilerplate.

---

## 2. Tech Stack & Prerequisites

| Tool | Purpose | Install / Access |
|------|---------|-----------------|
| **Claude Code** | AI coding agent (CLI) | `curl -fsSL https://claude.ai/install.sh \| bash` |
| **Figma MCP** | Gives Claude Code read access to Figma files | Two options — see Section 3 |
| **Storybook MCP** | Gives Claude Code awareness of existing component library | `npx @anthropic/storybook-mcp` |
| **Storybook** | Component documentation & visual testing | Already running in your project on a port (e.g., `6006`) |
| **Node.js 18+** | Runtime for MCP servers | `nvm install 18` |

---

## 3. MCP Setup

### 3.1 What Each MCP Server Provides

| MCP Server | Claude Code Gets Access To |
|------------|---------------------------|
| **Figma MCP** | Component dimensions, spacing, padding, colors, typography, border-radius, variants, states (hover, disabled, active), auto-layout rules, design token references |
| **Storybook MCP** | Existing components, their props/interfaces, story structure, naming conventions, file organization patterns, token usage |

### 3.2 Figma MCP Setup

1. Download and install [Figma Desktop App](https://www.figma.com/downloads/)
2. Open your design file in Figma Desktop
3. Open the **Inspect panel** (right sidebar) → find **MCP server** section → click **Enable desktop MCP server**
4. You should see a confirmation that the server is running at `http://127.0.0.1:3845/mcp`

Register it in Claude Code:

```bash
claude mcp add figma-desktop --url http://127.0.0.1:3845/mcp
```

### 3.3 Configuration

Full config with both MCP servers in `.claude/settings.json`:

```json
{
  "mcpServers": {
    "figma-desktop": {
      "url": "http://127.0.0.1:3845/mcp"
    },
    "storybook": {
      "command": "npx",
      "args": [
        "-y",
        "@anthropic/storybook-mcp",
        "--port=6006"
      ]
    }
  }
}
```

### 3.4 Verification

1. Make sure Figma Desktop is open with MCP server enabled
2. Start Storybook: `npm run storybook`
3. Run `claude` in your project root
4. Both MCP servers connect automatically on startup
5. Verify with: `"List available Figma components"` and `"What components exist in our Storybook?"`

---

## 4. CLAUDE.md Configuration

This is the most critical piece. Without explicit rules, Claude Code will generate generic code that doesn't match your system. Add a `CLAUDE.md` file to your project root.

### 4.1 Required Rules

```markdown
# Design System Rules

## Component Architecture
- ALWAYS use Radix UI primitives as base — never use raw HTML elements
  (`<button>`, `<input>`, `<select>`, `<dialog>` are forbidden)
- Compose components from existing design system atoms:
  `Button`, `Input`, `Select`, `Dialog`, `Popover`, `Tooltip`, etc.
- If a base component doesn't exist, create it as a primitive first,
  then compose the feature component from it

## Design Tokens
- NEVER hardcode color values (`#fff`, `rgb()`, `hsl()`)
  — use token references: `var(--color-primary)`, `theme.colors.primary`
- NEVER hardcode spacing (`16px`, `1rem`)
  — use token scale: `var(--space-4)`, `theme.spacing[4]`
- NEVER hardcode font sizes or weights
  — use typography tokens: `var(--font-size-sm)`, `var(--font-weight-medium)`
- NEVER hardcode border-radius
  — use radius tokens: `var(--radius-md)`

## File Structure
Every component MUST include:
- `ComponentName.tsx` — component implementation
- `ComponentName.stories.tsx` — Storybook stories
- `ComponentName.types.ts` — TypeScript props interface (if complex)
- `index.ts` — barrel export

Directory: `src/components/ui/ComponentName/`

## TypeScript
- All props must have an explicit TypeScript interface
- Use `React.ComponentPropsWithoutRef<"element">` for HTML prop inheritance
- Export the props type alongside the component

## Storybook Stories
- Every variant gets its own story
- Every interactive state gets a story (hover, focus, disabled, loading, error)
- Include a "Playground" story with all controls exposed via `args`
- Include a "Documentation" story showing real usage context

## Styling
- Use Radix UI Themes for component styling (`@radix-ui/themes`)
- Use Radix Colors for color tokens — never use raw hex/rgb values
- Apply styles via Radix's built-in `variant`, `size`, `color` props — not custom CSS classes
- For custom layout and spacing, use Radix `<Box>`, `<Flex>`, `<Grid>` layout components
- Responsive behavior must be included if the Figma design shows it
- Support dark mode via Radix `appearance` prop, not separate styles
```

### 4.2 Optional but Recommended

```markdown
## Naming Conventions
- Components: PascalCase (`AlertDialog`, `DataTable`)
- Props interfaces: `ComponentNameProps`
- Stories: `ComponentName.stories.tsx`
- Tokens: kebab-case (`--color-brand-primary`)

## Accessibility
- All interactive components must have ARIA attributes
- Keyboard navigation must work (Enter, Escape, Tab, Arrow keys)
- Color contrast must meet WCAG AA minimum

## What NOT To Generate
- No inline `style={{}}` attributes — use Radix props and tokens
- No `any` types
- No unnamed/default exports for components
- No stories without at least 3 variants
- No raw `<div>` for layout — use Radix `<Box>`, `<Flex>`, `<Grid>`
```

> **Tip:** Run `claude /init` to auto-generate a starter `CLAUDE.md`, then merge the rules above into it.

---

## 5. Workflow in Action

### Step-by-step Process

```
┌──────────┐     MCP      ┌─────────────┐     generates     ┌───────────────────┐
│  Figma   │ ──────────▶  │ Claude Code │ ──────────────▶   │ Component + Story │
│  Design  │              │             │                   │ (.tsx + .stories) │
└──────────┘              │  reads both │                   └───────────────────┘
                          │  sources    │
┌──────────┐     MCP      │             │
│Storybook │ ──────────▶  │             │
│ Existing │              └─────────────┘
└──────────┘
```

### 5.1 Point Claude Code at a Figma Component

Copy the Figma component link (right-click → "Copy link to selection" in Figma) and use it in your prompt.

### 5.2 Example Prompt

```
Look at the Modal component in this Figma file:
https://www.figma.com/file/ABC123/Design-System?node-id=1234:5678

Check our existing Storybook for current component conventions and token usage.

Create the Modal molecule component following our CLAUDE.md rules:
- Compound component pattern (Modal, Modal.Header, Modal.Content, Modal.Footer)
- All variants visible in the Figma design
- All states (default, confirmation/destructive, scrollable content, custom width)
- Full Storybook coverage
- Proper TypeScript interfaces
```

### 5.3 What Claude Code Does

| Step | Action |
|------|--------|
| 1 | Reads Figma node via MCP — extracts padding, colors, typography, border-radius, variants, states |
| 2 | Reads existing Storybook via MCP — learns naming patterns, prop conventions, token system |
| 3 | Reads `CLAUDE.md` — applies your architecture rules and constraints |
| 4 | Generates `Modal.tsx` using Radix UI Dialog primitive + compound component pattern (Modal.Header, Modal.Content, Modal.Footer) |
| 5 | Generates `Modal.scss` with design token CSS custom properties |
| 6 | Generates `Modal.stories.tsx` with all variants, states, and a Playground story |
| 7 | Generates `index.ts` barrel export |

---

## 6. Example Output

### Modal.tsx (simplified)

```tsx
import React from "react";
import * as RadixDialog from "@radix-ui/react-dialog";
import type { DialogProps as RadixDialogProps } from "@radix-ui/react-dialog";
import { Box, Text } from "@radix-ui/themes";
import { type ColorToken, getColorTokenVar } from "../../../tokens";
import { Button } from "../../atoms";
import { Cross1Icon } from "@radix-ui/react-icons";
import "./Modal.scss";

export interface ModalProps extends Omit<RadixDialogProps, "children"> {
  trigger?: React.ReactNode;
  children?: React.ReactNode;
  showClose?: boolean;
  overlayColorToken?: ColorToken;
  backgroundColorToken?: ColorToken;
  className?: string;
  width?: string;
}

export interface ModalHeaderProps {
  title: string;
  description?: string;
  titleColorToken?: ColorToken;
  descriptionColorToken?: ColorToken;
  className?: string;
  children?: React.ReactNode;
}

export interface ModalContentProps {
  children: React.ReactNode;
  maxHeight?: string;
  className?: string;
}

export interface ModalFooterProps {
  children: React.ReactNode;
  className?: string;
}

const ModalRoot = React.forwardRef<HTMLDivElement, ModalProps>(
  (
    {
      trigger,
      children,
      showClose = true,
      overlayColorToken,
      backgroundColorToken,
      className = "",
      width = "405px",
      ...rest
    },
    ref,
  ) => {
    const modalStyle = {
      "--modal-overlay-color": overlayColorToken
        ? getColorTokenVar(overlayColorToken)
        : undefined,
      "--modal-bg-color": backgroundColorToken
        ? getColorTokenVar(backgroundColorToken)
        : undefined,
      "--modal-width": width,
    } as React.CSSProperties;

    return (
      <RadixDialog.Root {...rest}>
        {trigger && (
          <RadixDialog.Trigger asChild>{trigger}</RadixDialog.Trigger>
        )}
        <RadixDialog.Portal>
          <RadixDialog.Overlay className="ds-modal__overlay" style={modalStyle} />
          <RadixDialog.Content
            ref={ref}
            className={`ds-modal__content ${className}`}
            style={modalStyle}
          >
            {children}
            {showClose && (
              <RadixDialog.Close asChild>
                <Button className="ds-modal__close" aria-label="Close">
                  <Cross1Icon width="12px" height="12px" />
                </Button>
              </RadixDialog.Close>
            )}
          </RadixDialog.Content>
        </RadixDialog.Portal>
      </RadixDialog.Root>
    );
  },
);

ModalRoot.displayName = "Modal";

const ModalHeader = React.forwardRef<HTMLDivElement, ModalHeaderProps>(
  ({ title, description, titleColorToken, descriptionColorToken, className = "", children }, ref) => (
    <Box ref={ref} className={`ds-modal__header ${className}`}>
      <Box className="ds-modal__header-text">
        <RadixDialog.Title asChild>
          <Text as="div" className="ds-modal__title" size="3" weight="bold">
            {title}
          </Text>
        </RadixDialog.Title>
        {description && (
          <RadixDialog.Description asChild>
            <Text as="div" className="ds-modal__description" size="1">
              {description}
            </Text>
          </RadixDialog.Description>
        )}
      </Box>
      {children}
    </Box>
  ),
);

ModalHeader.displayName = "Modal.Header";

const ModalContent = React.forwardRef<HTMLDivElement, ModalContentProps>(
  ({ children, maxHeight = "480px", className = "" }, ref) => (
    <Box
      ref={ref}
      className={`ds-modal__body ${className}`}
      style={{ "--modal-content-max-height": maxHeight } as React.CSSProperties}
    >
      {children}
    </Box>
  ),
);

ModalContent.displayName = "Modal.Content";

const ModalFooter = React.forwardRef<HTMLDivElement, ModalFooterProps>(
  ({ children, className = "" }, ref) => (
    <Box ref={ref} className={`ds-modal__footer ${className}`}>
      {children}
    </Box>
  ),
);

ModalFooter.displayName = "Modal.Footer";

export const Modal = Object.assign(ModalRoot, {
  Header: ModalHeader,
  Content: ModalContent,
  Footer: ModalFooter,
});
```

### Modal.stories.tsx (simplified)

```tsx
import type { Meta, StoryObj } from "@storybook/react";
import { Text } from "@radix-ui/themes";
import { Modal } from "./Modal";
import { Button } from "../../atoms";

const meta: Meta<typeof Modal> = {
  title: "Molecules/Modal",
  component: Modal,
  tags: ["autodocs"],
};
export default meta;

type Story = StoryObj<typeof Modal>;

export const ScrollableContent: Story = {
  args: {
    trigger: <Button>Open Long Modal</Button>,
    showClose: true,
    children: (
      <>
        <Modal.Header
          title="Terms and Conditions"
          description="Please read through our terms and conditions."
        />
        <Modal.Content maxHeight="300px">
          <Text size="2">Long scrollable content goes here...</Text>
        </Modal.Content>
        <Modal.Footer>
          <Button variant="ghost">Decline</Button>
          <Button color="blue">Accept</Button>
        </Modal.Footer>
      </>
    ),
  },
};


export const Playground: Story = {
  args: { ...Default.args },
  argTypes: {
    showClose: { control: "boolean" },
    width: { control: "text" },
  },
};
```

---

## 7. Tips & Gotchas

| Issue | Solution |
|-------|----------|
| Claude Code ignores your tokens and hardcodes values | Your `CLAUDE.md` rules aren't specific enough — list forbidden patterns explicitly with examples |
| Generated component doesn't match Figma spacing exactly | Figma MCP reads auto-layout values — make sure the Figma component uses auto-layout, not absolute positioning |
| Storybook MCP can't find existing components | Storybook must be running (`npm run storybook`) before you launch `claude` |
| Claude Code generates raw `<button>` instead of your `Button` | Add an explicit "forbidden elements" list in `CLAUDE.md` — Claude follows explicit rules better than implicit ones |
| Generated stories are too minimal | Require a minimum story count in `CLAUDE.md`: "Every component must have at least 4 stories: Default, each variant, edge states, Playground" |
| Component structure doesn't match your project | Run `claude /init` first so Claude Code scans your existing codebase — then refine `CLAUDE.md` |
| MCP server disconnects mid-session | Check Node.js version (18+ required), restart `claude` — MCP servers reconnect on startup |
| Figma MCP not connecting | Figma Desktop must be open with the design file active and MCP server enabled in Inspect panel |