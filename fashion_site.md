# Build Instruction - "vyrn." Fashion Runway Site

Build the website described below **exactly** as specified. The whole design is embedded
in this file - do not look for any external design source.

## 0. Deliverable & ground rules

- Produce **one single `index.html`** in the project root. All CSS and JavaScript are
  **inline** in that one file. **No frameworks, no build tools, no external requests**
  (no CDNs, web-fonts, or libraries).
- When finished, **serve the result on a fresh localhost port** (pick any free port,
  e.g. `python3 -m http.server <port>`), and give the user the URL.
- The result **must be responsive** - it fills any window size, on desktop and mobile,
  with no white letterbox bars and nothing important cropped. Re-layout on resize.
- The site has two states: the **runway** (all models walking) and the **detail** view
  (one model centered with product text). Behaviour is defined in sections 5-8.

## 1. The media (videos live in the project folder)

- At build time, look in the project folder for **video files** (`.mp4`, `.mov`,
  `.webm`, `.m4v`). Sort them by filename. **Each video becomes one model** on the
  runway, in that order. Do **not** hardcode filenames - discover whatever is present.
- If a video is in a container/codec the browser cannot play inline (e.g. HEVC in
  `.mov`), transcode it to **H.264 `.mp4`** (`yuv420p`, `+faststart`, no audio) with
  ffmpeg, and, for smooth simultaneous playback, downscale the longer side to ~900px.
  Reference the browser-playable files from the HTML.
- Every `<video>` is `muted`, `loop`, `autoplay`, `playsinline`, `preload="auto"`.
- The clips are expected to be full-body model shots on a **white/near-white
  background**; they are composited on a white page so the frame edges are invisible
  and a nearer model cleanly occludes a farther one.

### 1a. Fallback fill (REQUIRED - no videos, or a clip fails to load)

If the project folder contains **no** videos, still build the site with **7 models**.
For **any** model whose video is missing or fails to load, render a **solid fill of
color `#EDE6E2`** in place of that video frame. The fill occupies the same frame and
**behaves identically to a video** in every way: it travels along the runway, scales
with perspective, responds to hover (dim / freeze), and opens the same centered detail
view on tap. In other words, the fallback is a drop-in visual replacement for the clip;
only playback is absent. Implement it by giving each model either a `<video>` or, on
failure/absence, a filler element with `background:#EDE6E2` that fills the model frame
(width/height 100% of the model box).

## 2. Canvas, scaling & responsiveness

- Author the composition on a fixed **1440 x 900** "stage", then uniformly **scale it
  to COVER the viewport** so it always fills the window (no white side/top bars):
  `scale = max(innerWidth / 1440, innerHeight / 900)`.
- The stage is **anchored to the bottom-center** (`transform-origin: bottom center`,
  positioned `left:50%; bottom:0; transform: translateX(-50%) scale(scale)`), so the
  bottom nav stays put and only the empty area above the models is trimmed on wide
  screens. The stage has `overflow: hidden` (it clips cleanly at the screen edges).
- Recompute the scale on every `resize`.
- Page/stage background is **white `#ffffff`**.

## 3. Fonts & colors

- UI text font stack (sans): `"Suisse Intl", "Helvetica Neue", Helvetica, Arial, sans-serif`.
- Price font stack (serif, **lining figures** so digits sit on the baseline):
  `"Instrument Serif", "Didot", "Bodoni 72", "Times New Roman", Times, serif` plus
  `font-variant-numeric: lining-nums; font-feature-settings: "lnum" 1, "onum" 0;`.
- Text color `#000`; page `#fff`; fallback fill `#EDE6E2`.

## 4. Static furniture

### 4a. Logo - top-left, fixed to the viewport
Pinned `position: fixed; left:12px; top:16px; width:161.87px; height:71px; z-index:100000;
transform-origin: top left;` and scaled by the same cover-scale as the stage each resize
(so it grows proportionally but never gets clipped). It must sit **above** the models.
Use this exact inline SVG (the "vyrn." wordmark):

```html
<svg preserveAspectRatio="none" viewBox="0 0 161.87 71" fill="none" xmlns="http://www.w3.org/2000/svg">
  <path d="M156.75 51.699C155.305 51.699 154.058 51.2395 153.008 50.3204C152.023 49.3356 151.53 48.0555 151.53 46.4799C151.53 44.9699 152.023 43.7554 153.008 42.8363C154.058 41.8516 155.305 41.3592 156.75 41.3592C158.063 41.3592 159.244 41.8516 160.295 42.8363C161.345 43.7554 161.87 44.9699 161.87 46.4799C161.87 48.0555 161.345 49.3356 160.295 50.3204C159.244 51.2395 158.063 51.699 156.75 51.699Z" fill="black"/>
  <path d="M104.881 50.8127C103.896 50.8127 103.403 50.4517 103.403 49.7295C103.403 49.1387 103.83 48.7448 104.684 48.5478L105.865 48.3509C107.244 48.0883 108.13 47.6944 108.524 47.1692C108.984 46.5784 109.213 45.5936 109.213 44.215V11.1276C109.213 9.9459 109.016 9.15811 108.623 8.76421C108.294 8.30467 107.671 8.00924 106.752 7.87794L104.684 7.58252C103.83 7.51687 103.403 7.1558 103.403 6.49931C103.403 5.97411 103.929 5.61303 104.979 5.41609C107.014 5.08784 108.557 4.56264 109.607 3.8405C110.723 3.11835 111.872 2.23208 113.054 1.18169C113.645 0.590846 114.137 0.295422 114.531 0.295422C115.122 0.295422 115.417 0.689319 115.417 1.47711V5.90846C115.417 6.4993 115.614 6.8932 116.008 7.09015C116.468 7.22145 116.96 7.0245 117.485 6.49931C120.111 4.00462 122.409 2.29773 124.378 1.37864C126.414 0.459545 128.547 0 130.779 0C133.733 0 136.097 1.05039 137.869 3.15118C139.642 5.18632 140.528 8.20619 140.528 12.2108V44.215C140.528 45.5936 140.725 46.5784 141.119 47.1692C141.579 47.6944 142.498 48.0555 143.876 48.2524L146.043 48.5478C146.765 48.6791 147.126 49.073 147.126 49.7295C147.126 50.4517 146.732 50.8127 145.944 50.8127H129.401C128.482 50.8127 128.022 50.4517 128.022 49.7295C128.022 49.073 128.383 48.6791 129.105 48.5478L130.484 48.3509C131.862 48.1539 132.749 47.76 133.143 47.1692C133.602 46.5784 133.832 45.5936 133.832 44.215V13.294C133.832 10.0772 133.241 7.8123 132.059 6.49931C130.878 5.18631 129.204 4.52982 127.037 4.52982C124.936 4.52982 123.033 5.15349 121.326 6.40083C119.684 7.58252 118.371 9.19094 117.387 11.2261C116.402 13.1956 115.91 15.3948 115.91 17.8239V44.215C115.91 45.5936 116.107 46.5784 116.5 47.1692C116.96 47.76 117.879 48.1211 119.258 48.2524L121.917 48.5478C122.639 48.6791 123 49.0402 123 49.6311C123 50.4188 122.507 50.8127 121.523 50.8127H104.881Z" fill="black"/>
  <path d="M87.7106 6.8932C90.4022 2.29773 93.6519 0 97.4596 0C99.626 0 101.234 0.656495 102.285 1.96949C103.335 3.21683 103.86 4.62829 103.86 6.20388C103.86 7.45123 103.532 8.46879 102.876 9.25659C102.285 10.0444 101.333 10.4383 100.02 10.4383C98.9695 10.4383 98.1817 10.2085 97.6565 9.74896C97.197 9.22376 96.8031 8.63291 96.4748 7.97642C96.2122 7.31992 95.884 6.7619 95.4901 6.30236C95.0962 5.77716 94.4397 5.51456 93.5206 5.51456C92.3389 5.51456 91.2228 6.23671 90.1725 7.681C89.1221 9.05964 88.2358 10.9306 87.5137 13.294C86.8572 15.6574 86.5289 18.349 86.5289 21.3689V44.215C86.5289 45.5936 86.7259 46.5784 87.1198 47.1692C87.5793 47.76 88.4984 48.1211 89.877 48.2524L92.5358 48.5478C93.258 48.6791 93.6191 49.0402 93.6191 49.6311C93.6191 50.4188 93.1267 50.8127 92.1419 50.8127H75.4998C74.515 50.8127 74.0227 50.4517 74.0227 49.7295C74.0227 49.1387 74.4494 48.7448 75.3028 48.5478L76.4845 48.3509C77.8632 48.0883 78.7494 47.6944 79.1433 47.1692C79.6029 46.5784 79.8327 45.5936 79.8327 44.215V11.1276C79.8327 9.9459 79.6357 9.15811 79.2418 8.76421C78.9136 8.30467 78.2899 8.00924 77.3708 7.87794L75.3028 7.58252C74.4494 7.51687 74.0227 7.1558 74.0227 6.49931C74.0227 5.97411 74.5479 5.61303 75.5983 5.41609C77.6334 5.08784 79.1762 4.56264 80.2266 3.8405C81.3426 3.11835 82.4915 2.23208 83.6732 1.18169C84.264 0.590846 84.7564 0.295422 85.1503 0.295422C85.7411 0.295422 86.0365 0.689319 86.0365 1.47711V6.79472C86.0365 7.25427 86.2335 7.5497 86.6274 7.681C87.0213 7.74665 87.3823 7.48405 87.7106 6.8932Z" fill="black"/>
  <path d="M44.5104 71C42.7378 71 41.3264 70.3763 40.276 69.129C39.2256 67.9473 38.7004 66.5687 38.7004 64.9931C38.7004 63.7457 39.0615 62.6953 39.7836 61.8419C40.5058 60.9884 41.4905 60.5617 42.7378 60.5617C43.7882 60.5617 44.5432 60.7915 45.0028 61.251C45.4623 61.7106 45.7906 62.2358 45.9875 62.8266C46.1844 63.4175 46.3814 63.9427 46.5783 64.4022C46.7753 64.8618 47.1692 65.0915 47.76 65.0915C48.4822 65.0915 49.1387 64.5663 49.7295 63.516C50.3204 62.5312 51.141 60.3976 52.1914 57.1151C52.8479 55.1456 53.2418 53.4387 53.3731 51.9945C53.57 50.5502 53.5372 49.0074 53.2746 47.3662C53.0777 45.7249 52.5853 43.7226 51.7975 41.3592L41.0638 7.18865C40.6042 5.61306 40.1447 4.59549 39.6851 4.13595C39.2912 3.6764 38.6348 3.34815 37.7157 3.15121L36.2385 2.85578C35.3851 2.65884 34.9584 2.23211 34.9584 1.57562C34.9584 0.919121 35.4179 0.590873 36.337 0.590873H52.5853C53.5044 0.590873 53.9639 0.919121 53.9639 1.57562C53.9639 2.29776 53.5372 2.72449 52.6838 2.85578L51.1082 3.05273C49.5326 3.24968 48.515 3.64358 48.0555 4.23442C47.6616 4.75962 47.6944 5.74436 48.1539 7.18865L56.8197 36.0416C57.0166 36.6981 57.312 37.0264 57.7059 37.0264C58.1655 37.0264 58.4937 36.6981 58.6907 36.0416L67.061 8.96119C67.5862 7.18865 67.7175 5.87566 67.4549 5.02222C67.2579 4.10312 66.3389 3.51228 64.6976 3.24968L62.6297 2.85578C61.7762 2.65884 61.3495 2.23211 61.3495 1.57562C61.3495 0.919121 61.809 0.590873 62.7281 0.590873H75.2344C76.1535 0.590873 76.613 0.951945 76.613 1.67409C76.613 2.33059 76.2191 2.79014 75.4313 3.05273L74.5451 3.34816C73.5603 3.6764 72.7397 4.26725 72.0832 5.12069C71.4267 5.97414 70.7374 7.45125 70.0152 9.55204L55.3426 55.3426C53.8983 59.9381 52.5853 63.3518 51.4036 65.5839C50.2875 67.816 49.2043 69.2603 48.1539 69.9168C47.1035 70.6389 45.889 71 44.5104 71Z" fill="black"/>
  <path d="M20.4827 51.699C19.8918 51.699 19.4651 51.338 19.2025 50.6158L6.10541 7.18865C5.64586 5.61306 5.18631 4.5955 4.72677 4.13595C4.33287 3.6764 3.67637 3.34816 2.75728 3.15121L1.28017 2.85578C0.426722 2.65884 0 2.23211 0 1.57562C0 0.919121 0.459547 0.590873 1.37864 0.590873H17.6269C18.546 0.590873 19.0055 0.919121 19.0055 1.57562C19.0055 2.29776 18.5788 2.72449 17.7254 2.85578L16.1498 3.05273C14.5742 3.24968 13.5895 3.64358 13.1956 4.23442C12.8017 4.75962 12.8017 5.74436 13.1956 7.18865L21.3689 35.6477C21.5659 36.3699 21.8941 36.7309 22.3537 36.7309C22.8132 36.7309 23.1415 36.3699 23.3384 35.6477L31.3148 8.96119C31.84 7.18865 31.9713 5.87566 31.7087 5.02222C31.5118 4.10312 30.5927 3.51228 28.9514 3.24968L26.8835 2.85578C26.03 2.65884 25.6033 2.23211 25.6033 1.57562C25.6033 0.919121 26.0629 0.590873 26.982 0.590873H39.4882C40.4073 0.590873 40.8668 0.951945 40.8668 1.67409C40.8668 2.33059 40.4729 2.79014 39.6851 3.05273L38.7989 3.34816C37.8141 3.6764 36.9607 4.26725 36.2385 5.12069C35.5821 5.97414 34.9256 7.45125 34.2691 9.55204L21.7628 50.6158C21.5002 51.338 21.0735 51.699 20.4827 51.699Z" fill="black"/>
</svg>
```

### 4b. Bottom navigation bar - inside the stage, centered
A horizontal bar of pill buttons, `position:absolute; left:50%; bottom:20px;
transform:translateX(-50%); display:flex; gap:4px; z-index:100001;` (above the models).
Pills: `background: rgba(0,0,0,0.05); backdrop-filter: blur(20px);
border-radius: 200px; height: 28px;`.
- One search **icon pill** (28x28, `padding:0`, centers a 14px icon).
- One wide **links pill** (`width:600px; padding:6px 24px; justify-content:space-between`)
  containing 5 uppercase links, 12px, `font-weight:600`, `letter-spacing:0.2px`,
  color `#000`: **classic | vintage | about | instagram | pinterest | email | spotify | shipping**.
- Three more icon pills: heart, user, cart.
Icons are thin-line (feather-style), `stroke:#000; stroke-width:2; 14x14`. Use these:

```html
<!-- search -->  <svg viewBox="0 0 24 24" fill="none" stroke="#000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
<!-- heart  -->  <svg viewBox="0 0 24 24" fill="none" stroke="#000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/></svg>
<!-- user   -->  <svg viewBox="0 0 24 24" fill="none" stroke="#000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
<!-- cart   -->  <svg viewBox="0 0 24 24" fill="none" stroke="#000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="9" cy="21" r="1"/><circle cx="20" cy="21" r="1"/><path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"/></svg>
```

## 5. The runway - how the models move (the core animation)

The models walk **toward the viewer** along a perspective line: as a model advances it
**glides horizontally (left->right), drifts slightly downward, and grows in scale** - a
figure that is far away is small and high on the runway; as it comes forward it becomes
larger and lower, then walks off the right edge. It is a **seamless, infinite loop**:
the moment a model reaches the front and exits, another appears tiny in the back, so it
reads as an endless parade down a catwalk. Nothing ever pops or resets on screen.

**Model frame geometry.** Each model is an absolutely-positioned box of natural size
**1200 x 2980** (`transform-origin: 0 0`), placed and scaled per frame via `transform`.
The video (or fallback fill) fills this box. For a real clip framed with a small margin
around the model, map it so the feet sit on the box's bottom (the runway line) and the
head reaches the top:
```css
.model video { position:absolute; top:0; left:50%; height:109.29%; width:auto;
  transform: translate(-50%, -4%); transform-origin: top center; }
```
(The fallback `#EDE6E2` fill instead fills the box 100% x 100%.)

**Perspective path (exact keyframes).** The path is a fixed poly-line through 9 control
points, indexed `0..8`. Index 0 is the far-back (off-screen left) entry, index 7 is the
front (largest, exiting right), index 8 is an extrapolated fully-off-screen "exit" frame
used only to make the wrap invisible. For a point along the path (parameter `q`), lerp
between the two surrounding control points:
```js
var CX = [-96.5, -1.0, 108.5, 263, 460.5, 694, 1013, 1422.5, 1832]; // model centre X (stage px)
var BY = [506, 521, 554, 600, 662, 736, 831, 961, 1091];            // model bottom Y (stage px)
var SC = [0.034167,0.046667,0.0675,0.098333,0.139167,0.188333,0.251667,0.339167,0.4572]; // scale
var NAT_W = 1200, NAT_H = 2980;

function lerp(a,b,t){ return a + (b-a)*t; }
function place(el, q){
  var i = Math.floor(q), f = q - i;
  var s  = lerp(SC[i], SC[i+1], f);
  var cx = lerp(CX[i], CX[i+1], f);
  var by = lerp(BY[i], BY[i+1], f);
  var x = cx - (NAT_W*s)/2;   // box left  = centre - halfwidth
  var y = by - (NAT_H*s);     // box top   = bottom - height
  el.style.transform = "translate3d("+x+"px,"+y+"px,0) scale("+s+")";
  el.style.zIndex = String(Math.round(s*10000)); // nearer (larger) model on top
}
```

**Distribution & motion.** There are `N` models (one per video, or 7 in the no-video
fallback). Define `PERIOD = 8` (the number of path "slots"; keeping index 8 as the exit
frame is what makes the wrap seamless) and `SPACING = PERIOD / N`, so the models are
evenly spread along the path. Advance a single continuous progress value with real
delta-time and place every model each animation frame:
```js
var PERIOD = 8, SLOT_MS = 3600;      // 3600ms per slot ~ an elegant walking pace
var prog = 0, lastTime = null, paused = false;
function frame(now){
  if (lastTime === null) lastTime = now;
  var dt = now - lastTime; lastTime = now;
  if (!paused){
    prog = (prog + dt/SLOT_MS) % PERIOD;
    for (var i=0;i<models.length;i++) place(models[i], (i*SPACING + prog) % PERIOD);
  }
  requestAnimationFrame(frame);
}
requestAnimationFrame(frame);
```
Because a model becomes fully off-screen (right) as `q` approaches 8 and is still fully
off-screen (left) at `q = 0`, the wrap from 8->0 happens entirely out of view - the loop
is seamless. Stagger each clip's `currentTime` (e.g. `(i/N)*duration`) so the models do
not stride in unison. Honor `prefers-reduced-motion` by placing the models once and not
animating.

## 6. Hover (pointer devices)

Hovering any model **freezes the entire runway** (set `paused = true`; all horizontal
motion stops), **pauses and dims every other model to 70% opacity**, and lets **only the
hovered clip keep playing**. There is **no scale change** on hover. On mouse-out,
everything resumes: `paused = false`, opacities restore, all clips play again. Dim via a
0.45s opacity transition on an inner wrapper so it does not fight the per-frame transform
written to the model element.

## 7. Tap / click a model - open the detail view

Clicking (tapping) a model opens a product page for that model:
- **All other videos turn off** (pause) and fade out.
- The tapped model **animates to the center of the screen** and keeps playing, sized to
  roughly **90% of the viewport height** (leave room for the side text). Use a ~0.65s
  transform transition. The runway stays frozen while the detail view is open.
- Two text panels fade in (0.5s): a **left** panel and a **right** panel (section 8).

**Centering math.** The model lives inside the cover-scaled, bottom-anchored stage, so a
stage point `(sx,sy)` lands at viewport `(vw/2 + (sx-720)*cs, vh + (sy-900)*cs)` where
`cs` is the current cover-scale. To center the selected model in the viewport:
```js
function detailTransform(el){
  var cs = coverScale, vw = innerWidth, vh = innerHeight;
  var s = (vh*0.9) / (NAT_H*cs);              // ~90% of viewport height
  var maxBoxW = vw - 2*300;                   // keep clear of the side panels
  if (NAT_W*s*cs > maxBoxW) s = maxBoxW/(NAT_W*cs);
  var mcy = 900 - vh/(2*cs);                  // stage-y that maps to viewport centre
  var x = 720 - (NAT_W*s)/2;
  var y = mcy - (NAT_H*s)/2;
  el.style.transform = "translate3d("+x+"px,"+y+"px,0) scale("+s+")";
}
```
Recompute this on `resize` while the detail view is open (without the transition, so it
snaps to the new center). Lift the selected model above the others with a high `z-index`.

## 8. Detail panels & per-model product data

Both panels are **fixed to the viewport**, vertically centered, uppercase, 12px, weight
600, color `#000`. Hidden by default (opacity 0 / visibility hidden); shown when the
detail view is open.

- **Left panel** - `left:24px; width:220px;` column, `gap:20px`, left-aligned:
  1. product **name**
  2. product **description**  (`line-height:1.45`)
  3. **more details** (underlined)
- **Right panel** - `right:24px; width:154px;` column, `gap:12px`, right-aligned:
  1. **add to wishlist** (underlined)
  2. size row - `34 36 38 40 42` (`display:flex; gap:15px`; the selected size, e.g. `38`,
     is underlined)
  3. **price** - the serif/lining-figure style from section 3, `font-size:40px;
     letter-spacing:-0.8px; line-height:1;`
  4. **add to cart** - black rounded button, `width:107px; height:32px; border-radius:23px;`
     white uppercase 12px label

**Product copy.** Give **every model a distinct name, description and price** (all
different). Assign entries in order to the discovered models (cycle the list if there are
more models than entries). Use this list:

| # | Name | Description | Price |
|---|------|-------------|-------|
| 1 | Sabbia Croc Set | Two-piece in croc-embossed leather - a sculpted long-sleeve top with a floor-grazing column skirt and matching boots. | £2450 |
| 2 | Balloon-Sleeve Leather Top | Nappa leather top with dramatic leg-of-mutton sleeves, worn with second-skin leggings and opera gloves. | £1890 |
| 3 | Dune Jersey Catsuit | Second-skin turtleneck catsuit in stretch jersey. Body-hugging silhouette with a hidden zip and extra-long sleeves. | £1100 |
| 4 | Onyx Leather Playsuit | Long-sleeve leather playsuit with peaked shoulders and a nipped waist, shown over sheer tights. | £1650 |
| 5 | Drape Halter Bodysuit | Halter bodysuit in fluid jersey with a gathered neckline, styled with leather opera gloves. | £780 |
| 6 | Corset Leather Playsuit | Boned leather playsuit with a sweetheart bustier and fine straps. Finished with knee-high boots. | £1290 |
| 7 | Cascade Fringe Gown | Floor-sweeping halter gown in liquid fringe. Bias-cut column silhouette that moves with every step. | £3200 |
| 8 | Vapor Silk Slip | Bias-cut silk slip dress with a whisper-fine strap and a fluid, liquid drape. | £960 |

## 9. Closing the detail view - BACK ARROW ONLY

The detail view is dismissed **only by the browser Back button**. Wire it so opening the
detail view pushes a history entry (`history.pushState`), and a `popstate` event closes
it (reverse the transitions, fade the other models back in, resume all playback, unfreeze
the runway so it continues from where it froze). Do **not** add any other way to close it
- no tap-to-close, nothing else.

## 10. Acceptance checklist

- [ ] One self-contained `index.html`; inline CSS/JS; no frameworks or external requests.
- [ ] Models walk an endless, seamless perspective loop (grow + glide right + drift down);
      no visible reset/pop; clips staggered.
- [ ] Fills the window at any size with no white bars; nothing important cropped; re-lays
      out on resize (desktop and mobile).
- [ ] Logo top-left and nav bottom-center always sit above the models.
- [ ] Hover freezes the runway, dims others to 70%, keeps only the hovered clip playing,
      no scaling.
- [ ] Tap centers the model (~90% height), turns the others off, and fades in the left/
      right product panels with distinct name/description/price per model.
- [ ] Detail view closes **only** via the browser Back button.
- [ ] Missing/absent videos fall back to a solid `#EDE6E2` fill that behaves exactly like
      a clip (runway motion, hover, tap/detail).
- [ ] Served on a fresh localhost port; the URL is reported to the user.
