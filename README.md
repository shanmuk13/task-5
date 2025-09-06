# task-5
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Capstone — Single File E‑Commerce App</title>
  <meta name="description" content="A responsive, cross-browser single-file demo e-commerce app (HTML/CSS/JS) — cart, search, filters, lazy images, localStorage persistence." />

  <!-- Critical CSS (kept compact for single-file demo) -->
  <style>
    :root{
      --bg:#f7f7fb;--card:#fff;--accent:#0b79d0;--muted:#6b7280;--glass:rgba(255,255,255,0.6);
      --radius:12px;--shadow: 0 6px 18px rgba(16,24,40,0.08);
      --maxw:1200px;--gap:16px;
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue",Arial;line-height:1.35;color:#0f172a;background:linear-gradient(180deg,var(--bg),#fff)}
    .app{max-width:var(--maxw);margin:28px auto;padding:20px}

    header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:18px}
    .brand{display:flex;align-items:center;gap:12px}
    .logo{width:46px;height:46px;border-radius:10px;background:linear-gradient(135deg,var(--accent),#6dd3ff);display:grid;place-items:center;color:white;font-weight:700}
    h1{font-size:1.05rem;margin:0}
    p.lead{margin:0;color:var(--muted);font-size:0.9rem}

    /* Controls */
    .controls{display:flex;gap:12px;flex-wrap:wrap;align-items:center}
    .search{display:flex;align-items:center;background:var(--card);padding:8px;border-radius:10px;box-shadow:var(--shadow);min-width:220px}
    .search input{border:0;outline:0;background:transparent;font-size:0.95rem}
    .select, .price{background:var(--card);padding:8px;border-radius:10px;box-shadow:var(--shadow)}
    .btn{background:var(--accent);color:white;padding:10px 14px;border-radius:10px;border:0;cursor:pointer}

    /* Layout */
    .grid{display:grid;grid-template-columns:1fr 320px;gap:var(--gap)}
    .catalog{background:transparent}
    .product-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:var(--gap)}
    .card{background:var(--card);border-radius:var(--radius);overflow:hidden;box-shadow:var(--shadow);display:flex;flex-direction:column}
    .card img{width:100%;height:180px;object-fit:cover;display:block}
    .card-body{padding:12px;display:flex;flex-direction:column;gap:8px}
    .title{font-weight:600;font-size:0.95rem}
    .price{color:var(--accent);font-weight:700}
    .meta{color:var(--muted);font-size:0.85rem}
    .card-actions{margin-top:auto;display:flex;gap:8px}

    aside{position:sticky;top:20px}
    .cart{background:var(--card);padding:12px;border-radius:12px;box-shadow:var(--shadow)}
    .cart h3{margin:0 0 8px 0}
    .cart-items{max-height:320px;overflow:auto;margin-bottom:8px}
    .cart-row{display:flex;justify-content:space-between;gap:8px;padding:8px 0;border-bottom:1px dashed #eee}
    .cart-row small{color:var(--muted)}
    .checkout{display:flex;justify-content:space-between;align-items:center;margin-top:8px}

    /* Responsive */
    @media (max-width:1000px){.product-grid{grid-template-columns:repeat(2,1fr)}.grid{grid-template-columns:1fr 340px}}
    @media (max-width:720px){.product-grid{grid-template-columns:1fr}.grid{grid-template-columns:1fr}.brand h1{display:none}}

    /* Accessibility helpers */
    .sr-only{position:absolute!important;height:1px;width:1px;overflow:hidden;clip:rect(1px,1px,1px,1px);white-space:nowrap}

    /* Tiny utility classes */
    .muted{color:var(--muted)}
    .flex{display:flex;gap:8px;align-items:center}

  </style>
</head>
<body>
  <div class="app" id="app">
    <header>
      <div class="brand">
        <div class="logo" aria-hidden>Ce</div>
        <div>
          <h1>Capstone Store — Single File Demo</h1>
          <p class="lead">Responsive • Accessible • Lazy images • Local cart persistence</p>
        </div>
      </div>
      <div class="controls" role="region" aria-label="shop controls">
        <div class="search" title="Search products">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" aria-hidden><path d="M21 21l-4.35-4.35" stroke="#94a3b8" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg>
          <input id="search" placeholder="Search products..." aria-label="Search products" />
        </div>
        <div class="select">
          <label for="category" class="sr-only">Category</label>
          <select id="category" aria-label="Filter by category">
            <option value="all">All categories</option>
          </select>
        </div>
        <div class="price">
          <label for="maxprice" class="sr-only">Max price</label>
          <input id="maxprice" type="range" min="0" max="500" value="500" />
        </div>
        <button id="clear" class="btn" title="Clear filters">Clear</button>
      </div>
    </header>

    <main class="grid">
      <section class="catalog" aria-label="Product catalog">
        <div class="product-grid" id="productGrid" aria-live="polite"></div>
      </section>

      <aside>
        <div class="cart" aria-label="Shopping cart">
          <h3>Cart <small id="cartCount">(0)</small></h3>
          <div class="cart-items" id="cartItems"></div>
          <div class="checkout">
            <div>
              <div class="muted">Total</div>
              <div id="cartTotal">₹0.00</div>
            </div>
            <button id="checkoutBtn" class="btn">Checkout</button>
          </div>
        </div>
      </aside>
    </main>

    <footer style="margin-top:18px;color:var(--muted);font-size:0.85rem">Tip: This single-file demo is intended as a starting point. Replace images and data with your own API for production.</footer>
  </div>

  <!-- App logic: keep efficient, avoid heavy libraries for performance -->
  <script>
    // ---------- Data (sample) ----------
    const PRODUCTS = [
      {id:1,title:'Classic Watch',price:199,cat:'Accessories',img:'https://picsum.photos/id/1011/600/400'},
      {id:2,title:'Leather Wallet',price:79,cat:'Accessories',img:'https://picsum.photos/id/1025/600/400'},
      {id:3,title:'Running Shoes',price:259,cat:'Footwear',img:'https://picsum.photos/id/1020/600/400'},
      {id:4,title:'Denim Jacket',price:349,cat:'Clothing',img:'https://picsum.photos/id/1005/600/400'},
      {id:5,title:'Sunglasses',price:129,cat:'Accessories',img:'https://picsum.photos/id/1012/600/400'},
      {id:6,title:'Backpack',price:89,cat:'Bags',img:'https://picsum.photos/id/1018/600/400'},
      {id:7,title:'Formal Shoes',price:199,cat:'Footwear',img:'https://picsum.photos/id/1027/600/400'},
      {id:8,title:'Graphic Tee',price:39,cat:'Clothing',img:'https://picsum.photos/id/1003/600/400'},
      {id:9,title:'Beanie',price:19,cat:'Accessories',img:'https://picsum.photos/id/1001/600/400'}
    ];

    // ---------- App state & helpers ----------
    const state = {
      products: PRODUCTS.slice(),
      filtered: [],
      cart: {},
      maxPrice: 500,
      q: '',
      category: 'all'
    };

    // Price formatter
    const fmt = v => new Intl.NumberFormat('en-IN',{style:'currency',currency:'INR',maximumFractionDigits:0}).format(v);

    // ---------- Persistence ----------
    const CART_KEY = 'capstone_cart_v1';
    function loadCart(){
      try{const raw=localStorage.getItem(CART_KEY); if(raw) state.cart = JSON.parse(raw);}catch(e){console.warn('cart load failed',e)}
    }
    function saveCart(){ try{localStorage.setItem(CART_KEY,JSON.stringify(state.cart))}catch(e){console.warn('cart save failed',e)} }

    // ---------- UI Rendering ----------
    const productGrid = document.getElementById('productGrid');
    const cartItemsEl = document.getElementById('cartItems');
    const cartCountEl = document.getElementById('cartCount');
    const cartTotalEl = document.getElementById('cartTotal');

    function renderProducts(list){
      productGrid.innerHTML = '';
      const frag = document.createDocumentFragment();
      list.forEach(p=>{
        const el = document.createElement('article'); el.className='card';
        el.innerHTML = `
          <img data-src="${p.img}" alt="${p.title}" class="lazy" loading="lazy">
          <div class="card-body">
            <div class="title">${escapeHtml(p.title)}</div>
            <div class="meta">${escapeHtml(p.cat)}</div>
            <div style="display:flex;justify-content:space-between;align-items:center">
              <div class="price">${fmt(p.price)}</div>
            </div>
            <div class="card-actions">
              <button class="btn add" data-id="${p.id}" aria-label="Add ${escapeHtml(p.title)} to cart">Add</button>
              <button class="muted" data-id="${p.id}" aria-label="View details for ${escapeHtml(p.title)}">Details</button>
            </div>
          </div>`;
        frag.appendChild(el);
      });
      productGrid.appendChild(frag);
      observeLazyImages();
    }

    function renderCart(){
      cartItemsEl.innerHTML = '';
      const keys = Object.keys(state.cart);
      if(keys.length===0){cartItemsEl.innerHTML='<div class="muted">Cart is empty</div>'}
      let total = 0;
      keys.forEach(id=>{
        const qty = state.cart[id];
        const p = state.products.find(x=>x.id==id);
        if(!p) return;
        total += p.price*qty;
        const row = document.createElement('div'); row.className='cart-row';
        row.innerHTML = `<div><strong>${escapeHtml(p.title)}</strong><br><small>${fmt(p.price)} × ${qty}</small></div>
                         <div style="text-align:right"><div>₹${p.price*qty}</div>
                         <div class="flex"><button data-id="${id}" class="muted dec">−</button><button data-id="${id}" class="muted inc">+</button><button data-id="${id}" class="muted rem">Remove</button></div></div>`;
        cartItemsEl.appendChild(row);
      });
      cartCountEl.textContent = (${keys.length});
      cartTotalEl.textContent = fmt(total);
      attachCartEvents();
    }

    // ---------- Events ----------
    function attachMainEvents(){
      document.getElementById('search').addEventListener('input', debounce(e=>{state.q=e.target.value; applyFilters()},250));
      document.getElementById('category').addEventListener('change', e=>{state.category=e.target.value; applyFilters()});
      document.getElementById('maxprice').addEventListener('input', e=>{state.maxPrice=+e.target.value; applyFilters()});
      document.getElementById('clear').addEventListener('click', ()=>{document.getElementById('search').value='';document.getElementById('category').value='all';document.getElementById('maxprice').value=500;state.q='';state.category='all';state.maxPrice=500;applyFilters()});
      productGrid.addEventListener('click', e=>{
        if(e.target.matches('.add')) addToCart(e.target.dataset.id);
      });
      document.getElementById('checkoutBtn').addEventListener('click', ()=>{
        alert('Demo checkout — implement real payment in production.');
      });
    }

    function attachCartEvents(){
      cartItemsEl.querySelectorAll('.inc').forEach(b=>b.onclick=()=>{changeQty(b.dataset.id,1)});
      cartItemsEl.querySelectorAll('.dec').forEach(b=>b.onclick=()=>{changeQty(b.dataset.id,-1)});
      cartItemsEl.querySelectorAll('.rem').forEach(b=>{b.onclick=()=>{removeFromCart(b.dataset.id)}});
    }

    function addToCart(id){ state.cart[id] = (state.cart[id]||0)+1; saveCart(); renderCart(); }
    function changeQty(id,delta){ state.cart[id] = (state.cart[id]||0)+delta; if(state.cart[id] <= 0) delete state.cart[id]; saveCart(); renderCart(); }
    function removeFromCart(id){ delete state.cart[id]; saveCart(); renderCart(); }

    // ---------- Filtering & Search ----------
    function applyFilters(){
      const q = state.q.trim().toLowerCase();
      let list = state.products.filter(p=>p.price <= state.maxPrice);
      if(state.category !== 'all') list = list.filter(p=>p.cat === state.category);
      if(q) list = list.filter(p=> (p.title + ' ' + p.cat).toLowerCase().includes(q));
      state.filtered = list;
      renderProducts(list);
    }

    // ---------- Utilities ----------
    function debounce(fn,wait){let t;return function(...args){clearTimeout(t);t=setTimeout(()=>fn.apply(this,args),wait)}}
    function escapeHtml(s){return String(s).replace(/[&<>"']/g, c=>({"&":"&amp;","<":"&lt;",">":"&gt;","\"":"&quot;","'":"&#39;"}[c]))}

    // ---------- Lazy load implementation (IntersectionObserver) ----------
    let io = null;
    function observeLazyImages(){
      const lazy = Array.from(document.querySelectorAll('img.lazy'));
      if("IntersectionObserver" in window){
        if(!io) io = new IntersectionObserver((entries,obs)=>{
          entries.forEach(en=>{
            if(en.isIntersecting){ const img = en.target; img.src = img.dataset.src; img.classList.remove('lazy'); obs.unobserve(img); }
          });
        },{rootMargin:'200px'});
        lazy.forEach(img=>io.observe(img));
      } else {
        // fallback
        lazy.forEach(img=>img.src = img.dataset.src);
      }
    }

    // ---------- Bootstrapping ----------
    function populateCategories(){
      const cats = Array.from(new Set(state.products.map(p=>p.cat)));
      const sel = document.getElementById('category');
      cats.forEach(c=>{ const o = document.createElement('option'); o.value=c; o.textContent = c; sel.appendChild(o); });
    }

    // ---------- Init ----------
    (function init(){
      // Performance: small initial render, avoid heavy DOM thrash
      loadCart();
      state.products = PRODUCTS; // in production, fetch from API with proper caching headers
      populateCategories();
      state.maxPrice = 500;
      applyFilters();
      renderCart();
      attachMainEvents();

      // Accessibility: keyboard focus styles
      document.addEventListener('keyup', e=>{ if(e.key==='/' ){ document.getElementById('search').focus(); e.preventDefault(); }});

      // Quick hydration: defer any non-essential work
      window.requestIdleCallback && requestIdleCallback(()=>{
        // example: prefetch images low priority
        state.products.slice(0,3).forEach(p=>{ const i=new Image(); i.src = p.img; });
      });
    })();

    // ---------- Small notes for developers (inside file for clarity) ----------
    /*
      Production notes:
      - Replace inline data with API calls (use fetch with ETag caching)
      - Minify CSS/JS during build and use HTTP/2 or HTTP/3
      - Serve optimized images (WebP/AVIF) and multiple sizes via srcset
      - Add server-side rendering (SSR) for initial HTML if SEO matters
      - Add tests for key flows (add to cart, filters, persistence)
      - For cross-browser: test on latest Chrome/Firefox/Safari and mobile browsers
    */
  </script>
</body>
</html>
