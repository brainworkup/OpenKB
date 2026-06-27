---
sources: [summaries/top_level.md]
brief: Matplotlib is a Python data visualization library that uses cycler for automatic plot style cycling.
---

# Matplotlib

Matplotlib is a foundational Python library for creating static, animated, and interactive visualizations. It is one of the most widely used plotting frameworks in the scientific Python ecosystem.

## Overview

Matplotlib provides a MATLAB-like interface for generating plots, charts, and figures. It supports a wide range of plot types and offers fine-grained control over visual properties.

## Relationship to Cycler

One of Matplotlib's key dependencies is the `cycler` library (see [[summaries/top_level]]), which powers automatic style cycling in plots. When multiple data series are plotted, Matplotlib uses `cycler` to automatically cycle through visual properties such as:

- **Colors** — rotating through a defined color palette
- **Line styles** — dashed, dotted, solid, etc.
- **Markers** — circles, squares, triangles, etc.

This allows users to distinguish multiple series without manually specifying styles for each one.

## Style and Theming

Matplotlib's style system integrates with the `cycler` library to allow composable, reusable style definitions. Property cycles can be combined using set-product operations, enabling complex multi-property cycling.

This theming capability is related to broader concepts in [[concepts/brand-color-system]] and [[concepts/r-visualization-theming]], which address how visual identity is managed across plotting and reporting tools.

## Related Concepts

- [[concepts/iterators]] — The underlying programming pattern that cycler builds upon
- [[concepts/brand-color-system]] — Color palette design relevant to plot styling
- [[concepts/r-visualization-theming]] — Parallel visualization theming in the R ecosystem
- [[summaries/top_level]] — Source document referencing the cycler library
