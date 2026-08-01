# ⚙️ DevState — Visual Finite State Machine (FSM) Automata Architect (Vue 3)

DevState is a reactive visual finite state machine architect engineered with Vue 3 (Composition API `<script setup>`). It implements a state machine evaluation engine that maps node coordinates onto dynamic vector canvas maps (`<svg>`), processes state transitions deterministically, and compiles visual workflows into production-ready JSON schemas.

## ⚡ Key Architecture Concepts
* 🧪 **Vue 3 Reactivity Engine:** Uses Vue's reactive `ref` and `computed` primitives to calculate valid transition triggers on the fly based on active state pointers.
* 📐 **Vector Coordinate Math:** Dynamically calculates center-point SVG line connectors (`x1, y1` to `x2, y2`) between reactive node coordinates.

## ⚙️ Running Instructions
1. Install dependencies: `npm install`
2. Launch dev workspace: `npm run dev`