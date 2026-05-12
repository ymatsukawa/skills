<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>How authentication flows through birchline/web</title>

<!-- Fonts: Inter (body), Noto Sans JP (Japanese fallback), JetBrains Mono (code) -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;600&family=Noto+Sans+JP:wght@400;500;600;700&display=swap" rel="stylesheet">

<style>
  :root {
    /* GitHub-like palette (light) */
    --bg:           #ffffff;
    --bg-subtle:    #f6f8fa;
    --bg-code:      #f6f8fa;
    --bg-codeblock: #f6f8fa;
    --border:       #d0d7de;
    --border-muted: #d8dee4;
    --fg:           #1f2328;
    --fg-muted:     #59636e;
    --fg-subtle:    #6e7781;
    --accent:       #0969da;
    --accent-bg:    #ddf4ff;
    --highlight-bg: #fff8c5;     /* GitHub-style soft yellow highlight */
    --highlight-bd: #d4a72c;

    --sans:   'Inter', 'Noto Sans JP', system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
    --mono:   'JetBrains Mono', ui-monospace, 'SF Mono', Menlo, Monaco, monospace;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  html { -webkit-text-size-adjust: 100%; }

  body {
    background: var(--bg);
    color: var(--fg);
    font-family: var(--sans);
    font-size: 15px;
    line-height: 1.65;
    padding: 48px 24px 80px;
    font-feature-settings: 'cv02', 'cv03', 'cv04', 'cv11';
  }

  .page {
    max-width: 880px;
    margin: 0 auto;
  }

  /* ---------- Header ---------- */
  header {
    margin-bottom: 24px;
    padding-bottom: 20px;
    border-bottom: 1px solid var(--border-muted);
  }

  .repo-line {
    font-family: var(--mono);
    font-size: 12.5px;
    color: var(--fg-subtle);
    margin-bottom: 10px;
  }

  h1 {
    font-family: var(--sans);
    font-weight: 600;
    font-size: 30px;
    line-height: 1.25;
    color: var(--fg);
    margin-bottom: 18px;
    letter-spacing: -0.01em;
  }

  /* ---------- Summary (now a list) ---------- */
  .summary-list {
    list-style: disc;
    padding-left: 22px;
    max-width: 760px;
  }
  .summary-list li {
    font-size: 15px;
    color: var(--fg);
    margin-bottom: 8px;
  }
  .summary-list li:last-child { margin-bottom: 0; }

  h2 {
    font-family: var(--sans);
    font-weight: 600;
    font-size: 20px;
    color: var(--fg);
    margin: 40px 0 16px;
    letter-spacing: -0.005em;
  }
  h2:first-of-type { margin-top: 8px; }

  /* ---------- Inline code ---------- */
  code {
    font-family: var(--mono);
    font-size: 0.85em;
    background: var(--bg-code);
    border: 1px solid var(--border-muted);
    border-radius: 6px;
    padding: 0.1em 0.4em;
    color: var(--fg);
  }

  /* ---------- Diagram ---------- */
  .diagram-panel {
    border: 1px solid var(--border);
    border-radius: 8px;
    background: var(--bg);
    padding: 20px;
    overflow-x: auto;
  }

  svg.flow { display: block; max-width: 100%; height: auto; }
  .flow text {
    font-family: var(--mono);
    font-size: 12px;
    fill: var(--fg);
  }
  .flow .sub { font-size: 10px; fill: var(--fg-subtle); }
  .flow .box { fill: var(--bg); stroke: var(--border); stroke-width: 1.5; }
  .flow .box.hot {
    fill: var(--highlight-bg);
    stroke: var(--highlight-bd);
  }
  .flow .arrow { stroke: var(--fg-subtle); stroke-width: 1.5; fill: none; }

  /* ---------- Walkthrough ---------- */
  .step {
    display: grid;
    grid-template-columns: 36px 1fr;
    gap: 16px;
    padding: 20px 0;
    border-bottom: 1px solid var(--border-muted);
  }
  .step:last-of-type { border-bottom: none; }

  .badge {
    width: 28px;
    height: 28px;
    border-radius: 50%;
    background: var(--bg-subtle);
    border: 1px solid var(--border);
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: var(--mono);
    font-weight: 600;
    color: var(--fg);
    font-size: 13px;
    margin-top: 2px;
  }
  .step.hot .badge {
    background: var(--highlight-bg);
    border-color: var(--highlight-bd);
    color: var(--fg);
  }

  .step-loc {
    font-family: var(--mono);
    font-size: 13px;
    color: var(--fg);
    margin-bottom: 10px;
  }
  .step-loc .range { color: var(--fg-subtle); }

  .step-list {
    list-style: disc;
    padding-left: 22px;
    margin-bottom: 6px;
  }
  .step-list li {
    font-size: 14.5px;
    color: var(--fg);
    margin-bottom: 6px;
  }
  .step-list li:last-child { margin-bottom: 0; }

  /* ---------- Snippet (collapsible) ---------- */
  details.snippet { margin-top: 10px; }
  details.snippet summary {
    list-style: none;
    cursor: pointer;
    font-size: 12.5px;
    color: var(--accent);
    font-family: var(--mono);
    display: inline-flex;
    align-items: center;
    gap: 6px;
    padding: 4px 0;
    user-select: none;
  }
  details.snippet summary:hover { text-decoration: underline; }
  details.snippet summary::-webkit-details-marker { display: none; }
  details.snippet summary::before {
    content: "▸";
    font-size: 10px;
    transition: transform 0.15s ease;
    color: var(--fg-subtle);
  }
  details.snippet[open] summary::before { transform: rotate(90deg); }

  pre.code {
    background: var(--bg-codeblock);
    color: var(--fg);
    font-family: var(--mono);
    font-size: 12.5px;
    line-height: 1.7;
    border: 1px solid var(--border-muted);
    border-radius: 6px;
    padding: 14px 16px;
    overflow-x: auto;
    margin-top: 10px;
  }
  /* GitHub-like syntax tokens */
  pre.code .dim { color: var(--fg-subtle); font-style: italic; }   /* comments */
  pre.code .kw  { color: #cf222e; }                                /* keywords */
  pre.code .str { color: #0a3069; }                                /* strings */
</style>
</head>
<body>
<div class="page">

  <header>
    <h1>How authentication flows through the codebase</h1>
    <ul class="summary-list">
      <li>Birchline uses cookie-based sessions: the browser never holds a bearer token directly.</li>
      <li>Every authenticated request hits <code>/api/*</code>, passes through a single <code>verifyToken()</code> middleware, and resolves to a <code>Session</code> row that downstream handlers read off <code>req.ctx</code>.</li>
      <li>The middleware is the only place that talks to the session store, which is the only place that talks to the <code>sessions</code> table — so there's exactly one trust boundary to reason about.</li>
    </ul>
  </header>

  <main>
    <h2>Request path</h2>
    <div class="diagram-panel">
      <svg class="flow" viewBox="0 0 720 280" width="720" height="280" role="img" aria-label="Authentication flow diagram">
        <defs>
          <marker id="arrowHead" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
            <path d="M 0 0 L 10 5 L 0 10 z" fill="#6e7781"></path>
          </marker>
        </defs>

        <!-- Row 1 -->
        <g>
          <rect class="box" x="30" y="40" width="150" height="64" rx="8"></rect>
          <text x="105" y="68" text-anchor="middle">Browser</text>
          <text class="sub" x="105" y="84" text-anchor="middle">birchline.app</text>
        </g>
        <g>
          <rect class="box" x="245" y="40" width="150" height="64" rx="8"></rect>
          <text x="320" y="68" text-anchor="middle">/api/session</text>
          <text class="sub" x="320" y="84" text-anchor="middle">route handler</text>
        </g>
        <g>
          <rect class="box hot" x="460" y="40" width="180" height="64" rx="8"></rect>
          <text x="550" y="68" text-anchor="middle">verifyToken()</text>
          <text class="sub" x="550" y="84" text-anchor="middle">middleware/auth.ts</text>
        </g>

        <!-- Row 2 -->
        <g>
          <rect class="box" x="460" y="170" width="180" height="64" rx="8"></rect>
          <text x="550" y="198" text-anchor="middle">SessionStore</text>
          <text class="sub" x="550" y="214" text-anchor="middle">lib/sessionStore.ts</text>
        </g>
        <g>
          <rect class="box" x="245" y="170" width="150" height="64" rx="8"></rect>
          <text x="320" y="198" text-anchor="middle">Postgres</text>
          <text class="sub" x="320" y="214" text-anchor="middle">sessions table</text>
        </g>

        <!-- Arrows -->
        <line class="arrow" x1="180" y1="72" x2="245" y2="72" marker-end="url(#arrowHead)"></line>
        <line class="arrow" x1="395" y1="72" x2="460" y2="72" marker-end="url(#arrowHead)"></line>
        <line class="arrow" x1="550" y1="104" x2="550" y2="170" marker-end="url(#arrowHead)"></line>
        <line class="arrow" x1="460" y1="202" x2="395" y2="202" marker-end="url(#arrowHead)"></line>

        <text class="sub" x="212" y="62" text-anchor="middle">cookie</text>
        <text class="sub" x="560" y="140" text-anchor="start">lookup</text>
      </svg>
    </div>

    <h2>Callstack walkthrough</h2>

    <!-- Step 1 -->
    <div class="step">
      <div class="badge">1</div>
      <div class="step-body">
        <div class="step-loc">src/app/providers/AuthProvider.tsx<span class="range"> :22-48</span></div>
        <ul class="step-list">
          <li>On mount, the React provider issues a <code>GET /api/session</code> with <code>credentials: 'include'</code> so the <code>fw_sid</code> cookie rides along.</li>
          <li>The response either hydrates <code>currentUser</code> into context or leaves it <code>null</code>, which the router treats as "show the sign-in screen".</li>
        </ul>
        <details class="snippet">
          <summary>show source</summary>
<pre class="code"><span class="dim">// src/app/providers/AuthProvider.tsx</span>
<span class="kw">export function</span> AuthProvider({ children }: Props) {
  <span class="kw">const</span> [user, setUser] = useState&lt;User | <span class="kw">null</span>&gt;(<span class="kw">null</span>);

  useEffect(() =&gt; {
    fetch(<span class="str">'/api/session'</span>, { credentials: <span class="str">'include'</span> })
      .then(r =&gt; r.ok ? r.json() : <span class="kw">null</span>)
      .then(setUser);
  }, []);

  <span class="kw">return</span> &lt;AuthCtx.Provider value={{ user }}&gt;{children}&lt;/AuthCtx.Provider&gt;;
}</pre>
        </details>
      </div>
    </div>

    <!-- Step 2 -->
    <div class="step">
      <div class="badge">2</div>
      <div class="step-body">
        <div class="step-loc">src/server/routes/session.ts<span class="range"> :9-27</span></div>
        <ul class="step-list">
          <li>The route itself is thin: it just returns whatever <code>req.ctx.session</code> the middleware attached.</li>
          <li>If the middleware short-circuited with a 401, this handler never runs — so there's no auth logic duplicated here.</li>
        </ul>
        <details class="snippet">
          <summary>show source</summary>
<pre class="code"><span class="dim">// src/server/routes/session.ts</span>
router.get(<span class="str">'/session'</span>, verifyToken, (req, res) =&gt; {
  <span class="kw">const</span> { session } = req.ctx;
  res.json({
    id:    session.userId,
    email: session.email,
    role:  session.role,
    exp:   session.expiresAt,
  });
});</pre>
        </details>
      </div>
    </div>

    <!-- Step 3 -->
    <div class="step hot">
      <div class="badge">3</div>
      <div class="step-body">
        <div class="step-loc">src/middleware/auth.ts<span class="range"> :14-31</span></div>
        <ul class="step-list">
          <li>This is the trust boundary.</li>
          <li><code>verifyToken</code> reads the signed <code>fw_sid</code> cookie, asks <code>SessionStore</code> to resolve it, and either populates <code>req.ctx.session</code> or responds 401.</li>
          <li>Every protected route in the app is mounted behind this function, so changing its behaviour changes auth globally.</li>
        </ul>
        <details class="snippet">
          <summary>show source</summary>
<pre class="code"><span class="dim">// src/middleware/auth.ts</span>
<span class="kw">export async function</span> verifyToken(req, res, next) {
  <span class="kw">const</span> raw = req.signedCookies[<span class="str">'fw_sid'</span>];
  <span class="kw">if</span> (!raw) <span class="kw">return</span> res.status(401).end();

  <span class="kw">const</span> session = <span class="kw">await</span> SessionStore.get(raw);
  <span class="kw">if</span> (!session || session.expiresAt &lt; Date.now()) {
    <span class="kw">return</span> res.status(401).end();
  }

  req.ctx = { session };
  next();
}</pre>
        </details>
      </div>
    </div>

    <!-- Step 4 -->
    <div class="step">
      <div class="badge">4</div>
      <div class="step-body">
        <div class="step-loc">src/lib/sessionStore.ts<span class="range"> :8-52</span></div>
        <ul class="step-list">
          <li><code>SessionStore</code> is a small read-through cache: it checks an in-process LRU first, then falls back to Postgres.</li>
          <li>Writes (<code>create</code>, <code>revoke</code>) always go straight to the DB and invalidate the cache entry so other workers don't serve a stale session.</li>
        </ul>
        <details class="snippet">
          <summary>show source</summary>
<pre class="code"><span class="dim">// src/lib/sessionStore.ts</span>
<span class="kw">const</span> cache = <span class="kw">new</span> LRU&lt;string, Session&gt;({ max: 5000, ttl: 60_000 });

<span class="kw">export const</span> SessionStore = {
  <span class="kw">async</span> get(id: string) {
    <span class="kw">const</span> hit = cache.get(id);
    <span class="kw">if</span> (hit) <span class="kw">return</span> hit;
    <span class="kw">const</span> row = <span class="kw">await</span> db.one(SELECT_SESSION, [id]);
    <span class="kw">if</span> (row) cache.set(id, row);
    <span class="kw">return</span> row ?? <span class="kw">null</span>;
  },
  <span class="dim">/* create, revoke, touch ... */</span>
};</pre>
        </details>
      </div>
    </div>

    <!-- Step 5 -->
    <div class="step">
      <div class="badge">5</div>
      <div class="step-body">
        <div class="step-loc">db/migrations/004_sessions.sql<span class="range"> :1-18</span></div>
        <ul class="step-list">
          <li>The <code>sessions</code> table is keyed on a random 32-byte id (the cookie value) with a covering index on <code>user_id</code> for "sign out everywhere".</li>
          <li>Expiry is enforced both here (<code>expires_at</code>) and again in the middleware as defence in depth.</li>
        </ul>
        <details class="snippet">
          <summary>show source</summary>
<pre class="code"><span class="dim">-- db/migrations/004_sessions.sql</span>
<span class="kw">create table</span> sessions (
  id          <span class="kw">text primary key</span>,
  user_id     <span class="kw">uuid not null references</span> users(id),
  created_at  <span class="kw">timestamptz default</span> now(),
  expires_at  <span class="kw">timestamptz not null</span>,
  ip          <span class="kw">inet</span>,
  user_agent  <span class="kw">text</span>
);
<span class="kw">create index</span> sessions_user_id_idx <span class="kw">on</span> sessions(user_id);</pre>
        </details>
      </div>
    </div>
  </main>

</div>

<script>
  // Keep at most one snippet open at a time so the walkthrough stays scannable.
  var snippets = document.querySelectorAll('details.snippet');
  snippets.forEach(function (d) {
    d.addEventListener('toggle', function () {
      if (!d.open) return;
      snippets.forEach(function (other) {
        if (other !== d) other.open = false;
      });
    });
  });
</script>

</body>
</html>
