/**
 * @project Abeto.co - Ponyo Color System
 * @version 1.0.0
 * @date 2025-10-04
 * @author Abeto.co Design Team
 * @description Official documentation for the Ponyo-inspired color system. This document provides specifications for color palettes, SCSS variables, and usage guidelines to ensure a consistent, accessible, and thematic user interface.
 */

# Abeto.co Color System Documentation

---

## 1.0 Overview

### 1.1 Philosophy

This color system is inspired by the whimsical aesthetic of Studio Ghibli's "Ponyo on the Cliff by the Sea." The palette is built on the narrative duality between the magical, energetic **Sea (Ponyo)** and the warm, stable **Land (Sōsuke)**. This distinction forms the core of our UI's visual language, creating an intuitive user experience.

### 1.2 Core Principles

* **Thematic & Whimsical**: Colors are chosen to evoke the hand-painted, natural feeling of the film.
* **Accessibility First**: All primary color combinations are designed to meet or exceed WCAG AA standards.
* **Clear Hierarchy**: Colors have defined roles. Ponyo's palette is for **actions**, and Sōsuke's is for **information**.
* **Maintainable**: The system is built on a structured set of SCSS variables for easy implementation and maintenance.

---

## 2.0 Palette Specification

### 2.1 Light Theme (Land & Shore) 🎨

The light theme is designed to feel airy, warm, and inviting, like Sōsuke's house on the cliff overlooking the sea.

| Role | Color Name | Hex | SCSS Variable |
| :--- | :--- | :--- | :--- |
| **Background** | Warm Sand | `#F9F7F5` | `$ouiPageBackgroundColor` |
| **Primary Text** | Driftwood Brown | `#3D352E` | `$ouiColorDarkestShade` |
| **Primary Action** | Deep Sea Blue | `#2B546A` | `$ouiColorPrimary` |
| **CTA / Accent** | Ponyo's Coral | `#F9627D` | `$ouiColorAccent` |
| **Info Surface** | Sōsuke's Yellow | `#F7B538` | `$ouiColorWarning` |
| **Grounding Surface** | Cliffside Green | `#5F826E` | `$ouiColorSecondary` |
| **Borders**| Pebble Gray| `#D1CFCB`| `$ouiColorLightShade`|

### 2.2 Dark Theme (Deep Sea) 🌃

The dark theme is moody and magical, inspired by the depths of the ocean. Colors are made more vibrant to ensure high contrast and energy.

| Role | Color Name | Hex | SCSS Variable |
| :--- | :--- | :--- | :--- |
| **Background** | Deepest Sea | `#11202C` | `$ouiPageBackgroundColor`|
| **Primary Text** | Ocean Foam | `#F5F5F3` | `$ouiColorFullShade` |
| **Primary Action** | Magic Blue | `#36A2EB` | `$ouiColorPrimary` |
| **CTA / Accent** | Ponyo's Coral | `#F9627D` | `$ouiColorAccent` |
| **Info Surface** | Sōsuke's Yellow | `#F7B538` | `$ouiColorWarning` |
| **Grounding Surface** | Magic Seafoam | `#4ECFA2` | `$ouiColorSecondary` |
| **Card Surface**| Deep Sea Ink| `#182C3B`| `$ouiColorEmptyShade`|

---

## 3.0 Implementation & Usage Directives

### 3.1 Accessibility Directive: CTA Contrast

To maintain WCAG AA compliance, a specific rule must be followed for the primary accent color.

* **Rule:** Any button or interactive surface using `Ponyo's Coral` (`$ouiColorAccent`) as a background **MUST** use a dark text color, such as `Driftwood Brown` (`$ouiColorDarkestShade` in the light theme).

### 3.2 Advanced Effects: Shadows & Transparency

Standard palette colors are opaque. To create natural-looking effects like shadows, transparent versions of the palette colors must be used.

* **Best Practice:** Derive transparent colors directly from the core SCSS variables to maintain system consistency.

* **Example Implementation:**
    ```scss
    // Changed the mixin for the effect
    @mixin ouiSlightShadowHover($color: $ouiShadowColor, $opacity: .3) {
      box-shadow:
         2px 8px 20px -3px rgba($ouiColorHighlight, 0.5),
         -2px -1px 8px -4px rgba($ouiColorHighlight, 1);
      }
    ```

### 3.2 Smooth border on panel overflow

When you add too much content to the panel, as in the screenshot below, it tends to cut off the content abruptly.

// TODO: Add panel screenshot without "shadow"

To prevent this, I would add a "box shadow". Since seems like its a plugin, and not the code from the oui lib, I added a pseudo element with background from the theme into the [visualization class](src/plugins/visualizations/public/components/_visualization.scss).

```scss
  &::after {
    content: '';
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    height: 20px;
    background: linear-gradient(
      to bottom, 
      rgba($euiPageBackgroundColor, 0),
      rgba($euiPageBackgroundColor, 1)
    );
    pointer-events: none;
  }
```

// TODO: Add panel screenshot with "shadow"

---

## 4.0 Known Issues & Limitations

### 4.1 Build System Challenges

When integrating this customized OUI fork with OpenSearch Dashboards, several technical challenges may arise:

#### 4.1.1 TypeScript Configuration Requirements

**Issue:** The OUI build process may fail due to TypeScript type conflicts, particularly with Babel type definitions (`@types/babel__traverse`) showing syntax errors like `TS1005: '?' expected`.

**IA Suggested Workaround:** Add `"skipLibCheck": true` to the `compilerOptions` section of `tsconfig.json` in both the OUI fork and OpenSearch Dashboards repositories. This bypasses third-party type definition checking while maintaining type safety for your own code.

**Conclusion:** Since this compiler option its beeing used around the lib and for the test I've decided to keep it.

#### 4.1.2 Dependency Version Conflicts

**Issue:** Multiple version ranges for dependencies (particularly `@elastic/eui` vs `@opensearch-project/oui`) may be declared across different `package.json` files in OpenSearch Dashboards, causing bootstrap failures.

**Resolution:** All `package.json` files must reference the same version. Use the following command to update all references:
```bash
find . -type f -name package.json -exec sed -i '' 's/"@opensearch-project\/oui": ".*"/"@elastic\/eui": "file:\/path\/to\/oui-fork"/g' {} \;
```

Pay special attention to `@osd/ui-framework` and `@osd/ui-shared-deps` packages, which may require manual updates.

**Conclusion:** I think at this point claude was lost cause he didn't had all context. Happily, all you need to do is use the standard command  `find . -type f -name package.json -exec sed -i '' 's/"@elastic\/eui": ".*"/"@elastic\/eui": "file:\/Users\/larissajusten\/Documents\/oui"/g' {} \;` from the documentation, which worked well.

#### 4.1.3 React Type Definition Incompatibilities

**Issue:** During webpack compilation, you may encounter errors about React components not being valid JSX elements (e.g., `AutosizeInput`, `FixedSizeList`, `DragDropContext` showing "Property 'refs' is missing").

**Cause:** Conflicting React type definitions from different packages (particularly from `@types/enzyme` and the main `@types/react`).

**Mitigation:** The `skipLibCheck: true` setting addresses this issue by preventing TypeScript from strictly validating third-party type definitions.

**Conclusion:** Here was a problem with the package and libs installation. Doing the cleaning with `rm -rf node_modules && yarn osd clean` and then installing everything again with `yarn osd bootstrap` worked well.

#### 4.1.4 Missing Babel Dependencies

**Issue:** Initial builds may fail with `Cannot find module '@babel/core'` errors during the `extract-i18n-strings` step.

**Resolution:** Ensure all dependencies are properly installed:
```bash
yarn install
# If issues persist, explicitly install Babel dependencies:
yarn add --dev @babel/core @babel/cli
```

### 4.2 Integration Best Practices

#### 4.2.1 Local Development Setup

When linking a local OUI fork to OpenSearch Dashboards:

1. **Build OUI first** with all modifications in place
2. **Disable rebuild scripts** in OUI's `package.json` to prevent OpenSearch Dashboards from attempting to rebuild:
   ```json
   "build": "echo 'Build already completed, skipping'",
   "prepare": "echo 'Skipping prepare'"
   ```
3. Clean OpenSearch Dashboards completely before linking:
   ```bash
   rm -rf node_modules yarn.lock
   ```

### 4.3 Limitations

* **Type Safety Trade-offs:** Using `skipLibCheck` reduces type safety for third-party libraries. Ensure thorough testing of component integrations.
* **Bootstrap Complexity:** The OpenSearch Dashboards bootstrap process is sensitive to dependency versions. Fresh installs are recommended when switching OUI versions.
* **Git Workflow:** If working from a cloned repository without forking first, changes must be committed locally before creating a fork and pushing to your own remote.

### 4.4 Bug

#### 4.4.1 An Infinite Loop in a Function

> When you use two colors that are far too similar in the function `makeHighContrastColor` it creates a loop that don't let your lib finish the build, and doesn't show any error. 

By isolating the problem within the HEX color, we can find not only the source of the problem, but also the path. Knowing that the color was `#5A6875` ($ouiColorLightShade) from the Ponyo palette, we found all the places where it was used. By adding a debug to the only related @while function, we also found the source.

// TODO: Add photo console (debug)

`foreground #798189` and `background #6d7a85`: This is the root cause. These two colors are both dark mid-grays with very similar lightness levels. The contrast ratio between them is extremely low to begin with (around 1.15:1).

**The Culprit:**
```scss
$ouiBreadcrumbInactiveBackground: tint($ouiColorLightShade, 14%);
```

- The function `tint()` mixes a color with white.

- It's taking your `$ouiColorLightShade` (which is `#5A6875`) and mixing it with 14% white.

When Sass performs this calculation, the resulting color is #6C7882. This is almost identical to the mystery background color #6d7a85 from the debug log. The tiny difference is likely due to rounding in different color model calculations.

This is like asking a chef to make a vibrant, colorful meal using only two slightly different shades of gray potato. No matter how much they try to tweak the recipe, they can't create a contrast that doesn't exist in the source ingredients.

**The resolution:** A good replacement that fits the theme and solves the contrast problem would be `#D6D9DD`.


---

## 5.0 Changelog

* **v1.0.0** (2025-10-04) - Initial release of the Ponyo Color System documentation.