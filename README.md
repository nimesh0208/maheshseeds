# Mahesh Seeds &amp; Fertilizers

Kisanon ke liye ek simple, fast aur secure e-commerce website — **sirf HTML, CSS aur JavaScript**.
Koi framework, koi build step, koi external CDN ya font nahi. Poori site offline bhi chalti hai.

> **Demo store:** Yeh ek front-end demo hai. Koi server, database ya payment gateway connected nahi hai.
> Order aapke browser me save hota hai aur WhatsApp message ke through dukaan tak pahunchta hai.
> Real payments lene se pehle backend + payment gateway + server-side validation zaroori hai
> (neeche "Production checklist" dekhein).

---

## 1. Quick start

### Option A — seedha kholen

`index.html` par double-click karein. Saare links, cart, images aur pages bina internet ke chalenge.

### Option B — local server (recommended)

Kuch browsers `file://` par `localStorage` ko block karte hain. Isliye testing ke liye chhota server chalayein:

```bash
cd "mahesh seeds"

# Python 3
python -m http.server 4173

# ya Node.js
npx serve . -l 4173

# ya PHP
php -S localhost:4173
```

Phir browser me kholein: <http://localhost:4173/>

> **Note:** `file://` par security headers (CSP) apply nahi hote. Headers test karne ke liye
> hamesha local server ya hosting use karein.

---

## 2. Pages

| File | Kaam |
| --- | --- |
| `index.html` | Homepage — hero, categories, bestsellers, reviews, newsletter |
| `shop.html` | Catalogue — category filter, budget slider, search, sort, URL params |
| `product.html` | Product detail — `product.html?id=tomato-hybrid` |
| `cart.html` | Cart — quantity change, remove, coupon, order summary |
| `checkout.html` | Delivery form, COD / UPI-on-delivery, order confirmation |
| `about.html` | Dukaan ki kahani, values, FAQ |
| `contact.html` | Contact cards + message form (WhatsApp handoff) |
| `policies.html` | Shipping, returns, privacy, terms (`#shipping`, `#returns`, `#privacy`, `#terms`, `#contact`) |

Har page ka header, footer aur WhatsApp button same hai. Naya page banate waqt `shop.html` ka
header/footer copy karein aur niche diye gaye scripts ka order na badlein.

---

## 3. File structure

```text
mahesh seeds/
├── index.html
├── shop.html
├── product.html
├── cart.html
├── checkout.html
├── about.html
├── contact.html
├── policies.html
├── README.md
├── _headers              # Netlify / Cloudflare Pages security headers
├── .htaccess             # Apache security headers
├── assets/
│   └── img/              # 20 local SVG images (products, hero, logo, favicon)
├── css/
│   └── style.css         # design tokens, layout, components, responsive rules
└── js/
    ├── utils.js          # safe DOM builder, formatting, storage, toasts
    ├── data.js           # 17-product catalogue, categories, coupons + validation
    ├── cart.js           # cart state, pricing, coupon logic
    ├── main.js           # header/nav, product cards, global interactions
    ├── home.js           # homepage rendering
    ├── shop.js           # filters, search, sort, URL params
    ├── product.js        # product detail + related products
    ├── cart-page.js      # cart page UI
    ├── checkout.js       # form validation, order creation, WhatsApp handoff
    └── contact.js        # contact form validation + WhatsApp handoff
```

**Script order (har page par):** `utils.js` → `data.js` → `cart.js` → `main.js` → page script.
Sab `defer` hain. `utils.js` pehle chahiye kyunki `data.js` uske helpers use karta hai.

---

## 4. Features

- 17 products: seeds, fertilizers, pesticides aur tools.
- Category filter, budget slider, text search (Hinglish + English keywords), 5 sort options.
- Shareable URLs: `shop.html?category=seeds`, `shop.html?q=neem`, `product.html?id=urea-45kg`.
- Cart: quantity controls, stock caps, 30-item limit, live totals, free-delivery progress.
- Coupons, delivery charge, COD fee — sab client-side calculate hote hain.
- Checkout: Indian mobile/PIN validation, state dropdown, terms checkbox, order ID, success screen.
- Order confirmation ka WhatsApp message (aapke apne WhatsApp app me khulta hai).
- Delivery free ≥ ₹999, warna ₹49. COD fee ₹30. Coupon stack nahi hote (ek time par ek).
- Recent products (`ms_recent_v1`), cart badge, mobile navigation, keyboard/Escape support.
- Fully responsive, `prefers-reduced-motion` respected, skip-link aur ARIA labels.
- Zero external requests — fonts, icons aur images sab local.

### Coupon codes

| Code | Benefit | Minimum order |
| --- | --- | --- |
| `KHETI10` | 10% off, max ₹250 | ₹999 |
| `SEED50` | ₹50 flat off | ₹499 |
| `FREESHIP` | Free delivery | ₹299 |

Coupons `js/data.js` ke `COUPONS` object me hain. Shipping/COD amounts `js/cart.js` ke top par
(`FREE_SHIP_AT`, `SHIP_FEE`, `COD_FEE`) — dono jagah change karna ho to consistency rakhein,
warna policies page aur checkout text bhi update karein.

---

## 5. Security

Yeh site "secure by design" hai, lekin **static demo** hone ki wajah se iski limits bhi hain.

### Jo implement kiya gaya hai

- **No `innerHTML` / `outerHTML` / `document.write` / `eval` / `new Function`.**
  Saara dynamic DOM `MS.el()` se banta hai, jo `textContent` aur `setAttribute` use karta hai.
  Isse stored/reflected XSS ka rasta band hota hai.
- **No inline event handlers** (`onclick` etc.) — sirf `addEventListener`.
- **Strict validation:**
  - Product IDs: `^[a-z0-9-]{1,40}$`
  - Prices/stock integers, defined range me
  - Image names: `^[a-z0-9-]+\.svg$`
  - Naam/address: allowed characters + Devanagari range
  - Mobile: `^[6-9]\d{9}$`, PIN: `^[1-9]\d{5}$`
  - Text fields control characters strip + length cap
- **Catalogue freeze:** products aur coupons load par validate hokar `Object.freeze` ho jaate hain.
- **localStorage ko trust nahi kiya jaata:** cart har read par catalogue ke against re-validate hota hai.
  Tampered price, fake product ID, negative qty ya huge qty discard/cap ho jaati hai.
- **Quantity limits:** max 20 per product, max 30 line items.
- **No third-party scripts, analytics, pixels, cookies ya external fonts.**
- **No card / UPI PIN / OTP collection.** Payment sirf delivery ke waqt.
- **Safe external links:** WhatsApp/Maps links par `rel="noopener noreferrer"`.
- **Security headers** ke ready-made files: `_headers` (Netlify/Cloudflare) aur `.htaccess` (Apache).
- **Referrer policy** meta tag har page par.

### Jo nahi hai (jaan-boojhkar)

- Koi backend nahi — isliye **server-side validation nahi** hai. Koi bhi browser DevTools se
  localStorage badal sakta hai. Isse sirf apna hi cart badalta hai, dukaan ka data nahi.
- Koi authentication, admin panel ya database nahi.
- Koi payment gateway nahi.
- `file://` par CSP enforce nahi hota.

### Security headers

`_headers` aur `.htaccess` me yeh headers set hain:

```text
Content-Security-Policy: default-src 'self'; base-uri 'none'; object-src 'none';
  frame-ancestors 'none'; form-action 'self'; img-src 'self' data:;
  style-src 'self'; script-src 'self'; font-src 'self'; connect-src 'self';
  manifest-src 'self'; upgrade-insecure-requests
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
X-Frame-Options: DENY
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=()
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
Strict-Transport-Security: max-age=31536000   # sirf HTTPS par
```

`_headers` ko Netlify par site folder ke root me rakhein — automatically apply hota hai.
Apache hosting par `.htaccess` root me rakhein (`mod_headers` enabled hona chahiye).
GitHub Pages custom headers support nahi karta; wahan CSP `<meta>` tag se deni padegi
(note: `frame-ancestors`, `report-uri`, `sandbox` meta me kaam nahi karte).

Headers deploy ke baad <https://securityheaders.com> aur browser DevTools → Network → Response Headers
se verify karein.

---

## 6. Deployment

1. `mahesh seeds` folder ki **poori contents** upload karein (folder ke andar ki files, folder khud nahi).
2. HTTPS on karein — HSTS aur secure localStorage ke liye zaroori.
3. `_headers` (Netlify/Cloudflare Pages) ya `.htaccess` (Apache/cPanel) upload hua hai confirm karein.
4. `index.html` root par hona chahiye, warna links toot jayenge.
5. Custom 404 page abhi nahi hai — hosting par `404.html` add kar sakte hain.

Netlify par sabse aasan tareeka: folder ko <https://app.netlify.com/drop> par drag karein.
Koi build command nahi chahiye; publish directory `.` (root) hai.

---

## 7. Production checklist — live karne se pehle

Yeh sab **placeholder** hain, inhe apni asli jaankari se badlein:

- [ ] **WhatsApp number** — `js/main.js` me `MS.shop.whatsapp` (`919876543210`) aur har HTML page me
      `wa.me/919876543210` links.
- [ ] **Phone** — `MS.shop.phone` / `phoneHref`, topbar, footer aur contact page (`+91 98765 43210`).
- [ ] **Email** — `order@maheshseeds.in` (footer, contact, policies).
- [ ] **Address** — `Main Bazar Road, Shivpuri, Madhya Pradesh 473551` (footer, contact, policies, Google Maps link).
- [ ] **Dukaan ke samay** — topbar/footer/contact/policies.
- [ ] **Prices, stock, product details** — `js/data.js` (prices GST inclusive maane gaye hain).
- [ ] **Coupons aur shipping limits** — `js/data.js` + `js/cart.js` + page text + policies.
- [ ] **Policies** — `policies.html` me apni asli return window, refund timeline aur legal details;
      "Last updated" date badlein.
- [ ] **GSTIN / FSSAI / seed licence / pesticide licence** — footer ya policies me add karein (agar lagu ho).
- [ ] **Social links / Google Business** — abhi koi nahi hai.
- [ ] Har page ka `<title>`, description aur `theme-color` review karein.

### Real orders ke liye (demo se aage)

- [ ] Backend API + database (orders, stock, customers) — server par **dobara validation**.
- [ ] Payment gateway (Razorpay/Cashfree) — PCI compliance gateway par chhodein, card data kabhi store na karein.
- [ ] Order confirmation SMS/WhatsApp API (WhatsApp Business API).
- [ ] Admin panel — stock, price, order status.
- [ ] Rate limiting, CSRF protection, authentication agar accounts add karein.
- [ ] Privacy policy ko actual data practices ke hisaab se legal review karayein.
- [ ] Backups aur monitoring.

---

## 8. Customization

### Naya product add karna

`js/data.js` ke `RAW` array me ek object add karein:

```js
{
  id: 'new-seed-1',          // lowercase, digits aur hyphen only
  name: 'Product ka naam',
  local: 'Hinglish naam',
  category: 'seeds',         // seeds | fertilizers | pesticides | tools
  brand: 'Brand',
  price: 299,                // integer, MRP se chhota ya barabar
  mrp: 399,
  unit: '1 kg pack',
  stock: 25,
  rating: 4.5,
  reviews: 12,
  badge: 'New',              // optional
  img: 'new-seed-1.svg',     // assets/img/ me hona chahiye
  desc: '...',
  highlights: ['...'],
  specs: [['Variety', '...'], ['Maturity', '...']],
  usage: '...',
  safety: '...'              // pesticides ke liye zaroori
}
```

Agar object invalid hai (galat category, price < 1, MRP < price, missing image name) to woh
product silently skip ho jaata hai — isliye browser console check karein.

Naya image add karne ke liye `assets/img/` me SVG rakhein. External image URL allowed nahi hai
(validation `^[a-z0-9-]+\.svg$` hai), jo security ke liye jaan-boojhkar hai.

### Colors / design

`css/style.css` ke top par `:root` tokens hain — colors, radius, shadow, max width.
Ek jagah badalne se poori site update ho jaati hai.

### Storage keys

| Key | Kya rakhta hai |
| --- | --- |
| `ms_cart_v1` | `[{ id, qty }]` — re-validated on every read |
| `ms_coupon_v1` | Applied coupon code |
| `ms_orders_v1` | Last 20 orders (demo only, browser me) |
| `ms_recent_v1` | Last 8 viewed products |

Browser data clear karne par yeh sab delete ho jaata hai. Koi data server par nahi jaata.

---

## 9. Browser support

Chrome, Edge, Firefox, Safari ke current versions. ES5-style JS (const/let + arrow functions
kahin-kahin hain) — IE11 support nahi hai. `Intl.NumberFormat`, `URLSearchParams`,
`localStorage` aur `Element.closest` zaroori hain.

---

## 10. Credits

- Sabhi product images, hero illustration, logo aur favicon original SVGs hain (is project ke liye banaye gaye).
- Fonts system fonts hain — koi external download nahi.
- Icons inline SVG ya Unicode emoji hain.

---

## 11. Known limitations

- Search aur filters client-side hain — catalogue badhne par server-side search better hoga.
- Coupon codes publicly visible hain (JS file me) — production me server-side coupon validation chahiye.
- Order status tracking nahi hai.
- Newsletter form sirf success toast dikhata hai; koi email service connected nahi.
- Contact form data kahin save nahi karta — sirf WhatsApp message banata hai.
- `policies.html` sample text hai; apne business ke hisaab se legal review karayein.
