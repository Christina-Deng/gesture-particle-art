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
3. Discrete gestures shift temperament freely among Void / Ember / Ink (not a one-way ladder).  
4. A large sweep **splashes** live particles into a semi-permanent **stain** layer.  
5. A held pinch dragged across / toward the body **recalls** stain back into living particles.  
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

| Gesture | Disambiguation | Effect |
|---------|----------------|--------|
| Quick single-hand open (ignite) | Fast openness spike, little translation | Enter **Ember** |
| Slow pinch then slight open (dip ink) | Slow close→micro-open, hand mostly still | Enter **Ink** |
| Large wide horizontal sweep | High lateral velocity, open or semi-open hand(s) | **Splash** live → stain |
| Held pinch + drag | Pinch sustained while hand translates | **Recall** stain → live |
| Both hands raised open (optional) | Two hands high + open | Soft reset toward **Void** |

Pinch alone does nothing mode-changing; **Ink** needs slow pinch-release in place, **Recall** needs pinch **plus** drag. Implementation should use hysteresis / cooldowns so ignite and dip-ink do not false-trigger during normal play.

Gesture philosophy: continuous free influence (B), plus as many flourishes as fit without drowning the few clear mode actions (lean C).

Particle budget: start ~80k–120k points, adaptive down on frame-time pressure so mid laptops hold interactive feel.

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
