<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Free Fire MAX — Guild Admin</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <main class="wrap">
    <header class="site-header">
      <h1>Free Fire MAX — Guild Admin</h1>
      <div id="userArea"></div>
    </header>

    <!-- Login -->
    <section id="loginView" class="card center-card">
      <h2>Admin Login</h2>
      <input id="username" placeholder="Username" />
      <input id="password" placeholder="Password" type="password" />
      <button id="btnLogin" class="btn primary">Login</button>
      <p class="muted">Use admin credentials seeded in backend. Token stored locally.</p>
    </section>

    <!-- Dashboard -->
    <section id="dashboardView" class="hidden">
      <div class="layout">
        <div class="col left">
          <div class="card">
            <h3>Add Player</h3>
            <input id="p_name" placeholder="Player Name" />
            <input id="p_guildId" placeholder="Player Guild ID" />
            <input id="p_joinDate" type="date" />
            <input id="p_photo" type="file" accept="image/*" />
            <div class="row">
              <button id="btnAdd" class="btn primary">Add Player</button>
              <button id="btnExport" class="btn">Export Excel</button>
            </div>
            <div id="addMsg" class="muted"></div>
          </div>
        </div>

        <div class="col right">
          <div class="card">
            <h3>Guild Roster</h3>
            <div id="stats" class="stats">
              <div>Total: <span id="totalCount">0</span></div>
              <div>Latest: <span id="latestName">—</span></div>
            </div>
            <div id="roster"></div>
          </div>
        </div>
      </div>
    </section>

    <footer class="muted center">Built for Free Fire MAX guild management • Host on GitHub Pages</footer>
  </main>

  <script src="app.js"></script>
</body>
</html>
