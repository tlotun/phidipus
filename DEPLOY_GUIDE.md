# Hướng dẫn Deploy phidipus.com

## Cấu trúc thư mục
```
phidipus_site/
├── index.html        # Website hoàn chỉnh (single file)
└── logo.webp         # Logo (đã embed trong index.html)
```

## Option 1: Cloudflare Pages (FREE - Khuyến nghị)

### Bước 1: Tạo GitHub repo
```bash
git init
git add index.html
git commit -m "init phidipus.com"
git remote add origin https://github.com/YOUR_NAME/phidipus-site.git
git push -u origin main
```

### Bước 2: Deploy lên Cloudflare Pages
1. Đăng nhập cloudflare.com → "Workers & Pages"
2. "Create application" → "Pages" → "Connect to Git"
3. Chọn repo vừa tạo
4. Build settings:
   - Framework preset: None
   - Build command: (để trống)
   - Build output directory: /  (dấu gạch chéo)
5. Click "Save and Deploy"

### Bước 3: Kết nối domain phidipus.com
Trong Cloudflare Pages → Settings → Custom domains:
1. "Set up a custom domain" → nhập: phidipus.com
2. Cloudflare tự động thêm DNS record vì bạn mua domain ở đây
3. Đợi ~2 phút → Done!

**URL:** https://phidipus.com ✅

---

## Option 2: Netlify (FREE)

### Bước 1: Kéo thả deploy
1. Truy cập app.netlify.com
2. Kéo thả folder `phidipus_site/` vào trang chủ
3. Netlify tự deploy trong 30 giây

### Bước 2: Kết nối domain
1. Site settings → Domain management → Add custom domain
2. Nhập: phidipus.com
3. Vào Cloudflare DNS:
   - Thêm CNAME record: @ → [tên site].netlify.app
   - Hoặc A record: @ → 75.2.60.5

---

## Option 3: GitHub Pages (FREE)

```bash
# Tạo repo tên: phidipus.github.io HOẶC bất kỳ tên
git init
git add index.html
git commit -m "phidipus.com"
git remote add origin https://github.com/YOUR_NAME/phidipus.com.git
git push -u origin main
```

Settings → Pages → Source: main branch → /root

**Kết nối domain Cloudflare:**
DNS → Add record:
- Type: A, Name: @, Content: 185.199.108.153
- Type: A, Name: @, Content: 185.199.109.153
- Type: CNAME, Name: www, Content: YOUR_NAME.github.io

---

## Cloudflare Settings (Bảo mật + Tốc độ)

Sau khi kết nối domain, bật các tính năng sau trong Cloudflare:

1. **SSL/TLS** → Full (strict) ✅
2. **Speed** → Auto Minify: HTML, CSS, JS ✅
3. **Caching** → Cache Level: Standard ✅
4. **Security** → Bot Fight Mode: ON ✅
5. **Network** → HTTP/3 (QUIC): ON ✅
6. **Speed** → Rocket Loader: ON ✅

---

## Cập nhật website

Mỗi khi muốn cập nhật:
```bash
# Edit index.html
git add index.html
git commit -m "update"
git push
# Cloudflare Pages tự deploy trong 1-2 phút
```

---

## Kiểm tra sau deploy

- https://phidipus.com → Trang chủ load OK
- https://phidipus.com (mobile) → Responsive OK
- Lighthouse score: Performance 90+ (do logo embed lớn, dùng external URL cho logo để cải thiện)

## Tối ưu thêm (optional)

Nếu muốn file nhỏ hơn, thay base64 logo bằng:
```html
<img src="./logo.webp" alt="...">
```
Và upload logo.webp riêng. File sẽ giảm từ 1.9MB → ~300KB.
