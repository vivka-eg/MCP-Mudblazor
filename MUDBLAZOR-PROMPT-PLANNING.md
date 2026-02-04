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
11. [Hybrid Approach](#hybrid-approach-plain-html-vs-mudblazor-components)
12. [Testing & Results](#testing-your-mcp-server-output)

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
- **Hybrid Approach**: Plain HTML for visuals, MudBlazor for complex interactions
- **Automatic Dark Mode**: CSS variables switch themes without re-rendering

### What This Guide Covers

✅ Complete code generation workflow (tokens → Blazor app)
✅ Two deployment scenarios (new vs existing projects)
✅ File management strategy (what to keep/delete)
✅ Automatic dark mode implementation
✅ Decision tree for plain HTML vs MudBlazor components
✅ Testing and validation approach

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

**Accuracy:** 95-100%

### For Existing MudBlazor Projects

**Input:**
- `_tokens.css` - Design system tokens
- `dashboard.html` - UI structure reference

**Output:**
- `Pages/Property.razor` - Scoped page components
- `wwwroot/css/dashboard-scoped.css` - Scoped tokens + overrides

**Accuracy:** 95-100% (existing pages unaffected)

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
| Plain MudBlazor | 50-60% |
| MudBlazor + Theme | 60-70% |
| **Three-Layer Strategy** | **95-100%** |

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
✅ **Works with all components** - Both plain HTML and MudBlazor components switch automatically

---

## Hybrid Approach: Plain HTML vs MudBlazor Components

### When to Use Plain HTML vs MudBlazor Components

Your MCP server should follow this decision tree:

#### Use Plain HTML When:

✅ **Visual/Static Components**
- Buttons (basic click actions)
- Badges/status indicators
- Tables (simple data display)
- Cards
- Typography
- Icons

**Why:** 100% accurate to design, no CSS override conflicts

```razor
<!-- Plain HTML - Perfect accuracy -->
<button class="btn btn-primary" @onclick="Save">
    <i data-lucide="plus"></i>
    <span>New row</span>
</button>

<span class="badge badge-success">Active</span>

<table class="data-table">
    <thead>
        <tr>
            <th>Name</th>
            <th>Status</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var item in items)
        {
            <tr>
                <td>@item.Name</td>
                <td>@item.Status</td>
            </tr>
        }
    </tbody>
</table>
```

#### Use MudBlazor Components When:

✅ **Complex Interactive Components**
- Dialogs/Modals (overlay management, focus trap, animations)
- Date/Time Pickers (calendar logic, date validation)
- Autocomplete (search, dropdown, keyboard nav)
- Tooltips/Popovers (positioning, z-index management)
- Dropdowns/Menus (click outside, keyboard nav)
- File Uploads (drag-drop, progress)

**Why:** Complex behavior that's time-consuming to build from scratch

```razor
<!-- Complex behavior - Use MudBlazor -->
<MudDialog @bind-Visible="_showDialog" Options="_dialogOptions">
    <TitleContent>
        <h2 class="modal-title">Add Agent Roles</h2>
    </TitleContent>
    <DialogContent>
        <!-- Plain HTML inside for styling accuracy -->
        <div class="form-group">
            <label class="form-label">Agent</label>
            <select class="form-select">
                <option>Agent 1</option>
            </select>
        </div>
    </DialogContent>
    <DialogActions>
        <button class="btn btn-ghost" @onclick="Close">Cancel</button>
        <button class="btn btn-primary" @onclick="Save">Save</button>
    </DialogActions>
</MudDialog>

<MudDatePicker @bind-Date="_selectedDate" />

<MudAutocomplete T="string"
                 SearchFunc="@SearchAgents"
                 Label="Search agents" />
```

### Generation Strategy

```python
def generate_component(html_element):
    """
    Decide whether to use plain HTML or MudBlazor component
    """

    # Visual components → Plain HTML
    if html_element.tag == 'button' and not has_complex_behavior(html_element):
        return generate_plain_button(html_element)

    if html_element.tag == 'span' and 'badge' in html_element.classes:
        return generate_plain_badge(html_element)

    if html_element.tag == 'table':
        return generate_plain_table(html_element)

    # Complex components → MudBlazor (with plain HTML content)
    if is_modal(html_element):
        return generate_mud_dialog_with_html_content(html_element)

    if is_date_picker(html_element):
        return '<MudDatePicker @bind-Date="_date" />'

    if is_autocomplete(html_element):
        return '<MudAutocomplete T="string" SearchFunc="..." />'
```

### Best Practice: Nest Plain HTML Inside MudBlazor

```razor
<!-- ✅ GOOD: MudBlazor for behavior, HTML for styling -->
<MudDialog @bind-Visible="_showDialog">
    <DialogContent>
        <!-- Plain HTML follows your design tokens exactly -->
        <div class="modal-body">
            <div class="form-group">
                <label class="form-label">Name</label>
                <input type="text" class="form-input" @bind="_name" />
            </div>
            <button class="btn btn-primary" @onclick="Submit">
                Submit
            </button>
        </div>
    </DialogContent>
</MudDialog>

<!-- ❌ BAD: MudBlazor components for form inputs -->
<MudDialog @bind-Visible="_showDialog">
    <DialogContent>
        <MudTextField @bind-Value="_name" Label="Name" />
        <MudButton Color="Color.Primary" OnClick="Submit">Submit</MudButton>
        <!-- Now you need CSS overrides for EVERYTHING -->
    </DialogContent>
</MudDialog>
```

### Accuracy Comparison

| Approach | Accuracy | Effort | Maintenance |
|----------|----------|--------|-------------|
| Plain HTML everywhere | 100% | Medium | Low |
| MudBlazor everywhere | 60-75% | Low | High (overrides) |
| **Hybrid (Plain + MudBlazor)** | **95-100%** | **Low** | **Low** |

### MCP Server Prompt Decision

When generating code, ask the user:

```
? Component generation strategy?
  › Hybrid (Plain HTML for visuals, MudBlazor for complex interactions) - Recommended
    Plain HTML only (requires manual implementation of dialogs, date pickers, etc.)
    MudBlazor only (requires extensive CSS overrides, ~70% accuracy)
```

---

**Bottom Line:** MudBlazor components that look identical to your HTML/CSS design, whether for new projects or integrating into existing codebases.
