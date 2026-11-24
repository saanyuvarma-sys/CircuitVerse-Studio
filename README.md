# CircuitVerse Studio

**Design. Simulate. Explore.**

CircuitVerse Studio is an interactive engineering learning platform for exploring electronics, digital circuits, aerospace concepts, and simulation workflows.

## Live website

https://saanyuvarma-sys.github.io/CircuitVerse-Studio/

> The live URL becomes available after GitHub Pages completes a successful deployment and Pages is enabled for this repository.

## Fixed build (deployed)

The deployed `index.html` is a small loader that boots the original single-file app from a pinned commit and applies the verified fixes from [`fix.patch`](fix.patch) at runtime. Fixes included:

1. **Run button works** — the toolbar binding threw on `#emode-seg` and silently killed every toolbar binding (Run/Pause/Stop/Reset/Validate/PNG/Save/zoom/panel tabs). Fixed, and the first simulation evaluation now happens immediately so LEDs light up without the 220 ms delay.
2. **"Browser storage is full" handled** — old simulation recordings are pruned automatically and the save is retried before showing a warning; the toast is rate-limited; Settings now shows a storage usage bar and a **Free up space** button; dense simulation tracks are slimmed before persisting.
3. **Example circuits** — a new **Examples** button in the Electronics Lab loads three ready-to-run circuits (LED + Battery, Blink D13, Button + LED), and the first-run starter circuit is now fully wired (battery → resistor → LED) so the very first press of Run lights the LED.
4. **Card alignment** — stat, lab and mission cards align consistently.

## Laboratories

- **Electronics Lab** — explore circuit construction, components, wiring, and digital logic.
- **Aerospace Lab** — explore educational aerospace and UAV concepts.
- **Simulation Lab** — work with simulation activities and analysis.

## Project features

- Interactive engineering-learning interface
- Circuit design and simulation learning
- Project workspace and experiment documentation
- GitHub-friendly project sharing workflow

## Technology

- HTML
- CSS
- JavaScript

## Run locally

Open `index.html` in a modern browser. Some browser APIs or browser-storage features may behave differently when opened as a local file; use a local static web server for the closest match to GitHub Pages.

## Single-file version

Prefer a plain single-file app instead of the loader? Apply the patch and use the result as `index.html`:

```bash
git apply fix.patch
```

`fix.patch` applies to the original single-file `index.html` (commit `b5e8fc4`) and produces the same fixed experience with no loader.

## Deployment

The workflow at `.github/workflows/deploy-pages.yml` publishes the repository root to GitHub Pages whenever changes are pushed to the `main` branch, and can also be run manually from the Actions tab.

In the repository settings, set **Settings → Pages → Build and deployment → Source** to **GitHub Actions**.

## Developer

Saanyu Varma

## License

MIT. See [LICENSE](LICENSE).
