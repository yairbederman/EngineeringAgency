# Figma – Design System Translation Protocol

> **Load When**: Frontend/UI tasks

**Purpose**: Translate visual intent into project-compliant Design Tokens, Component Structures, AND visual context for LLM implementation.

## When to Use

Use Figma when:
- Working on any Frontend/UI task
- The spec references Figma files/frames

---

## Translation Protocol (The "Design-to-Code" Bridge)

1. **Extract, Don't Guess**: Use Figma MCP to read the frame properties.

2. **Extract Variables First**: Call `${MCP_FIGMA_GET_VARS}` to get designer-defined semantic tokens (highest priority).

3. **Capture Screenshots (Visual Grounding)**:
   - Call `${MCP_FIGMA_GET_SCREENSHOT}` for each frame
   - Embed screenshots in Design Review Report and Task descriptions
   - Add visual annotations for pinned elements, scroll areas, visual rhythm
   - **Strict Rule**: LLMs need to "see" the design, not just parse tokens

4. **Extract Interaction States**:
   - Parse `variants.availableVariants` for component states (Default, Hover, Pressed, Focused, Disabled)
   - Document state transitions and timing
   - Flag missing states (especially Focus for a11y)
   - See: `figma-extraction-protocol.md` Step 2I

5. **Map to Tokens (CRITICAL)**:
   - *Do not* use raw values (e.g., `#1D4ED8`, `16px`) unless they are one-off overrides
   - *Priority*: Figma Variables > Style Names > `design-tokens.json` match > Algorithmic closest match
   - *Do*: Map Figma values to the project's Design System found in `${COPILOT_INSTRUCTIONS_PATH}` or `${DESIGN_TOKENS_PATH}`

6. **Component Identification (Enhanced)**:
   - Identify Figma component instances and match to project components
   - Extract FULL context: props, slots, stateProps, usage examples
   - See: `figma-extraction-protocol.md` Step 3.5 for enhanced component schema

---

## Tool Failure & Missing Designs

- If Figma is unreachable: Ask for a **screenshot**
- If designs are missing: Use `[TBD – Design]` placeholders
- **Strict Rule**: Do not invent UI. If layout is unknown, implement a semantic skeleton (stack/group) without specific spacing/colors.

---

## Component-First Development Protocol

> **Principle**: Reuse > Recreate. Before implementing any UI element, verify it doesn't already exist.

### Workflow

1. **Extract Component Instances from Figma**:
   - When `${MCP_FIGMA_GET_DESIGN}` returns component instances, extract their names
   - Common patterns: `Button/Primary`, `Icon/Search`, `Avatar/Medium`, `Card/Default`

2. **Cross-Reference with Project Component Registry**:
   - Check `${FILE_CATEGORIZATION_PATH}` for `react-components` category
   - Match Figma component names to existing project components

3. **Implementation Decision Tree**:
   ```
   Is there an exact match in project components?
   ├── YES → Use existing component, pass appropriate props
   └── NO
       └── Is there a partial match (similar component)?
           ├── YES → Extend existing component with new variant
           └── NO → Create NEW component following project patterns
   ```

4. **Documentation Requirement**:
   - **MANDATORY**: Every Frontend task MUST include "Component Instances" section
   - Missing this section = Task is NOT implementation-ready
   - Each instance must specify: Figma name → Project component → Import path
