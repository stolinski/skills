---
name: motion-choreography-patterns
description: Use when orchestrating multi-element UI motion, stagger systems, list reorder/insert/remove flows, modal and overlay stacks, gesture-driven transitions, and route-level choreography that preserves hierarchy and attention.
---

# Motion Choreography Patterns

## Overview

This skill focuses on choreography: how multiple moving parts coordinate in time and space to communicate interaction intent.

Use it alongside foundational motion guidance to create motion systems that are legible, consistent, and production-ready across complex UI flows.

## When to Use

- Designing sequences with more than one animated element
- Building staggered reveals, list transitions, and layout mode changes
- Implementing modal, drawer, popover, and overlay interaction stacks
- Handling add/remove/reorder scenarios in dense interfaces
- Mapping user input (tap, drag, scroll) to motion responses
- Auditing whether motion hierarchy matches product hierarchy

## Choreography Model

Treat each sequence as three actor layers:

1. **Primary actor** - the element representing the state change
2. **Supporting actors** - nearby elements that reinforce context
3. **Environment actors** - backdrop, scrim, container, page-level continuity

Animate primary first, then supporting, then environment unless the interaction model requires the reverse.

## Timing Architecture

Use predictable beat structure:

- **Lead beat** (`0-80ms`) - immediate acknowledgment
- **Primary beat** (`120-240ms`) - main state change
- **Follow beat** (`20-60ms` offset) - supporting context update
- **Settle beat** (`80-180ms`) - final stabilization

Favor asymmetry: exits slightly faster than enters.

## Core Choreography Principles

- One dominant axis per beat (avoid conflicting direction signals)
- Keep simultaneous high-amplitude motions limited
- Use stagger only when it improves scan order and hierarchy
- Keep rhythm consistent across similar components
- Preserve spatial anchors during reflow/reorder transitions

## Pattern Library

For concrete recipes and timelines, see:

- [references/CHOREOGRAPHY_PATTERNS.md](references/CHOREOGRAPHY_PATTERNS.md)
- [references/INTERACTION_SCENARIOS.md](references/INTERACTION_SCENARIOS.md)

For reusable stagger utilities, see:

- [references/STAGGER_SYSTEMS.css](references/STAGGER_SYSTEMS.css)

## Input-to-Motion Mapping

- **Tap/click:** immediate visual response in under `80ms`
- **Press/hold:** include press state before commitment transition
- **Drag:** motion follows pointer directly; settle animation occurs only on drop/commit
- **Keyboard navigation:** preserve focus continuity and avoid disorienting travel

## Reduced Motion Choreography

When reduced motion is requested:

- Remove spatial travel when possible
- Keep sequencing logic with opacity/state changes
- Preserve hierarchy with contrast, layering, and timing rather than movement distance
- Ensure state change remains obvious and immediate

## Anti-Patterns

- Cascades that delay interaction completion
- Independent components using unrelated easing and duration tokens
- Large travel distances for frequent interactions
- Simultaneous enter and exit motions that compete for focus
- Decorative infinite animations in core task surfaces

## Output Contract

When using this skill, return these five sections:

1. **Storyboard** - who moves, in what order, and why
2. **Timeline Spec** - durations, offsets, easing tokens, and distances
3. **Implementation Plan** - CSS/JS/View Transition structure
4. **A11y + Fallback** - reduced-motion and unsupported-feature strategy
5. **Validation Plan** - performance and UX checks

## QA Checklist

Use this before shipping:

- [references/QA_STORYBOARD_CHECKLIST.md](references/QA_STORYBOARD_CHECKLIST.md)
