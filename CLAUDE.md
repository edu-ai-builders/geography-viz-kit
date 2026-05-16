# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A suite of single-file HTML interactive 3D geography and astronomy visualizations for Chinese K-12 through university education. Each module is a fully self-contained HTML file that loads Three.js from CDN — no build step, no package manager.

See `知识体系.md` for the complete Chinese curriculum knowledge map (all 12 modules, grade levels, misconceptions, formulas, cognitive conflict triggers).

## Module Status

| File | Status | Topic |
|------|--------|-------|
| `index.html` | Complete | Landing page (card grid) |
| `01-earth-rotation.html` | Complete | Earth rotation: local time, time zones, terminator, Coriolis |
| `02-earth-revolution.html` | Complete | Earth revolution: axial tilt, seasons, elliptical orbit, declination curve |
| `03-solar-altitude.html` | Complete | Solar noon altitude H = 90° − \|φ − δ\|; ArrowHelper, angle arc, 2D curve |
| `04-shadow.html` | Complete | Shadow direction and length (L = h/tan(H)); flat ground scene with shadows |
| `05-solar-path.html` | Complete | Solar path diagrams; inverted celestial dome, 4 arc lines, moving sun |
| `06-day-length.html` | Complete | Day length variation; heat map (lat×doy), cosH₀ formula |
| `07-atmospheric-circulation.html` | Complete | Pressure belts & wind bands; Coriolis toggle, seasonal shift |
| `08-ocean-currents.html` | Complete | Ocean currents; CatmullRom flowlines, animated particles, fishing grounds |
| `09-celestial-coordinates.html` | Complete | Equatorial coordinates; RA/Dec grid, ecliptic, analemma, star catalog |
| `10-kepler-orbits.html` | Complete | Kepler's laws; adjustable ellipse, equal-area sweep, T²∝a³ table |
| `11-solar-system-scale.html` | Complete | Solar system scale; log/linear toggle, all 8 planets, orbital animation |
| `12-magnetosphere.html` | Complete | Magnetosphere; dipole field lines, solar wind particles, aurora oval, Kp slider |

## Architecture

**All modules share the same design system:**
- CSS custom properties: `--bg-deep: #070a13`, `--accent-warm: #f4a853`, `--accent-hot: #e85d4a`, `--accent-cool: #6db3d9`, etc.
- Fonts: Fraunces (display/headings), Noto Sans SC (UI text), JetBrains Mono (data/labels/code)
- Layout: `grid-template-columns: 340px 1fr 280px` — left control panel / center Three.js canvas / right data panel
- Three.js locked at r160 via importmap from `cdn.jsdelivr.net/npm/three@0.160.0`
- Renderer: `setClearColor(0x050811, 1)` — NOT `alpha: true` (bloom requires opaque framebuffer)
- All modules include: `EffectComposer` + `UnrealBloomPass` + `OutputPass`

## Shared Earth Render Block v2

Modules that show Earth contain a block delimited by:
```
// ╔══ BEGIN: SHARED EARTH RENDER BLOCK v2 ══╗
// ╚══ END: SHARED EARTH RENDER BLOCK v2   ══╝
```
**Make changes to this block in `01-earth-rotation.html` first, then copy the block verbatim to all other Earth modules.**

### Five textures (all CDN, no auth, each loads independently with placeholder fallback)

| Role | URL |
|------|-----|
| Day map | `https://cdn.jsdelivr.net/gh/mrdoob/three.js@r160/examples/textures/planets/earth_atmos_2048.jpg` |
| Specular (ocean mask) | `https://cdn.jsdelivr.net/gh/mrdoob/three.js@r160/examples/textures/planets/earth_specular_2048.jpg` |
| Normal map | `https://cdn.jsdelivr.net/gh/mrdoob/three.js@r160/examples/textures/planets/earth_normal_2048.jpg` |
| Night lights (city glow) | `https://cdn.jsdelivr.net/gh/vasturiano/three-globe@2.31.1/example/img/earth-night.jpg` |
| Cloud layer (5 MB PNG, slow) | `https://raw.githubusercontent.com/turban/webgl-earth/master/images/fair_clouds_4k.png` |

### Shader uniforms

`uDayMap`, `uSpecMap`, `uNightMap`, `uNormalMap`, `uSunDir` (vec3) + float guards `uHasTex`, `uHasSpec`, `uHasNight`, `uHasNorm`. Each texture's `onLoad` sets its guard flag to 1.0 — no blocking.

### Normal map TBN computation

In world space. Pole guard prevents artifacts at `|N.y| > 0.97`:
```glsl
float poleGuard = 1.0 - smoothstep(0.97, 0.999, abs(N.y));
N = normalize(mix(N, mat3(T, B, N) * mapN, poleGuard));
```

### Night lights blend

```glsl
float nightF = 1.0 - smoothstep(-0.25, 0.05, lighting);  // feathers through dusk band
vec3 nightLights = texture2D(uNightMap, uv).rgb;
nightLights = nightLights * nightLights * 2.8;  // gamma contrast boost
```

### Cloud mesh

`MeshLambertMaterial` at `EARTH_RADIUS * 1.012`, same texture for `map` and `alphaMap`. In animate loop: `cloudMesh.rotation.y += dt * 0.00008`.

### Bloom

```javascript
const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));
const bloomPass = new UnrealBloomPass(new THREE.Vector2(w, h), 0.35, 0.5, 0.88);
composer.addPass(bloomPass);
composer.addPass(new OutputPass());
// In animate: composer.render() then labelRenderer.render(scene, camera)
// In resize: composer.setSize(w, h); bloomPass.resolution.set(w, h);
```
`devicePixelRatio` capped at 1.5. `CSS2DRenderer` overlay is unaffected by bloom.

## Module 02 Two-Group Architecture

Earth revolution uses a two-group pattern for correct axial tilt:

```
earthTiltGroup (world space)
  position = computed orbit position
  rotation = setRotationFromAxisAngle(TILT_AXIS, -tiltRad)  // fixed in world space
  └─ earthSpinGroup
       rotation.y = spinAngle  // day/night cycle
       ├─ earthMesh (shared Earth block)
       ├─ atmo, cloudMesh
       └─ tropicsGroup
  ├─ axisLine, npLabel  (tilt but don't spin)
  └─ terminatorHolder   (rotated to face sun in local frame)
```

`TILT_AXIS = new THREE.Vector3(0, 0, 1)` — Z axis, fixed in world space. This ensures the north pole always leans in the same world direction regardless of orbital position, which is what produces seasons.

## Coordinate Convention (critical — inconsistency causes longitude offsets)

- Longitude 0° (prime meridian) = world **+X** axis
- Longitude 90°E = world **+Z** axis  
- City/point positions: `x = R·cos(lat)·cos(lon)`, `z = R·cos(lat)·sin(lon)`
- Sun fixed at world +X (module 01); Earth's origin in module 02
- Subsolar longitude: `subLon = -rotDeg` (normalized −180..180)

## UV Convention

UV computed from surface normals, NOT from SphereGeometry's built-in UVs (avoids seam artifacts):
```glsl
float lon = atan(rawN.z, rawN.x);  // +X→0, +Z→π/2
vec2 uv = vec2(lon / (2.0 * PI) + 0.5, 0.5 - lat / PI);
// u=0.5 → lon 0° (prime meridian); u=0.75 → lon 90°E
```

## Earth Revolution Math (Module 02)

```javascript
const ORBIT_A = 5.0, ORBIT_E = 0.0167;
const ORBIT_B = ORBIT_A * Math.sqrt(1 - ORBIT_E**2);
const ORBIT_C = ORBIT_A * ORBIT_E;  // focal offset

// Earth world position: sun at origin (left focus), center of ellipse at (ORBIT_C, 0, 0)
function doyToTheta(doy) {
  const fraction = ((doy - 185) / 365.25 + 1) % 1;  // 0 at aphelion (Jul 4, doy 185)
  return fraction * Math.PI * 2;  // increases CCW
}
earth_x = ORBIT_C + ORBIT_A * Math.cos(theta);
earth_z = ORBIT_B * Math.sin(theta);
```

## Running a Module

Open any `.html` file directly in a browser (`file://` protocol). No server needed. For cloud texture (5 MB), a brief wait on first load is expected — Earth renders immediately with placeholder.

If testing locally with CORS restrictions:
```bash
python3 -m http.server 8080
```

## Pedagogical Constraints (non-negotiable per module)

Every module must have:
1. **知识点 dropdown** → changes explanation text in canvas overlay (`KP_DOCS` object)
2. **Layer toggles** — all visual elements individually hideable
3. **Right panel data readout** — numerical data synchronized with 3D every frame (dual coding)
4. **≥1 productive struggle hook** — a slider whose extreme value creates cognitive conflict (documented in 知识体系.md `认知冲突触发器` column)
5. **Back link** to `./index.html` in left panel
