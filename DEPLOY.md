# PHIDIPUS — Deploy Guide (v2: Terminal Style)

Thay landing page cũ tại `https://phidipus.com/` bằng version mới: **terminal-style với ASCII art, hiệu ứng anime.js, tông đen-tím**.

---

## 📦 Files cần deploy

```
landing/
├── index.html           ← Landing page mới (terminal style)
├── login.html           ← Google sign-in + email allowlist
└── assets/
    └── spider-logo.png  ← Logo nhện đã tách nền (transparent)
```

---

## 🎯 4 bước deploy

### Bước 1 — Setup Google OAuth Client ID

1. Mở https://console.cloud.google.com
2. Tạo project mới (hoặc dùng có sẵn).
3. **APIs & Services → OAuth consent screen**:
   - User Type: **External**
   - App name: `PHIDIPUS`
   - User support email + Developer contact: email của bạn
   - **Test users**: thêm các Google email được phép vào (cùng email với allowlist client-side bên dưới)
   - Save.
4. **APIs & Services → Credentials → Create credentials → OAuth client ID**:
   - Type: **Web application**
   - Name: `Phidipus Web`
   - **Authorized JavaScript origins**:
     - `https://phidipus.com`
     - `https://www.phidipus.com`
     - `http://localhost:3000` (cho dev local)
   - Bỏ trống Authorized redirect URIs.
   - Create → copy **Client ID** (dạng `123456789-abc...apps.googleusercontent.com`)

> App ở mode "Testing" → chỉ user trong "Test users" sign in được. Đây chính là điều mình muốn cho private alpha. Không cần Verification.

### Bước 2 — Edit `login.html`

Mở `login.html`, scroll tới block CONFIG ở đầu `<script type="module">` (~dòng 460):

```js
const GOOGLE_CLIENT_ID = 'YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com';

const ALLOWED_EMAILS = [
  'your.email@gmail.com',
  // 'cofounder@gmail.com',
  // 'teammate@gmail.com',
];

const APP_URL = 'https://app.phidipus.com';
```

Thay bằng giá trị thật:

```js
const GOOGLE_CLIENT_ID = '1234567890-abcdef.apps.googleusercontent.com';

const ALLOWED_EMAILS = [
  'long.dev@gmail.com',
  'cofounder.real@gmail.com',
];

const APP_URL = 'https://app.phidipus.com';
```

### Bước 3 — Push lên GitHub Pages

Repo của bạn `tlotun/phidipus` hiện có:
- `index.html` (landing cũ — sẽ replace)
- `docs.html` (giữ)
- `logo.webp` (có thể giữ hoặc bỏ)

Làm:

```bash
# Clone repo nếu chưa có local
git clone https://github.com/tlotun/phidipus.git
cd phidipus

# Backup landing cũ
git mv index.html index.old.html

# Copy 3 file mới (giả sử bạn unzip phidipus-landing.zip vào ./tmp/)
cp ./tmp/index.html ./index.html
cp ./tmp/login.html ./login.html
mkdir -p assets
cp ./tmp/assets/spider-logo.png ./assets/spider-logo.png

# Commit + push
git add index.html login.html assets/spider-logo.png index.old.html
git commit -m "v2: terminal-style landing + spider logo + Google OAuth"
git push origin main
```

GitHub Pages rebuild trong ~60s. Verify tại https://phidipus.com.

> Nếu dùng Cloudflare proxy: dashboard → Caching → **Purge Everything** sau khi push.

### Bước 4 — Trỏ `app.phidipus.com` về Mac mini

Bạn đã có `tunnel-setup.command` trong project Spider Hunter. Chạy:

```bash
cd /path/to/spider_hunter
./tunnel-setup.command
# → domain: phidipus.com
# → subdomain: app
```

Kết quả: `https://app.phidipus.com` proxy về `http://localhost:3000` (Spider Hunter server) qua Cloudflare Tunnel.

Sau khi tunnel chạy, login flow:
- User vào `phidipus.com/login.html`
- Click Google sign-in → JWT đẩy về callback
- Nếu email trong `ALLOWED_EMAILS` → redirect đến `https://app.phidipus.com/?token=<jwt>`
- Spider Hunter server đọc token, cho qua hoặc reject.

---

## ⚠ Quan trọng — Bảo mật allowlist

Phiên bản hiện tại là **client-side gating** — có thể bypass bằng cách edit DOM/script. Đủ cho UX gating (deter casual visitors), KHÔNG đủ cho real security.

### Để thực sự bảo mật, làm thêm server-side verify trong `server.js`:

```js
// Trên đầu server.js
const GOOGLE_CLIENT_ID = '1234567890-abcdef.apps.googleusercontent.com';
const ALLOWED_EMAILS = new Set([
  'long.dev@gmail.com',
  'cofounder.real@gmail.com',
]);

async function verifyGoogleIdToken(idToken) {
  const res = await fetch(`https://oauth2.googleapis.com/tokeninfo?id_token=${idToken}`);
  if (!res.ok) return null;
  const payload = await res.json();
  if (payload.aud !== GOOGLE_CLIENT_ID) return null;
  if (payload.exp * 1000 < Date.now()) return null;
  return payload;
}

// Middleware
async function requireAuth(req, res, next) {
  const token = req.query.token
             || req.cookies?.phidipus_token
             || req.headers['authorization']?.replace('Bearer ', '');
  if (!token) return res.status(401).send('Unauthorized');

  const payload = await verifyGoogleIdToken(token);
  if (!payload || !ALLOWED_EMAILS.has(payload.email.toLowerCase())) {
    return res.status(403).send('Forbidden');
  }

  req.user = payload;
  // Set httpOnly cookie để không truyền token qua URL mãi
  if (req.query.token) {
    res.cookie('phidipus_token', token, {
      httpOnly: true, secure: true, sameSite: 'lax',
      maxAge: 3600 * 1000,  // 1h
    });
  }
  next();
}

// Cài cookie-parser: npm install cookie-parser
const cookieParser = require('cookie-parser');
app.use(cookieParser());

// Apply lên tất cả route trừ webhook receiver (Helius phải public)
app.use((req, res, next) => {
  if (req.path.startsWith('/api/webhook/helius')) return next();
  return requireAuth(req, res, next);
});
```

Sau khi setup server-side, dù user bypass client-side cũng không vào được app.

---

## 📁 Cấu trúc cuối cùng repo `tlotun/phidipus`

```
phidipus/
├── index.html         ← LANDING MỚI (terminal-style)
├── login.html         ← GOOGLE SIGN-IN
├── assets/
│   └── spider-logo.png
├── docs.html          (giữ nguyên cũ)
├── logo.webp          (có thể xóa — landing mới dùng spider-logo.png)
├── index.old.html     (backup landing cũ — xoá sau khi verify OK)
├── DEPLOY_GUIDE.md
└── .gitignore
```

---

## 🧪 Test checklist sau khi deploy

- [ ] `https://phidipus.com/` render landing mới (ASCII PHIDIPUS art, bubble particles, tông đen-tím)
- [ ] Logo nhện hiện ở navbar góc trái (nền trong suốt, không thấy hộp đen)
- [ ] Hero ASCII art có hiệu ứng glow-in character-by-character
- [ ] Headlines "The autonomous terminal..." slide-up animation
- [ ] Scroll xuống các section: modules cards reveal mượt
- [ ] Hover module card: scramble text effect trên tên + 3D tilt nhẹ
- [ ] Ticker chạy ngang liên tục
- [ ] Stat counters đếm lên từ 0 đến giá trị thật khi vào view
- [ ] Click "Open App" → tới `/login.html`
- [ ] Login page có ASCII "ACCESS" art, corner brackets mint, terminal readout panel
- [ ] Google sign-in button render
- [ ] Sign in bằng email **trong** allowlist → `[ ✓ GRANTED ]` → redirect tới `app.phidipus.com`
- [ ] Sign in bằng email **ngoài** allowlist → `[ ✗ DENIED ]` với hiệu ứng shake
- [ ] Mobile: responsive, ASCII art không vỡ
- [ ] Tiếng Việt render đẹp (IBM Plex Mono load đầy đủ qua Google Fonts CSS2)

---

## 🎨 Tuỳ biến nhanh

**Đổi tông màu**: cả 2 file có CSS variables ở `:root`. Sửa 5 dòng:

```css
--violet:      #9945FF;   /* Solana purple — màu chính */
--violet-soft: #c084fc;   /* sáng hơn — accent */
--violet-deep: #6b21a8;   /* tối hơn — shadow */
--mint:        #14F195;   /* Solana green — màu phụ */
--magenta:     #ff3eb5;   /* alert/highlight */
```

**Đổi headline**: search `"The autonomous"` trong `index.html`.

**Đổi ASCII art logo**: thay text trong `<pre class="hero-art">`. Sinh ASCII font khác tại http://patorjk.com/software/taag — chọn "ANSI Shadow" hoặc "Slant" cho vibe terminal.

**Đổi số liệu ticker**: search `tickerData` trong `<script>` của `index.html`.

**Thêm module card**: copy 1 block `<div class="module-card reveal">...</div>` trong `#modules` và sửa nội dung.

**Thay logo nhện**: thay `assets/spider-logo.png`. Nếu logo bạn có nền đen, dùng Photoshop/GIMP/Photopea remove background trước.

---

## 🐞 Troubleshooting

**Vấn đề**: Vietnamese ký tự `ế ổ ư` bị tách dấu/rời  
**Nguyên nhân**: Google Fonts CSS2 chưa nạp Vietnamese subset (network/CORS)  
**Fix**: Verify file `<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono...">` load OK trong DevTools Network. Nếu fail → host font local trong `assets/fonts/` (tải từ https://fontsource.org/fonts/ibm-plex-mono).

**Vấn đề**: Logo nhện vẫn thấy nền đen  
**Fix**: file `assets/spider-logo.png` đã có nền trong suốt. Nếu vẫn thấy nền, có thể cache cũ. Hard refresh (Ctrl+Shift+R) hoặc purge Cloudflare cache.

**Vấn đề**: anime.js không chạy → site bị blank  
**Fix**: file có **fallback safety** — sau 600ms tự show. Nếu vẫn blank, check console: có thể CSP block jsdelivr. Add `<meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net https://accounts.google.com;">` vào `<head>`.

**Vấn đề**: Google sign-in không render  
**Fix**: chưa cấu hình `GOOGLE_CLIENT_ID` — file sẽ show notice nhắc bạn edit constants. Sau khi edit, hard refresh.
