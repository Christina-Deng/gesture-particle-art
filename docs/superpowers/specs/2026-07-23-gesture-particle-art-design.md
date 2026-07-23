# Gesture Particle Art — Design Spec

**Date:** 2026-07-23  
**Status:** Approved for planning  
**Project:** `/Users/christinadeng/Projects/gesture-particle-art`

## Goal

A pure artistic, full-screen particle experience controlled by **webcam hand tracking**. No product UI, no content site — a visual performance that moves through three temperaments and can splash / recall ink onto a background layer.

Look targets (generated stills, approximate in real time — do not pixel-match):

- Void: near-black, cool white star-dust particles  
- Ember: deep warm field, orange / gold / crimson embers  
- Ink: pale paper ground, living sumi-e black mist  
- Splash: horizontal flung ink stained onto the surface, some particles still airborne  

## Experience Flow

1. Near-black void with slow white/cool-gray particles. Minimal camera-permission prompt; fades after grant.  
2. Hands continuously influence the field (position, velocity, openness).  
3. Discrete gestures shift temperament: Void → Ember → Ink.  
4. A large sweep **splashes** live particles into a semi-permanent **stain** layer.  
5. A pinch-and-pull **recalls** stain back into living particles.  
6. Optional soft reset returns toward Void without a hard cut.

## Interaction Model

### Continuous (always on)

| Signal | Effect |
|--------|--------|
| Hand position | Attract / steer nearby particles (mirrored selfie space) |
| Hand velocity | Turbulence + motion trails |
| Palm openness | Spread ↔ gather density |
| Two hands | Two overlapping force fields |

Lost tracking: force fields decay smoothly over ~0.5s (no hard stop).

### Discrete gestures

| Gesture | Effect |
|---------|--------|
| Quick single-hand open (ignite) | Enter **Ember** |
| Slow pinch then slight open (dip ink) | Enter **Ink** |
| Large two-hand / wide horizontal sweep | **Splash** live particles → stain layer |
| Pinch and pull back | **Recall** stain → live particles |
| Both hands raised open (optional) | Soft reset toward **Void** |

Gesture philosophy: continuous free influence (B), plus as many flourishes as fit without drowning the few clear mode actions (lean C).

## Visual Temperaments

| State | Atmosphere | Particles | Motion |
|-------|-------------|-----------|--------|
| **Void** | Near black | Small cool white / gray | Slow drift, soft trails |
| **Ember** | Deep warm black-brown | Orange, gold, crimson | Faster turbulence, light bloom |
| **Ink** | Off-white / rice-paper | Black / charcoal clusters | Brush-like clumps, restrained |

**Splash** is not a fourth temperament — it is an overlay action available especially in Ink (and usable from other states). Stain edges may slightly bleed; recall peels stain back into motion.

## Architecture

### Render layers (back → front)

1. **Atmosphere** — ground color / subtle wash per temperament  
2. **Stain** — 2D canvas; receives splash stamps; erased/absorbed on recall  
3. **Live particles** — WebGL points / custom shaders; physics + hand forces  

Camera feed is hidden by default (optional tiny corner preview, off by default) to avoid a “surveillance” feel.

### Data flow

```
Webcam → MediaPipe Hands (landmarks)
      → Gesture parser (openness, speed, sweep, pinch)
      → Force fields + temperament state machine
      → Particle simulation
      → WebGL render
      ↳ on splash: sample live → write Stain
      ↳ on recall: sample Stain → re-seed / attract live
```

### Tech stack

- **Vite + TypeScript** (no heavy app framework)  
- **`@mediapipe/tasks-vision`** for hand landmarks  
- **Three.js** Points (or equivalent WebGL) for particles  
- Small state machine: `void | ember | ink`; splash/recall as actions  

## Scope

### In (v1)

- Desktop Chrome / Safari first  
- Full-screen art experience with the interactions above  
- ~60fps feel on mid-range laptops  
- Playable within ~10s of camera grant  

### Out (v1)

- Accounts, sharing, multi-page marketing copy  
- Mobile-first hand-feel guarantees  
- Audio / music  
- Multi-person tracking, export / save artwork  
- Photoreal still-frame lighting parity with look-target images  

## Success Criteria

- Camera granted → readable play within 10 seconds  
- Temperament changes and splash/recall are obvious without instruction beyond a one-line hint  
- Sustained interactive frame feel (~60fps target) on a mid laptop  
- Visual temperament reads as Void / Ember / Ink relative to the look targets  

## Non-goals / honesty bar

Real-time particles approximate cinematic stills. Close the gap with bloom, trails, tone mapping, layered stain, and motion design — not by chasing photographic path tracing in the browser.
