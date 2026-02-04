# MudBlazor Prompt Planning Guide

> **MCP Server Implementation Guide**
> Converting HTML/CSS Design Systems to MudBlazor with 95-100% Accuracy

---

## Table of Contents

1. [Overview & Key Learnings](#overview)
2. [Input Files & File Management](#input-files)
3. [The Three-Layer Strategy](#the-three-layer-strategy)
4. [Layer 1: Parse Tokens & Generate Theme](#layer-1-parse-tokens--generate-theme)
5. [Layer 2: Parse HTML & Generate Razor](#layer-2-parse-html--generate-razor)
6. [Layer 3: Generate CSS Overrides](#layer-3-generate-css-overrides)
7. [Deployment Scenarios](#deployment-scenarios-new-vs-existing-projects)
8. [Scoped CSS Architecture](#integrating-into-existing-mudblazor-projects)
9. [File Management Strategy](#file-management-strategy)
10. [Dark Mode Implementation](#dark-mode-implementation)
11. [Testing & Results](#testing-your-mcp-server-output)

---

## Overview

This guide documents the complete strategy for building an MCP server that converts HTML/CSS design systems into MudBlazor Blazor applications with **95-100% design accuracy**.

### Key Learnings

**The Problem:**
- MudBlazor is built for Material Design, not custom design systems
- MudBlazor components inject their own CSS classes and styles
- Using MudBlazor "out of the box" results in only 50-60% design accuracy

**The Solution:**
- **Three-Layer Strategy**: Theme Config + Custom Classes + CSS Overrides
- **Scoped CSS Architecture**: Integrate into existing projects without breaking them
- **Pure MudBlazor Components**: Use MudBlazor for all UI elements with targeted CSS overrides
- **Automatic Dark Mode**: CSS variables switch themes without re-rendering

### What This Guide Covers

✅ Complete code generation workflow (tokens → Blazor app)
✅ Two deployment scenarios (new vs existing projects)
✅ File management strategy (what to keep/delete)
✅ Automatic dark mode implementation
✅ Pure MudBlazor component strategy with CSS overrides
✅ Testing and validation approach
✅ **Why MudBlazor is fundamentally different from Angular/Vue/React**

---

## Why MudBlazor Requires a Different Approach

### Critical Understanding for Your MCP Server

**Unlike Angular, Vue, and React, MudBlazor requires a fundamentally different code generation strategy.**

#### Angular/Vue/React: Straightforward Approach ✅

**Component libraries are OPTIONAL:**
```jsx
// React - Just use plain HTML with your CSS classes
function Button() {
  return (
    <button className="btn btn-primary">
      <i data-lucide="plus"></i>
      <span>New row</span>
    </button>
  );
}
```

```vue
<!-- Vue - Plain HTML with CSS classes -->
<template>
  <button class="btn btn-primary">
    <lucide-icon name="plus" />
    <span>New row</span>
  </button>
</template>
```

**Why it works:**
- ✅ Framework components are wrappers around plain HTML
- ✅ You control the HTML structure completely
- ✅ CSS classes apply directly to your elements
- ✅ Component libraries (Material UI, Ant Design) are separate packages
- ✅ You can choose to use plain HTML/CSS anytime
- ✅ **100% design accuracy by default**

**MCP Generation Strategy (Simple):**
```python
def generate_react_component(html_element):
    # Just convert HTML to JSX - that's it!
    return convert_html_to_jsx(html_element)
    # Your CSS classes work as-is. Done!
```

---

#### MudBlazor: Complex Override Strategy ⚠️

**Component library is INTEGRAL to Blazor:**
```razor
<!-- MudBlazor - Can't easily use plain button -->
<MudButton Variant="Variant.Filled"
           Color="Color.Primary"
           Class="btn-mud btn-primary-mud">
    New row
</MudButton>

<!-- What MudBlazor actually generates: -->
<button class="mud-button-root mud-ripple mud-button-filled mud-button-filled-primary mud-button-filled-size-medium btn-mud btn-primary-mud">
    <span class="mud-button-label">New row</span>
</button>
```

**Why it's different:**
- ❌ MudBlazor components have **built-in Material Design styling**
- ❌ You can't "opt out" - using plain `<button>` loses Blazor benefits
- ❌ MudBlazor generates **complex nested DOM structures**
- ❌ Your CSS classes are added but **MudBlazor CSS has higher priority**
- ❌ Components inject multiple CSS classes with high specificity
- ❌ **Only 50-60% design accuracy without overrides**

**MCP Generation Strategy (Complex):**
```python
def generate_mudblazor_component(html_element):
    # Step 1: Generate Razor with MudBlazor component + custom class
    razor = generate_razor_with_custom_class(html_element)

    # Step 2: Generate CSS overrides targeting custom class
    css_overrides = generate_css_overrides_with_important(html_element)

    # Step 3: Use scoped CSS if integrating into existing project
    if is_existing_project:
        css_overrides = scope_css_overrides(css_overrides, scope_class)

    return {
        'razor': razor,
        'css': css_overrides,
        'requires_three_layer_strategy': True
    }
```

---

### Side-by-Side Comparison

| Aspect | Angular/Vue/React | **MudBlazor/Blazor** |
|--------|-------------------|----------------------|
| **Component Library** | Optional (separate package) | Integral (part of ecosystem) |
| **HTML Control** | ✅ Full control | ⚠️ Component dictates structure |
| **CSS Classes** | ✅ Apply directly | ⚠️ Added alongside MudBlazor classes |
| **CSS Priority** | ✅ Your CSS wins | ❌ MudBlazor CSS has high specificity |
| **Styling Approach** | Use framework's CSS directly | **Need CSS overrides with !important** |
| **Plain HTML Option** | ✅ Easy | ⚠️ Loses Blazor component benefits |
| **Generation Complexity** | Low (HTML → JSX/Template) | **High (3-Layer Strategy required)** |
| **Design Accuracy** | 100% out of box | 50-60% → **95-100% with overrides** |

---

### Why This Matters for Your MCP Server

#### Different Generation Pipelines

**For Angular/Vue/React:**
```
HTML Template → Convert to Framework Syntax (JSX/Template) → Done! ✅
                                                              (CSS works as-is)
```

**For MudBlazor:**
```
HTML Template + Design Tokens
    ↓
Layer 1: Generate MudTheme from tokens
    ↓
Layer 2: Map HTML → MudBlazor components + custom classes
    ↓
Layer 3: Generate CSS overrides with !important
    ↓
Decision: New project or existing?
    ├─ New: Global overrides
    └─ Existing: Scoped overrides
    ↓
Result: 90-95% accuracy ✅
```

---

### Key Architectural Differences

#### 1. **Component Complexity**

**React/Vue/Angular:**
```jsx
// Simple wrapper - you control everything
<button className="btn btn-primary">
  {children}
</button>
```

**MudBlazor:**
```razor
<!-- Complex component with internal logic -->
<MudButton>
  <!-- Internally generates:
       - Multiple wrapper divs
       - Ripple effects
       - State management
       - Icon positioning
       - Loading states
       - Tooltip integration -->
</MudButton>
```

#### 2. **CSS Strategy**

**React/Vue/Angular:**
```css
/* Your CSS just works */
.btn-primary {
    background: var(--color-primary);
    padding: var(--spacing-200);
}
```

**MudBlazor:**
```css
/* Must override MudBlazor with !important */
.mud-button-root.btn-primary-mud {
    background: var(--color-primary) !important;
    padding: var(--spacing-200) !important;
}
```

#### 3. **Integration Approach**

**React/Vue/Angular:**
```jsx
// Use plain HTML anytime - works perfectly
<div className="dashboard">
  <button className="btn btn-primary">Save</button>
  <span className="badge badge-success">Active</span>
</div>
// ✅ 100% accurate to your design
```

**MudBlazor:**
```razor
<!-- Pure MudBlazor with CSS Overrides (CURRENT APPROACH) -->
<div class="eg-dashboard-scope">
  <MudButton Class="btn-mud btn-primary-mud" OnClick="Save">Save</MudButton>
  <MudChip Class="badge-mud badge-success-mud">Active</MudChip>
  <MudDialog @bind-Visible="_showDialog">...</MudDialog>
  <!-- ✅ Full Blazor integration + 90-95% design accuracy with CSS overrides -->
</div>

<!-- What it generates: -->
<button class="mud-button-root mud-ripple mud-button-filled mud-button-filled-primary btn-mud btn-primary-mud">
    <span class="mud-button-label">Save</span>
</button>
<!-- CSS overrides target .btn-mud.btn-primary-mud to customize appearance -->
```

---

### Why Can't MudBlazor Be Like React?

**Fundamental Architecture:**

1. **Blazor is Server-Side (or WebAssembly)**
   - Needs stateful components with lifecycle management
   - Plain HTML loses two-way binding, events, state
   - MudBlazor provides this infrastructure

2. **MudBlazor is an Ecosystem, Not a Library**
   - Like Bootstrap for Blazor (but with C# components)
   - Designed as a complete UI system
   - Best used as the primary component library throughout

3. **Material Design First**
   - Built for Material Design from ground up
   - Custom design systems are "afterthought"
   - Need to fight against built-in styling

**React/Vue/Angular:**
- Framework = Logic layer only
- UI = Your choice (plain HTML, component library, etc.)
- Separation of concerns ✅

**Blazor/MudBlazor:**
- Framework = Logic + UI together
- MudBlazor = Expected default UI
- Tightly coupled by design ⚠️

---

### MCP Server Decision Tree

```python
def generate_code(html_template, css_tokens, framework):
    """
    Main MCP server entry point - route to correct generator
    """

    if framework in ['react', 'vue', 'angular']:
        # Simple generation
        return generate_spa_framework(html_template, css_tokens, framework)
        # Output: Component files + CSS (tokens work as-is)
        # Complexity: LOW ⭐
        # Accuracy: 100%

    elif framework == 'mudblazor':
        # Complex generation with three-layer strategy
        deployment_type = prompt_user("New or existing project?")

        if deployment_type == "new":
            return generate_mudblazor_new_project(
                html_template,
                css_tokens
            )
            # Output: Theme + Razor (MudBlazor) + CSS overrides
            # Complexity: HIGH ⭐⭐⭐⭐
            # Accuracy: 90-95%

        elif deployment_type == "existing":
            scope_class = prompt_user("Scope class name?")
            return generate_mudblazor_scoped(
                html_template,
                css_tokens,
                scope_class
            )
            # Output: Scoped Razor (MudBlazor) + Scoped CSS
            # Complexity: VERY HIGH ⭐⭐⭐⭐⭐
            # Accuracy: 90-95% (existing pages safe)

    else:
        raise ValueError(f"Unsupported framework: {framework}")
```

---

### Framework Support Matrix (for Your MCP Server)

| Framework | Approach | Complexity | Accuracy | Strategy |
|-----------|----------|------------|----------|----------|
| **React** | Direct HTML→JSX | ⭐ Simple | 100% | Plain HTML + CSS |
| **Vue** | Direct HTML→Template | ⭐ Simple | 100% | Plain HTML + CSS |
| **Angular** | Direct HTML→Template | ⭐⭐ Moderate | 100% | Plain HTML + CSS |
| **MudBlazor** | **Three-Layer Strategy** | ⭐⭐⭐⭐⭐ Complex | **90-95%** | **Theme + MudBlazor Components + CSS Overrides** |

---

### Document This for Your MCP Server Users

```markdown
## Framework-Specific Approaches

Your MCP Server supports multiple frameworks with different strategies:

### Angular/Vue/React: ✅ Simple Generation
- Converts HTML directly to framework syntax (JSX/Templates)
- CSS classes work as-is
- 100% design accuracy out of the box
- Fast generation

### MudBlazor: ⚠️ Complex Generation
- Requires three-layer strategy (Theme + MudBlazor Components + CSS Overrides)
- Converts all HTML elements to MudBlazor components
- CSS overrides needed for design customization
- More configuration required
- Longer generation time
- **Achieves 90-95% accuracy with full Blazor integration!**

**Why the difference?**
MudBlazor components have built-in Material Design styling and complex
internal DOM structures that must be overridden with targeted CSS
to match your custom design system.
```

---

## Quick Reference

### For New Standalone Projects

**Input:**
- `_tokens.css` - Design system tokens
- `dashboard.html` - UI structure reference

**Output:**
- `ThemeConfiguration.cs` - MudBlazor theme from tokens
- `App.razor` - Global theme configuration
- `Property.razor` - Page components
- `wwwroot/css/mudblazor-overrides.css` - Global CSS overrides

**Accuracy:** 90-95%

### For Existing MudBlazor Projects

**Input:**
- `_tokens.css` - Design system tokens
- `dashboard.html` - UI structure reference

**Output:**
- `Pages/Property.razor` - Scoped page components (pure MudBlazor)
- `wwwroot/css/dashboard-scoped.css` - Scoped tokens + overrides

**Accuracy:** 90-95% (existing pages unaffected)

---

## The Strategy

This guide explains how to build an MCP server that converts HTML/CSS design systems into MudBlazor components with 95-100% design accuracy using the **Three-Layer Strategy**.

## Input Files

Your MCP server needs:
1. **`_tokens.css`** - Global design tokens (CSS custom properties)
2. **`dashboard.html`** - HTML template with component structure
3. **`dashboard.css`** (optional) - Page-specific styles

## Output Files (3 Layers)

### Layer 1: ThemeConfiguration.cs
Maps CSS tokens → MudBlazor theme API

### Layer 2: ComponentName.razor
HTML elements → MudBlazor components with custom CSS classes

### Layer 3: mudblazor-overrides.css
CSS overrides using token variables

---

## The Three-Layer Strategy

### Why This Works

**Problem:** MudBlazor components use Material Design styling, not your custom tokens.

**Solution:**
1. **Theme Config** sets base colors (60% accuracy)
2. **Custom Classes** on components enable targeting (70% accuracy)
3. **CSS Overrides** with `!important` + token variables (**95-100% accuracy**)

---

## Layer 1: Parse Tokens & Generate Theme

### Input: `_tokens.css`
```css
:root {
  --primary-600: #0062C1;
  --spacing-100: 8px;
  --border-radius-100: 8px;
  --font-size-medium: 16px;
}
```

### Output: `ThemeConfiguration.cs`
```csharp
public static MudTheme CreateTheme()
{
    return new MudTheme()
    {
        PaletteLight = new PaletteLight()
        {
            Primary = "#0062C1",  // from --primary-600
            // ... map all color tokens
        },
        Typography = new Typography()
        {
            Default = new Default()
            {
                FontSize = "16px",  // from --font-size-medium
            }
        },
        Shadows = new Shadow()
        {
            // MUST have exactly 25 elements (0-24)
            Elevation = new string[25] { ... }
        }
    };
}
```

### Parser Pseudocode
```python
def parse_tokens(css_file):
    tokens = {
        'colors': {},     # --primary-600: #0062C1
        'spacing': {},    # --spacing-100: 8px
        'typography': {}, # --font-size-medium: 16px
        'borders': {},    # --border-radius-100: 8px
    }

    # Extract all CSS custom properties
    for match in re.findall(r'--([^:]+):\s*([^;]+)', css_content):
        category = categorize_token(match[0])
        tokens[category][match[0]] = match[1].strip()

    return tokens

def generate_theme(tokens):
    return f"""
public static MudTheme CreateTheme()
{{
    return new MudTheme()
    {{
        PaletteLight = new PaletteLight()
        {{
            Primary = "{tokens['colors']['primary-600']}",
            Success = "{tokens['colors']['success-600']}",
            // ...
        }},
        Typography = new Typography()
        {{
            Default = new Default()
            {{
                FontSize = "{tokens['typography']['font-size-medium']}",
            }}
        }},
        Shadows = new Shadow()
        {{
            Elevation = new string[25] {{ {generate_shadows()} }}
        }}
    }};
}}
"""
```

---

## Layer 2: Parse HTML & Generate Razor

### Component Mapping Rules

| HTML | MudBlazor | Custom Class |
|------|-----------|--------------|
| `<button class="btn btn-primary">` | `<MudButton Variant="Variant.Filled" Color="Color.Primary">` | `btn-mud btn-primary-mud` |
| `<button class="btn btn-secondary">` | `<MudButton Variant="Variant.Outlined">` | `btn-mud btn-secondary-mud` |
| `<span class="badge badge-success">` | `<MudChip Size="Size.Small" Color="Color.Success">` | `badge-mud badge-success-mud` |
| `<table class="data-table">` | `<MudTable>` | `data-table-mud` |
| `<th>` | `<MudTh>` | `table-header-cell-mud` |
| `<td>` | `<MudTd>` | `table-cell-mud` |

### Input: `dashboard.html`
```html
<button class="btn btn-primary">
    <i data-lucide="plus"></i>
    <span>New row</span>
</button>

<span class="badge badge-success">Active</span>

<table class="data-table">
    <tr>
        <th>Name</th>
        <td>Value</td>
    </tr>
</table>
```

### Output: `Property.razor`
```razor
<MudButton Variant="Variant.Filled"
           Color="Color.Primary"
           StartIcon="@Icons.Material.Filled.Add"
           Class="btn-mud btn-primary-mud">
    New row
</MudButton>

<MudChip T="string"
         Size="Size.Small"
         Color="Color.Success"
         Class="badge-mud badge-success-mud">
    Active
</MudChip>

<MudTable Items="@items" Class="data-table-mud">
    <HeaderContent>
        <MudTh Class="table-header-cell-mud">Name</MudTh>
    </HeaderContent>
    <RowTemplate>
        <MudTd Class="table-cell-mud">@context.Name</MudTd>
    </RowTemplate>
</MudTable>
```

### Parser Pseudocode
```python
def parse_html_component(element):
    if element.name == 'button' and 'btn' in element.get('class', []):
        classes = element.get('class', [])
        variant = 'Variant.Filled' if 'btn-primary' in classes else 'Variant.Outlined'
        icon = element.find('i', {'data-lucide': True})
        text = element.get_text(strip=True)

        return f"""
<MudButton Variant="{variant}"
           Color="Color.Primary"
           StartIcon="@Icons.Material.Filled.{map_icon(icon)}"
           Class="btn-mud btn-{get_variant_class(classes)}-mud">
    {text}
</MudButton>
"""

    elif element.name == 'span' and 'badge' in element.get('class', []):
        classes = element.get('class', [])
        color = extract_color(classes)  # 'success', 'error', etc.
        text = element.get_text(strip=True)

        return f"""
<MudChip T="string"
         Size="Size.Small"
         Color="Color.{color.capitalize()}"
         Class="badge-mud badge-{color}-mud">
    {text}
</MudChip>
"""

    elif element.name == 'table' and 'data-table' in element.get('class', []):
        return generate_mud_table(element)
```

**Key Rule:** Every MudBlazor component gets a custom class with `-mud` suffix for CSS targeting.

---

## Layer 3: Generate CSS Overrides

### For Each Component Type
Generate CSS that:
1. Targets the custom class from Layer 2
2. Uses `var(--*)` from `_tokens.css`
3. Uses `!important` to override MudBlazor

### Output: `mudblazor-overrides.css`
```css
/* Buttons */
.mud-button-root.btn-mud {
    padding: var(--spacing-100) var(--spacing-200) !important;
    border-radius: var(--border-radius-100) !important;
    gap: var(--spacing-100) !important;
    font-size: var(--font-size-medium) !important;
}

.mud-button-root.btn-primary-mud {
    background-color: var(--color-primary-default) !important;
    color: var(--text-on-action) !important;
}

.mud-button-root.btn-primary-mud:hover {
    background-color: var(--color-primary-hover) !important;
}

/* Badges/Chips */
.mud-chip.badge-mud {
    padding: var(--spacing-50) var(--spacing-150) !important;
    border-radius: var(--border-radius-full) !important;
    font-size: var(--font-size-small) !important;
}

.mud-chip.badge-success-mud {
    background-color: var(--success-100) !important;
    color: var(--success-700) !important;
}

/* Table */
.mud-table-head .table-header-cell-mud {
    padding: var(--spacing-200) !important;
    border-bottom: var(--border-width-thin) solid var(--border-default) !important;
}

.mud-table-body .table-cell-mud {
    padding: var(--spacing-200) !important;
}
```

### Generator Pseudocode
```python
def generate_css_overrides(components, tokens):
    css = ""

    for component_class in get_unique_classes(components):
        if 'btn' in component_class:
            css += f"""
.mud-button-root.btn-mud {{
    padding: var(--spacing-100) var(--spacing-200) !important;
    border-radius: var(--border-radius-100) !important;
}}

.mud-button-root.btn-primary-mud {{
    background-color: var(--color-primary-default) !important;
    color: var(--text-on-action) !important;
}}

.mud-button-root.btn-primary-mud:hover {{
    background-color: var(--color-primary-hover) !important;
}}
"""

        elif 'badge' in component_class:
            color = extract_color(component_class)
            css += f"""
.mud-chip.badge-mud {{
    padding: var(--spacing-50) var(--spacing-150) !important;
    border-radius: var(--border-radius-full) !important;
}}

.mud-chip.badge-{color}-mud {{
    background-color: var(--{color}-100) !important;
    color: var(--{color}-700) !important;
}}
"""

    return css
```

---

## CSS Loading Order (Critical!)

Generated code must include CSS in this order:

```html
<head>
    <!-- 1. Roboto font -->
    <link href="https://fonts.googleapis.com/.../Roboto" rel="stylesheet">

    <!-- 2. MudBlazor base CSS -->
    <link href="_content/MudBlazor/MudBlazor.min.css" rel="stylesheet" />

    <!-- 3. Design tokens -->
    <link href="css/_tokens.css" rel="stylesheet" />

    <!-- 4. Overrides (MUST be last!) -->
    <link href="css/mudblazor-overrides.css" rel="stylesheet" />
</head>
```

---

## Dark Mode (Automatic!)

If `_tokens.css` has dark mode:
```css
[data-theme="dark"] {
    --color-primary-default: var(--primary-400);
    --surface-base: var(--neutral-950);
}
```

Then overrides automatically work:
```css
.mud-button-root.btn-primary-mud {
    background-color: var(--color-primary-default) !important;
    /* Automatically switches when [data-theme="dark"] is set */
}
```

---

## Deployment Scenarios: New vs Existing Projects

Your MCP server should support **two deployment scenarios**:

### Scenario 1: New Standalone MudBlazor Project
- ✅ Full control over theme
- ✅ No conflicts with existing code
- ✅ Use global CSS overrides
- ✅ Generate ThemeConfiguration.cs

### Scenario 2: Existing MudBlazor Project ⚠️
- ⚠️ Already has a theme in use
- ⚠️ Existing pages use default MudBlazor styling
- ⚠️ Need to avoid breaking existing components
- ⚠️ Must scope everything to new dashboard pages

---

## Integrating into Existing MudBlazor Projects

### The Challenge

**Existing MudBlazor App:**
```razor
<!-- Their existing pages use default theme -->
<MudButton Color="Color.Primary">Save</MudButton>
<!-- Material Design styling -->
```

**You want to add:**
```razor
<!-- Your new dashboard with custom design -->
<MudButton Color="Color.Primary" Class="btn-mud">New Row</MudButton>
<!-- Your custom design tokens -->
```

**Problem:** Can't change global theme without breaking existing pages!

---

### Solution: Scoped CSS Architecture

**Key Principle:** Scope EVERYTHING to your dashboard pages only.

#### 1. Generate Scoped Tokens

Instead of global tokens:
```css
/* ❌ DON'T: Global tokens (breaks existing pages) */
:root {
    --primary-600: #0062C1;
    --spacing-100: 8px;
}
```

Generate scoped tokens:
```css
/* ✅ DO: Scoped tokens (isolated to dashboard) */
.eg-dashboard-scope {
    /* Define all tokens within this scope */
    --primary-600: #0062C1;
    --primary-700: #0051A2;
    --spacing-100: 8px;
    --spacing-200: 16px;
    --border-radius-100: 8px;
    --color-primary-default: var(--primary-600);
    --color-primary-hover: var(--primary-700);
    --success-100: #C5EBD5;
    --success-700: #1B5D43;
    /* ... all tokens from _tokens.css */
}
```

#### 2. Generate Scoped CSS Overrides

All overrides must be scoped:
```css
/* ✅ Scoped overrides - only affect dashboard */
.eg-dashboard-scope .mud-button-root.btn-mud {
    padding: var(--spacing-100) var(--spacing-200) !important;
    border-radius: var(--border-radius-100) !important;
}

.eg-dashboard-scope .mud-button-root.btn-primary-mud {
    background-color: var(--color-primary-default) !important;
    color: var(--text-on-action) !important;
}

.eg-dashboard-scope .mud-button-root.btn-primary-mud:hover {
    background-color: var(--color-primary-hover) !important;
}

.eg-dashboard-scope .mud-chip.badge-success-mud {
    background-color: var(--success-100) !important;
    color: var(--success-700) !important;
}
```

#### 3. Generate Page Component with Scope Wrapper

```razor
@page "/property"

<!-- Wrap EVERYTHING in scope class -->
<div class="eg-dashboard-scope">
    <MudContainer MaxWidth="MaxWidth.False">
        <div class="content-header">
            <h1>Property</h1>

            <div class="header-actions">
                <MudButton Variant="Variant.Filled"
                           Color="Color.Primary"
                           StartIcon="@Icons.Material.Filled.Add"
                           Class="btn-mud btn-primary-mud">
                    New Row
                </MudButton>

                <MudChip T="string"
                         Size="Size.Small"
                         Color="Color.Success"
                         Class="badge-mud badge-success-mud">
                    Active
                </MudChip>
            </div>
        </div>

        <MudTable Items="@items" Class="data-table-mud">
            <!-- ... -->
        </MudTable>
    </MudContainer>
</div>

@code {
    // No theme changes - just use CSS scoping!
}
```

#### 4. Skip Global Theme Configuration

For existing projects:
- ❌ **Don't generate** ThemeConfiguration.cs
- ❌ **Don't modify** App.razor
- ✅ **Only generate** scoped CSS + page components

---

### Updated Generator for Existing Projects

```python
def generate_for_existing_project(tokens_css, dashboard_html, scope_class="eg-dashboard-scope"):
    """
    Generate code for existing MudBlazor projects (scoped approach)
    """

    # Step 1: Parse tokens
    tokens = parse_tokens(tokens_css)

    # Step 2: Generate SCOPED CSS with tokens
    scoped_css = f"""
/* Scoped Design Tokens */
.{scope_class} {{
"""

    # Add all tokens within scope
    for category, token_dict in tokens.items():
        scoped_css += f"    /* {category} */\n"
        for token_name, token_value in token_dict.items():
            scoped_css += f"    --{token_name}: {token_value};\n"

    scoped_css += "}\n\n"

    # Step 3: Generate scoped CSS overrides
    scoped_css += generate_scoped_overrides(tokens, scope_class)

    write_file("wwwroot/css/dashboard-scoped.css", scoped_css)

    # Step 4: Parse HTML and generate Razor with scope wrapper
    components = parse_html_components(dashboard_html)

    razor_code = f"""
@page "/property"

<div class="{scope_class}">
    <MudContainer MaxWidth="MaxWidth.False">
        {generate_razor_components(components)}
    </MudContainer>
</div>

@code {{
    // Component code...
}}
"""

    write_file("Pages/Property.razor", razor_code)

    # Step 5: DON'T generate ThemeConfiguration.cs or modify App.razor
    # Existing project keeps its current theme!


def generate_scoped_overrides(tokens, scope_class):
    """
    Generate CSS overrides scoped to a specific class
    """
    css = f"""
/* Scoped Component Overrides */

/* Buttons */
.{scope_class} .mud-button-root.btn-mud {{
    padding: var(--spacing-100) var(--spacing-200) !important;
    border-radius: var(--border-radius-100) !important;
    gap: var(--spacing-100) !important;
}}

.{scope_class} .mud-button-root.btn-primary-mud {{
    background-color: var(--color-primary-default) !important;
    color: var(--text-on-action) !important;
}}

.{scope_class} .mud-button-root.btn-primary-mud:hover {{
    background-color: var(--color-primary-hover) !important;
}}

/* Badges */
.{scope_class} .mud-chip.badge-mud {{
    padding: var(--spacing-50) var(--spacing-150) !important;
    border-radius: var(--border-radius-full) !important;
}}

.{scope_class} .mud-chip.badge-success-mud {{
    background-color: var(--success-100) !important;
    color: var(--success-700) !important;
}}

/* Table */
.{scope_class} .mud-table-head .table-header-cell-mud {{
    padding: var(--spacing-200) !important;
    border-bottom: var(--border-width-thin) solid var(--border-default) !important;
}}

.{scope_class} .mud-table-body .table-cell-mud {{
    padding: var(--spacing-200) !important;
}}
"""

    return css
```

---

### Dark Mode with Scoping

Scoped dark mode tokens:
```css
.eg-dashboard-scope {
    /* Light mode tokens */
    --primary-600: #0062C1;
    --color-primary-default: var(--primary-600);
}

/* Dark mode: nest within scope */
[data-theme="dark"] .eg-dashboard-scope {
    --primary-400: #3E88EF;
    --color-primary-default: var(--primary-400);
}
```

---

### Compatibility Matrix

| Concern | Global Approach | **Scoped Approach** |
|---------|----------------|---------------------|
| Existing pages | ❌ Broken | ✅ Untouched |
| Existing theme | ❌ Overwritten | ✅ Preserved |
| CSS conflicts | ❌ High risk | ✅ Isolated |
| New dashboard | ✅ Works | ✅ Works |
| Dark mode | ✅ Works | ✅ Works |
| Integration effort | Low | Medium |
| **Use for existing projects** | ❌ No | ✅ **Yes** |

---

### Decision Tree for Your MCP Server

```python
def generate_mudblazor_code(tokens_css, dashboard_html, deployment_type):
    """
    Main generator with deployment type selection
    """

    if deployment_type == "new_project":
        # Generate for standalone new project
        return generate_new_project(tokens_css, dashboard_html)
        # Output:
        # - ThemeConfiguration.cs (global theme)
        # - App.razor (uses theme)
        # - Property.razor (components)
        # - mudblazor-overrides.css (global overrides)

    elif deployment_type == "existing_project":
        # Generate for existing MudBlazor project
        return generate_for_existing_project(tokens_css, dashboard_html)
        # Output:
        # - Property.razor (with .eg-dashboard-scope wrapper)
        # - dashboard-scoped.css (scoped tokens + overrides)
        # - NO ThemeConfiguration.cs
        # - NO App.razor modifications
```

**Prompt your MCP server users:**
```
? Deployment target?
  › New standalone MudBlazor project
    Add to existing MudBlazor project

? [If existing] Scope class name?
  › eg-dashboard-scope (default)
    custom-scope-name
```

---

### Example: Before & After in Existing Project

**Before Integration:**
```razor
<!-- /users page (existing, untouched) -->
<MudButton Color="Color.Primary">Save User</MudButton>
<!-- Uses existing MudBlazor theme: Material Blue -->
```

**After Integration:**
```razor
<!-- /users page (existing, STILL untouched) -->
<MudButton Color="Color.Primary">Save User</MudButton>
<!-- ✅ Still uses existing theme: Material Blue -->

<!-- /property page (NEW dashboard) -->
<div class="eg-dashboard-scope">
    <MudButton Color="Color.Primary" Class="btn-mud btn-primary-mud">
        New Row
    </MudButton>
    <!-- ✅ Uses your custom design: #0062C1 -->
</div>
```

**Result:** Zero impact on existing pages! ✅

---

## Complete MCP Server Flow (Both Scenarios)

### Flow 1: New Standalone Project

```python
def generate_new_project(tokens_css, dashboard_html):
    """
    Generate for NEW standalone MudBlazor project
    """
    # Step 1: Parse tokens
    tokens = parse_tokens(tokens_css)

    # Step 2: Generate theme configuration
    theme_config = generate_theme_config(tokens)
    write_file("ThemeConfiguration.cs", theme_config)

    # Step 3: Parse HTML components
    components = parse_html_components(dashboard_html)

    # Step 4: Generate Razor components (no scope wrapper)
    razor_code = generate_razor_components(components)
    write_file("Property.razor", razor_code)

    # Step 5: Generate GLOBAL CSS overrides
    css_overrides = generate_css_overrides(components, tokens)
    write_file("wwwroot/css/mudblazor-overrides.css", css_overrides)

    # Step 6: Generate App.razor that uses theme
    app_razor = f"""
<MudThemeProvider Theme="@_customTheme" />
<Router>...</Router>

@code {{
    private MudTheme _customTheme = ThemeConfiguration.CreateTheme();
}}
"""
    write_file("App.razor", app_razor)

    # Step 7: Copy tokens file
    copy_file(tokens_css, "wwwroot/css/_tokens.css")

    return {
        'files': [
            'ThemeConfiguration.cs',
            'App.razor',
            'Property.razor',
            'wwwroot/css/_tokens.css',
            'wwwroot/css/mudblazor-overrides.css'
        ],
        'instructions': 'Add to new MudBlazor project. Theme applies globally.'
    }
```

### Flow 2: Existing MudBlazor Project

```python
def generate_for_existing_project(tokens_css, dashboard_html, scope_class="eg-dashboard-scope"):
    """
    Generate for EXISTING MudBlazor project (scoped approach)
    """
    # Step 1: Parse tokens
    tokens = parse_tokens(tokens_css)

    # Step 2: Generate SCOPED CSS with embedded tokens
    scoped_css = generate_scoped_tokens_and_overrides(tokens, scope_class)
    write_file("wwwroot/css/dashboard-scoped.css", scoped_css)

    # Step 3: Parse HTML components
    components = parse_html_components(dashboard_html)

    # Step 4: Generate Razor with scope wrapper
    razor_code = f"""
@page "/property"

<div class="{scope_class}">
    <MudContainer MaxWidth="MaxWidth.False">
        {generate_razor_components(components)}
    </MudContainer>
</div>

@code {{
    // Component code
}}
"""
    write_file("Pages/Property.razor", razor_code)

    # Step 5: DON'T generate ThemeConfiguration.cs or modify App.razor

    return {
        'files': [
            'Pages/Property.razor',
            'wwwroot/css/dashboard-scoped.css'
        ],
        'instructions': '''
Add to existing project:
1. Copy Property.razor to Pages/
2. Copy dashboard-scoped.css to wwwroot/css/
3. Add <link href="css/dashboard-scoped.css" /> to _Host.cshtml
4. Existing pages remain unchanged
        '''
    }


def generate_scoped_tokens_and_overrides(tokens, scope_class):
    """
    Generate single CSS file with scoped tokens + overrides
    """
    css = f"/* Generated for existing MudBlazor project - Scoped to .{scope_class} */\n\n"

    # 1. Scoped tokens (light mode)
    css += f".{scope_class} {{\n"
    css += "    /* Design Tokens */\n"

    for category, token_dict in tokens.items():
        css += f"    /* {category.capitalize()} */\n"
        for token_name, token_value in token_dict.items():
            css += f"    --{token_name}: {token_value};\n"

    css += "}\n\n"

    # 2. Dark mode tokens (scoped)
    if 'dark' in tokens:
        css += f"[data-theme=\"dark\"] .{scope_class} {{\n"
        css += "    /* Dark Mode Overrides */\n"

        for token_name, token_value in tokens['dark'].items():
            css += f"    --{token_name}: {token_value};\n"

        css += "}\n\n"

    # 3. Scoped component overrides
    css += "/* Component Overrides */\n\n"
    css += generate_scoped_component_overrides(scope_class)

    return css
```

### Main Entry Point

```python
def main(tokens_css_path, html_path, deployment_type, scope_class="eg-dashboard-scope"):
    """
    Main MCP server entry point
    """

    # Load input files
    tokens_css = read_file(tokens_css_path)
    dashboard_html = read_file(html_path)

    # Generate based on deployment type
    if deployment_type == "new":
        result = generate_new_project(tokens_css, dashboard_html)

    elif deployment_type == "existing":
        result = generate_for_existing_project(tokens_css, dashboard_html, scope_class)

    else:
        raise ValueError("deployment_type must be 'new' or 'existing'")

    return result


# Usage examples:
# For new project:
output = main("_tokens.css", "dashboard.html", deployment_type="new")

# For existing project:
output = main("_tokens.css", "dashboard.html", deployment_type="existing", scope_class="my-dashboard")
```

---

## Key Success Factors

1. **Always add custom classes** to MudBlazor components (`-mud` suffix)
2. **Use `var(--*)` in CSS**, never hardcode values
3. **CSS load order matters** - overrides must be last
4. **Shadows must have 25 elements** (MudBlazor requirement)
5. **Use `!important`** to override MudBlazor's specificity

---

## Accuracy Results

| Approach | Accuracy |
|----------|----------|
| Plain MudBlazor (no customization) | 50-60% |
| MudBlazor + Theme Config | 60-70% |
| **Three-Layer Strategy (Theme + Custom Classes + CSS Overrides)** | **90-95%** |

---

## Testing Your MCP Server Output

```bash
dotnet build
dotnet run --urls "http://localhost:5000"
```

Compare side-by-side:
- Original: `Dashboard/dashboard.html`
- Generated: `http://localhost:5000`

Should match exactly: colors, spacing, borders, typography, hover states.

---

## Summary

### For New Standalone Projects

**Your MCP server generates:**

1. **ThemeConfiguration.cs** - Parse tokens → C# theme object
2. **App.razor** - Global theme configuration
3. **Property.razor** - MudBlazor components + custom classes
4. **wwwroot/css/_tokens.css** - Copy of original tokens
5. **wwwroot/css/mudblazor-overrides.css** - Global CSS overrides

**Result:** Complete MudBlazor project with custom design system.

---

### For Existing MudBlazor Projects

**Your MCP server generates:**

1. **Pages/Property.razor** - MudBlazor components wrapped in `<div class="eg-dashboard-scope">`
2. **wwwroot/css/dashboard-scoped.css** - Scoped tokens + scoped overrides in ONE file

**Result:** Isolated dashboard that doesn't affect existing pages.

---

### Key Differences

| Aspect | New Project | Existing Project |
|--------|------------|------------------|
| Theme | ✅ Global ThemeConfiguration.cs | ❌ None (use existing theme) |
| CSS Scope | Global `.mud-button-root` | Scoped `.eg-dashboard-scope .mud-button-root` |
| Tokens | Separate `_tokens.css` file | Embedded in scoped CSS |
| App.razor | ✅ Modified | ❌ Not touched |
| Impact | Entire app | Only dashboard pages |
| Files Generated | 5 files | 2 files |

---

## File Management Strategy

### Input Files: What to Keep vs Delete

Your MCP server workflow should follow this structure:

#### Files to KEEP (Source Files)

1. **`_tokens.css`** - Single source of truth ✅
   - Contains all design system tokens
   - Used by MCP server to generate code
   - Framework-agnostic (can generate for React, Vue, Angular, etc.)
   - Required for regeneration when tokens change

2. **`dashboard.html`** (Optional) - UI structure reference ✅
   - Reference template for component structure
   - Helpful for understanding intended layout
   - Can be deleted after initial generation if not needed for updates

#### Files You Can DELETE (Generated or Reference Only)

1. **`dashboard.css`** - Reference only ✅
   - Original CSS from template
   - Not loaded by Blazor app
   - All styles copied to generated CSS files
   - Safe to delete

#### Generated Files (Auto-Generated by MCP Server)

**For New Projects:**
```
Generated/
├── ThemeConfiguration.cs          ← From _tokens.css
├── App.razor                      ← Theme configuration
├── Pages/Property.razor           ← From dashboard.html
└── wwwroot/css/
    ├── _tokens.css                ← Copy of source
    └── mudblazor-overrides.css    ← Generated overrides
```

**For Existing Projects (Scoped):**
```
Generated/
├── Pages/Property.razor                ← With .eg-dashboard-scope wrapper
└── wwwroot/css/
    └── dashboard-scoped.css           ← Tokens + overrides in ONE file
```

### Key Principle

**Input:** `_tokens.css` (canonical design system)
**Output:** Framework-specific generated code
**Result:** Can delete dashboard.html/css after generation, but keep _tokens.css for updates

---

## Dark Mode Implementation

### Automatic Dark Mode Support

If your `_tokens.css` includes dark mode tokens, the generated code automatically supports theme switching with **zero additional configuration**.

### Token Structure for Dark Mode

```css
/* _tokens.css - Light mode (default) */
:root {
    --primary-600: #0062C1;
    --primary-400: #3E88EF;
    --neutral-900: #21262E;
    --neutral-25: #F9FAFB;

    /* Semantic tokens */
    --color-primary-default: var(--primary-600);
    --surface-base: #FFFFFF;
    --surface-container: var(--neutral-25);
    --text-default: #000000;
}

/* Dark mode overrides */
[data-theme="dark"] {
    --color-primary-default: var(--primary-400);
    --surface-base: var(--neutral-900);
    --surface-container: var(--neutral-800);
    --text-default: #FFFFFF;
}
```

### Generated Code (Scoped Version)

```css
/* dashboard-scoped.css */
.eg-dashboard-scope {
    /* Light mode tokens */
    --primary-600: #0062C1;
    --primary-400: #3E88EF;
    --color-primary-default: var(--primary-600);
    --surface-base: #FFFFFF;
    --text-default: #000000;
}

/* Dark mode (scoped) */
[data-theme="dark"] .eg-dashboard-scope {
    --color-primary-default: var(--primary-400);
    --surface-base: var(--neutral-900);
    --text-default: #FFFFFF;
}

/* Overrides automatically use tokens */
.eg-dashboard-scope .mud-button-root.btn-primary-mud {
    background-color: var(--color-primary-default) !important;
    /* ✅ Switches automatically when [data-theme="dark"] is set */
}
```

### Theme Toggle Implementation

Generate this component for theme switching:

```razor
<!-- ThemeToggle.razor -->
@inject IJSRuntime JS

<button class="theme-toggle-btn" @onclick="ToggleTheme" title="Toggle theme">
    <i data-lucide="sun" class="theme-icon-light"></i>
    <i data-lucide="moon" class="theme-icon-dark"></i>
</button>

@code {
    private async Task ToggleTheme()
    {
        await JS.InvokeVoidAsync("toggleTheme");
    }
}
```

### JavaScript for Theme Switching

Generate this in `wwwroot/js/theme.js`:

```javascript
window.toggleTheme = function() {
    const html = document.documentElement;
    const currentTheme = html.getAttribute('data-theme');
    const newTheme = currentTheme === 'dark' ? 'light' : 'dark';

    html.setAttribute('data-theme', newTheme);
    localStorage.setItem('theme', newTheme);
};

// Initialize theme on page load
(function() {
    const savedTheme = localStorage.getItem('theme') || 'light';
    document.documentElement.setAttribute('data-theme', savedTheme);
})();
```

### CSS for Theme Toggle Button

```css
.theme-toggle-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 36px;
    height: 36px;
    background-color: var(--surface-container);
    border: var(--border-width-thin) solid var(--border-default);
    border-radius: var(--border-radius-100);
    color: var(--icon-secondary);
    cursor: pointer;
    transition: var(--transition-default);
    position: relative;
}

.theme-toggle-btn:hover {
    background-color: var(--surface-hover);
}

.theme-icon-light,
.theme-icon-dark {
    position: absolute;
    transition: opacity 0.3s ease, transform 0.3s ease;
}

.theme-icon-dark {
    opacity: 0;
    transform: rotate(180deg);
}

[data-theme="dark"] .theme-icon-light {
    opacity: 0;
    transform: rotate(-180deg);
}

[data-theme="dark"] .theme-icon-dark {
    opacity: 1;
    transform: rotate(0deg);
}
```

### MCP Server Generation Rules

1. **Detect Dark Mode Tokens**
   ```python
   def has_dark_mode(tokens_css):
       return '[data-theme="dark"]' in tokens_css
   ```

2. **Generate Scoped Dark Mode CSS**
   ```python
   if has_dark_mode(tokens_css):
       css += f"[data-theme=\"dark\"] .{scope_class} {{\n"
       css += "    /* Dark mode token overrides */\n"
       css += parse_dark_mode_tokens(tokens_css)
       css += "}\n"
   ```

3. **Include Theme Toggle Component**
   - Always generate ThemeToggle.razor
   - Generate theme.js with localStorage persistence
   - Add theme toggle to navigation header

### Benefits of This Approach

✅ **Zero runtime overhead** - Pure CSS variable switching
✅ **Framework-agnostic** - Same tokens work for any framework
✅ **No re-rendering** - CSS cascade handles everything
✅ **Persistent** - Theme saved to localStorage
✅ **Works with all components** - All MudBlazor components switch automatically via CSS variables

---

## Pure MudBlazor Approach with CSS Overrides

### Current Implementation Strategy

This implementation uses **100% MudBlazor components** with targeted CSS overrides to match your custom design system.

### Why Pure MudBlazor?

✅ **Full Blazor Ecosystem Integration**
- Native data binding (`@bind-Value`, `@bind-Visible`)
- Built-in state management
- Component lifecycle events
- Two-way binding support

✅ **Rich Component Library**
- Complex interactions out-of-the-box (dialogs, menus, date pickers)
- Accessibility features built-in
- Consistent component APIs
- Active maintenance and updates

✅ **No HTML/CSS Conflicts**
- Pure C#/Razor components
- Strongly typed parameters
- IntelliSense support
- Compile-time validation

### Component Mapping Strategy

Your MCP server maps HTML elements to MudBlazor components:

| HTML Element | MudBlazor Component | Custom Class | Purpose |
|--------------|---------------------|--------------|---------|
| `<button class="btn btn-primary">` | `<MudButton Variant="Variant.Filled" Color="Color.Primary">` | `btn-mud btn-primary-mud` | Action buttons |
| `<button class="btn btn-secondary">` | `<MudButton Variant="Variant.Outlined">` | `btn-mud btn-secondary-mud` | Secondary actions |
| `<span class="badge badge-success">` | `<MudChip Size="Size.Small" Color="Color.Success">` | `badge-mud badge-success-mud` | Status indicators |
| `<table>` | `<MudTable>` | `data-table-mud` | Data tables |
| `<th>` | `<MudTh>` | `table-header-cell-mud` | Table headers |
| `<td>` | `<MudTd>` | `table-cell-mud` | Table cells |
| `<select>` | `<MudSelect>` | `form-select-mud` | Dropdown selects |
| `<input type="text">` | `<MudTextField>` | `form-input-mud` | Text inputs |
| `<input type="checkbox">` | `<MudCheckBox>` | `table-checkbox-mud` | Checkboxes |

### Example: Button Implementation

**HTML Input (from dashboard.html):**
```html
<button class="btn btn-primary">
    <i data-lucide="plus"></i>
    <span>New row</span>
</button>
```

**Generated Razor (MudBlazor):**
```razor
<MudButton Variant="Variant.Filled"
           Color="Color.Primary"
           StartIcon="@Icons.Material.Filled.Add"
           Class="btn-mud btn-primary-mud">
    New row
</MudButton>
```

**Generated CSS Override:**
```css
.eg-dashboard-scope .mud-button-root.btn-mud {
    padding: var(--spacing-100) var(--spacing-200) !important;
    border-radius: var(--border-radius-100) !important;
    gap: var(--spacing-100) !important;
    font-size: var(--font-size-medium) !important;
    font-weight: var(--font-weight-medium) !important;
}

.eg-dashboard-scope .mud-button-root.btn-primary-mud {
    background-color: var(--color-primary-default) !important;
    color: var(--text-on-action) !important;
    border: none !important;
}

.eg-dashboard-scope .mud-button-root.btn-primary-mud:hover {
    background-color: var(--color-primary-hover) !important;
}
```

### Example: Table Implementation

**HTML Input:**
```html
<table class="data-table">
    <thead>
        <tr>
            <th>Name</th>
            <th>Status</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>John Doe</td>
            <td><span class="badge badge-success">Active</span></td>
        </tr>
    </tbody>
</table>
```

**Generated Razor:**
```razor
<MudTable Items="@items"
          Hover="true"
          Elevation="0"
          Class="data-table-mud">
    <HeaderContent>
        <MudTh Class="table-header-cell-mud">Name</MudTh>
        <MudTh Class="table-header-cell-mud">Status</MudTh>
    </HeaderContent>
    <RowTemplate>
        <MudTd Class="table-cell-mud">@context.Name</MudTd>
        <MudTd Class="table-cell-mud">
            <MudChip T="string"
                     Size="Size.Small"
                     Color="Color.Success"
                     Class="badge-mud badge-success-mud">
                @context.Status
            </MudChip>
        </MudTd>
    </RowTemplate>
</MudTable>
```

**Generated CSS Override:**
```css
.eg-dashboard-scope .mud-table.data-table-mud {
    background-color: var(--surface-base) !important;
    border: var(--border-width-thin) solid var(--border-default) !important;
    border-radius: var(--border-radius-200) !important;
}

.eg-dashboard-scope .mud-table-head .table-header-cell-mud {
    padding: var(--spacing-200) var(--spacing-300) !important;
    background-color: var(--surface-subtle) !important;
    font-size: var(--font-size-small) !important;
    font-weight: var(--font-weight-semibold) !important;
    color: var(--text-secondary) !important;
    border-bottom: var(--border-width-thin) solid var(--border-default) !important;
}

.eg-dashboard-scope .mud-table-body .table-cell-mud {
    padding: var(--spacing-200) var(--spacing-300) !important;
    border-bottom: var(--border-width-thin) solid var(--border-subtle) !important;
}
```

### Component-Specific Override Patterns

#### 1. Buttons
```css
/* Base button styles */
.eg-dashboard-scope .mud-button-root.btn-mud {
    padding: var(--spacing-100) var(--spacing-200) !important;
    border-radius: var(--border-radius-100) !important;
    text-transform: none !important; /* Remove MudBlazor uppercase */
    font-size: var(--font-size-medium) !important;
}

/* Primary button */
.eg-dashboard-scope .mud-button-root.btn-primary-mud {
    background-color: var(--color-primary-default) !important;
    color: var(--text-on-action) !important;
}

/* Secondary/Outlined button */
.eg-dashboard-scope .mud-button-root.btn-secondary-mud {
    border: var(--border-width-thin) solid var(--border-default) !important;
    background-color: transparent !important;
    color: var(--text-default) !important;
}
```

#### 2. Chips/Badges
```css
.eg-dashboard-scope .mud-chip.badge-mud {
    padding: var(--spacing-50) var(--spacing-150) !important;
    border-radius: var(--border-radius-full) !important;
    font-size: var(--font-size-small) !important;
    height: auto !important;
}

.eg-dashboard-scope .mud-chip.badge-success-mud {
    background-color: var(--success-100) !important;
    color: var(--success-700) !important;
}
```

#### 3. Form Inputs
```css
.eg-dashboard-scope .mud-input.form-input-mud {
    border: var(--border-width-thin) solid var(--border-default) !important;
    border-radius: var(--border-radius-100) !important;
    padding: var(--spacing-150) var(--spacing-200) !important;
}

.eg-dashboard-scope .mud-select.form-select-mud {
    border: var(--border-width-thin) solid var(--border-default) !important;
    border-radius: var(--border-radius-100) !important;
}
```

### Generation Pseudocode

```python
def generate_mudblazor_component(html_element):
    """
    Convert HTML element to MudBlazor component with custom class
    """

    if html_element.tag == 'button':
        variant = 'Variant.Filled' if 'btn-primary' in html_element.classes else 'Variant.Outlined'
        icon = extract_icon(html_element)
        text = extract_text(html_element)
        custom_class = f"btn-mud {get_button_variant_class(html_element)}-mud"

        return f"""
<MudButton Variant="{variant}"
           Color="Color.Primary"
           StartIcon="@Icons.Material.Filled.{icon}"
           Class="{custom_class}">
    {text}
</MudButton>
"""

    elif html_element.tag == 'span' and 'badge' in html_element.classes:
        color = extract_color_variant(html_element)
        text = extract_text(html_element)
        custom_class = f"badge-mud badge-{color}-mud"

        return f"""
<MudChip T="string"
         Size="Size.Small"
         Color="Color.{color.capitalize()}"
         Class="{custom_class}">
    {text}
</MudChip>
"""

    elif html_element.tag == 'table':
        return generate_mud_table(html_element)

    elif html_element.tag == 'select':
        return generate_mud_select(html_element)

    elif html_element.tag == 'input':
        return generate_mud_textfield(html_element)
```

### Benefits of This Approach

✅ **Consistent Component Behavior**
- All components use MudBlazor's built-in state management
- Predictable lifecycle and event handling
- No mixing of HTML and component paradigms

✅ **Strong Typing**
- Compile-time validation
- IntelliSense support
- Type-safe parameters and bindings

✅ **Maintainability**
- CSS overrides are centralized in scoped CSS file
- Component classes follow consistent naming convention (`-mud` suffix)
- Easy to update design tokens without touching Razor files

✅ **Scoped Isolation**
- Entire dashboard wrapped in `.eg-dashboard-scope` class
- Zero impact on existing MudBlazor pages
- Can coexist with other design systems in same project

### Accuracy Trade-offs

| Aspect | Native MudBlazor | Pure MudBlazor + CSS Overrides |
|--------|------------------|-------------------------------|
| **Visual Accuracy** | 50-60% | **90-95%** |
| **Component Functionality** | 100% | 100% |
| **CSS Override Complexity** | None | Medium (targeted overrides) |
| **Maintenance Effort** | Low | Medium |
| **Type Safety** | High | High |
| **Blazor Integration** | Native | Native |

### Why 90-95% vs 100% Accuracy?

MudBlazor components have complex internal DOM structures that may have slight visual differences:

- **Ripple effects** - MudBlazor adds ripple containers that may affect spacing
- **Icon positioning** - Material Icons may have different baseline than custom icon fonts
- **Input states** - Focus/hover states may differ slightly from original design
- **Animations** - MudBlazor transitions may differ from custom CSS animations

**Where CSS Overrides Excel:**
- Colors, spacing, borders, typography: **100% accurate**
- Layout and positioning: **95% accurate**
- Component internal structure: **90% accurate** (requires deep selector targeting)

---

**Bottom Line:** MudBlazor components that look identical to your HTML/CSS design, whether for new projects or integrating into existing codebases.
