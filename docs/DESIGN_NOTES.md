# docs/DESIGN_NOTES.md

# MomentVault: Visual Design Notes & Creative Direction

| Item | Detail |
|---|---|
| Product | MomentVault: "Your Digital Time Capsule" (Visitor Photo Slideshow) |
| Document Version | 1.0 (Draft for Approval - Decision P-5) |
| Date | 18 November 2026 |
| Target Scope | Phases 3 & 4 (Upload Page & TV Display Page) |
| Precedence | `FRONTEND_DESIGN.md` component specifications and token architecture win. This document captures the visual philosophy, proposed palette, fluid type scale, and motion rules awaiting approval. |

---

## 1. Visual Study & Reference Synthesis

To inform the creative direction of **The Vault**, we studied the visual language, typography, spatial rhythm, and interaction patterns of five premier visual archive and inspiration sites, combined with the functional clarity of four secondary benchmarks:

1. **save.design**: Oversized editorial display typography, monospace microcopy, dotted/ASCII rules, subtle archival noise, brutalist-editorial restraint, keyboard-friendly navigation, and cataloguer metadata tags.
2. **savee.com/features/**: Calm photo-first grids, quiet typography, generous breathing room, subdued status overlays, and a measured, unhurried visual rhythm.
3. **cosmos.so**: Warm off-white canvas (`#F7F5F3`), confident editorial headlines, soft tactile radiuses, curated collection feel, and near-zero interface chrome.
4. **same.energy**: Radical reduction—the image is the interface; full-viewport drop targets that make contributing feel immediate and frictionless.
5. **designspiration.com**: Tight, ordered contact-sheet grids with disciplined alignment and mood-board energy.
6. **Secondary Benchmarks**:
   - **WeTransfer**: Single-purpose upload focus; eliminate every distraction until the upload is done.
   - **Apple TV**: Massive legibility across a room, cinematic letterboxing, uncropped framing.
   - **Stripe**: Unambiguous, friendly inline error handling that never blames the user.
   - **Airbnb**: Big, tactile touch targets (minimum 44×44 px) built phone-first.

> **Strict Boundary Notice:** MomentVault adopts *only* the visual grammar and mood of these references. It incorporates zero out-of-scope features (no search, no public gallery browsing, no moodboards, no AI tagging, no social accounts). It stays 100% self-contained with no external fonts, CDNs, or third-party assets.

---

## 2. The Creative Concept: "The Vault"

- **Name & Tagline**: *MomentVault — Your Digital Time Capsule*
- **Visual Metaphor**: A quiet, impeccably lit physical archive where each visitor's photograph is received, inspected, catalogued, and projected as an authentic preserved moment.
- **Privacy Primacy**: The vault metaphor serves elegance and clarity, but **never** obscures privacy. The consent notice is plain, unticked by default, fully visible, and unequivocally states that submitted photos will appear on a public display screen.

---

## 3. The 12 Design Principles

### Principle 1: The Image is the Sacred Hero
Interface chrome exists only to serve the image. On the upload page, file previews are treated as precious proof prints; on the TV display, the photo occupies the center of the universe. No decorative borders, gradient overlays, or loud banners compete with the photograph.

### Principle 2: Warm Editorial Paper Ground
Instead of clinical, sterile digital white (`#FFFFFF`) or harsh contrast, the visitor upload experience rests on a warm, tactile, archival paper background (`#F7F5F3` / `#F8FAFC`). This creates an immediate feeling of physical gallery space and thoughtful visual culture.

### Principle 3: Architectural Whitespace & Few Containers
Content sits directly on the page surface. We reject nested cards and boxed-in widgets. Borders and surfaces are deployed strictly where grouping is physically required: the upload dropzone, preview tiles, and modal dialogs. Whitespace establishes the visual hierarchy.

### Principle 4: Confident Editorial Display Typography
Headlines speak with the authority of an exhibition catalogue. Sized via fluid CSS `clamp()` rules, the display titles feel bespoke on mobile (28–32px) and expand comfortably on larger viewports, executed entirely through the system font stack without network penalty.

### Principle 5: Monospace as the Archivist's Notation
Microcopy, step counters (`01 Name`, `02 Consent`, `03 Photos`), file specifications, and status chips are set in clean system monospace. This gives the interface an honest, precise, "cataloguer's ledger" aesthetic that balances the warm display serif/sans typography.

### Principle 6: One Restrained Accent
Color is never decorative noise. Exactly one primary blue accent (`#1D4ED8`) is used across the system for primary actions, focus rings, and selection indicators. Status colors (amber, green, red) are reserved exclusively for semantic feedback.

### Principle 7: Tactile, Full-Surface Dropzone
Uploading photos feels physical and effortless. The dropzone acts as an expansive, welcoming target with subtle dashed architectural borders and gentle drag-hover tinting, while preserving an obvious, large tap button (≥44px height) as the primary path on mobile devices.

### Principle 8: Transparent, Per-File Status Ledger
Each selected photograph is displayed as an individual tile in a tidy grid, accompanied by an explicit status chip (`Ready`, `Uploading`, `Added`, `Rejected`) with plain-language diagnostic feedback (e.g., "Must be JPEG or PNG", "Exceeds 5 MB"). No generic failures; no mystery states.

### Principle 9: Uncompromising Consent Transparency
The consent agreement is never tucked behind links, hidden in modals, or pre-ticked. The checkbox label plainly declares: *"I understand my photos will appear publicly on the venue slideshow."* Truth in privacy takes precedence over aesthetic minimalism.

### Principle 10: The TV is an Ambient Exhibition, Not an App
The TV display operates as a living gallery installation for room-scale viewing. It features zero interactive buttons, a hidden cursor, uncropped framing (`object-fit: contain`), a deep obsidian-charcoal backdrop (`#0B0F14`), and a quiet, monospace join indicator in the corner.

### Principle 11: Subtle Ambient Projection (CSS-Only)
To soften harsh letterbox bars on the TV screen without extra network load, an optional CSS-only ambient glow projects a heavily blurred, dimmed duplicate of the currently active photo behind itself. It never crops or obscures the sharp, original photograph.

### Principle 12: Zero-Dependency, Low-Carbon Durability
The system relies entirely on native CSS custom properties, vanilla system font stacks, and zero external CDNs or heavy graphics libraries. It renders instantly on spotty venue Wi-Fi, conserves battery on older phones, and runs smoothly on low-power TV browsers.

---

## 4. Moodboards (Word Descriptions)

### 4.1 Upload Page (Phone First, 360 px)
- **Atmosphere**: A sunlit, serene gallery registration table or physical archive accession desk.
- **Palette**: Warm sun-bleached paper (`#F7F5F3`), crisp deep-slate ink (`#0F172A`), muted charcoal captions (`#475569`), and a single cobalt accent stamp (`#1D4ED8`).
- **Texture**: Clean, tactile, hairline architectural dividers (`#CBD5E1`) and subtle dotted or dashed guide lines reminiscent of archival storage boxes.
- **Typographic Cadence**: Large, authoritative title (*"MomentVault"* / *"Your Digital Time Capsule"*) anchored by disciplined, small-caps monospace step markers:
  - `01 / VISITOR NAME`
  - `02 / PUBLIC DISPLAY CONSENT`
  - `03 / SELECT MOMENTS`
- **Component Feel**:
  - The dropzone is a spacious, dotted-border card inviting drop or tap.
  - Image preview tiles are square, tidy contact sheets with rounded corners (6px), showing instant status badges in the upper corner.
  - The consent checkbox is large (minimum 24×24px touch box), clear, and legible.
  - The primary action button (*"Add to the Vault"*) is solid cobalt, prominent, and states the destination clearly.
  - Returning visitors see a warm, quiet card: *"Welcome back, Asha"* with an understated *"Not you?"* switch and a plain-text withdrawal option.

### 4.2 Display Page (TV, 1920 × 1080)
- **Atmosphere**: A darkened, contemplative museum projection room.
- **Palette**: Deep velvet black-slate (`#0B0F14`), bright neutral white text (`#F8FAFC`), subdued slate captions (`#CBD5E1`), and soft ambient halo illumination.
- **Staging**:
  - Absolute focus on the artwork. The active photograph sits centered, rendered completely with `object-fit: contain`. No heads or horizons are cropped.
  - Behind the image, an ambient glow (the identical image enlarged, blurred at 40px, and dimmed to 20% opacity via CSS) gently fills the negative space.
  - Lower corner: a quiet monospace watermark (`momentvault.local • Join the Vault`) legible from 4 meters away without stealing visual attention.
- **Empty State**:
  - A quiet, spacious invitation screen in oversized typography: *"The Vault is Waiting"*.
  - Displays the local join address and a subtle, CSS-only pulse or soft breathing keyframe to signal the screen is live.

---

## 5. Proposed Design Tokens (Decision P-5 Proposal)

These tokens expand upon `FRONTEND_DESIGN.md` Section 2 while remaining 100% compatible with the approved specifications and WCAG AA contrast standards.

### 5.1 Color Tokens

```css
:root {
  /* Surface & Canvas (Light Theme - Warm Paper) */
  --color-bg: #F7F5F3;              /* Warm archival paper ground */
  --color-surface: #FFFFFF;         /* Card / tile white */
  --color-surface-subtle: #EFECE6;  /* Secondary / muted surface */
  --color-border: #DCD8D0;          /* Subtle archival divider */
  --color-border-strong: #64748B;   /* Interactive borders (WCAG AA) */
  --color-scrim: rgba(15, 23, 42, 0.60);

  /* Typography (Light Theme) */
  --color-text: #0F172A;            /* Near-black ink (17:1 on paper) */
  --color-text-muted: #475569;      /* Muted secondary (7:1 on paper) */
  --color-text-on-primary: #FFFFFF; /* High contrast on blue button */

  /* Brand Accent */
  --color-primary: #1D4ED8;         /* Deep cobalt accent */
  --color-primary-hover: #1E40AF;
  --color-primary-subtle: #EFF6FF;  /* Light blue dropzone highlight */

  /* Status Colors (Semantic with Icons & Text) */
  --color-success: #166534;
  --color-success-bg: #F0FDF4;
  --color-success-border: #16A34A;

  --color-warning: #92400E;
  --color-warning-bg: #FFFBEB;
  --color-warning-border: #D97706;

  --color-error: #B91C1C;
  --color-error-bg: #FEF2F2;
  --color-error-border: #DC2626;

  /* Display Theme (TV - 1920x1080) */
  --display-bg: #0B0F14;            /* Deep obsidian dark ground */
  --display-text: #F8FAFC;          /* Clean white text (18:1 contrast) */
  --display-text-muted: #CBD5E1;    /* Slate microcopy */
  --display-accent: #93C5FD;        /* Soft join highlight */
  --display-ambient-opacity: 0.22;  /* Blur intensity for glow */
}
```

### 5.2 Typography Tokens (System Font Stack & Fluid Scale)

```css
:root {
  /* Zero external downloads, instant rendering */
  --font-sans: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  --font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", monospace;

  /* Fluid Display Scale for Mobile & Desktop */
  --text-display: clamp(1.75rem, 4vw + 1rem, 2.5rem); /* 28px - 40px */
  --text-h2: clamp(1.25rem, 2vw + 0.8rem, 1.75rem);   /* 20px - 28px */
  --text-h3: 1.125rem;                                /* 18px */
  --text-body: 1rem;                                  /* 16px (prevents iOS input zoom) */
  --text-small: 0.875rem;                             /* 14px (minimum readable) */
  --text-mono-step: 0.75rem;                          /* 12px monospace step badge */

  /* TV Scale (Viewport-Width Dependent for 1080p and 4K) */
  --display-title: 5vw;                               /* ~96px at 1080p */
  --display-subtitle: 2.5vw;                          /* ~48px at 1080p */
  --display-join: 3.2vw;                              /* ~61px at 1080p */
  --display-caption: 2.2vw;                           /* ~42px at 1080p */
  --display-meta: 1.5vw;                              /* ~29px at 1080p */
}
```

### 5.3 Motion & Transition Rules

1. **Phase 1 to Gate G4 Baseline**:
   - Slide swap on TV: **Instant swap** (0 ms) to keep the initial build robust, predictable, and failure-proof.
   - Button states: Gentle 150 ms color transition for `:hover` and `:active`.
2. **Post-Gate G4 (Tier B3 Enhancement)**:
   - TV Crossfade: 600 ms opacity fade between outgoing and incoming slide.
   - Strictly disabled when `prefers-reduced-motion: reduce` is active:
     ```css
     @media (prefers-reduced-motion: reduce) {
       *, *::before, *::after {
         animation-duration: 0.01ms !important;
         animation-iteration-count: 1 !important;
         transition-duration: 0.01ms !important;
       }
     }
     ```
3. **TV Ambient Glow Implementation**:
   - Implemented via a background positioned duplicate image layer:
     `filter: blur(40px) brightness(0.6); transform: scale(1.1); pointer-events: none;`
   - Completely CSS-based, reusing the loaded browser image cache (0 additional network requests).
