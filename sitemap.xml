(function () {
  var buttons = Array.prototype.slice.call(document.querySelectorAll('.lang-btn'));
  var nodes = Array.prototype.slice.call(document.querySelectorAll('[data-en]'));
  var themeBtn = document.getElementById('theme-btn');
  var root = document.documentElement;
  var currentLang = 'en';

  var THEME_LABEL = {
    en: { light: 'Switch to light theme', dark: 'Switch to dark theme' },
    sk: { light: 'Prepnúť na svetlú tému', dark: 'Prepnúť na tmavú tému' }
  };

  function labelTheme() {
    if (!themeBtn) return;
    var next = root.getAttribute('data-ft-theme') === 'light' ? 'dark' : 'light';
    var text = (THEME_LABEL[currentLang] || THEME_LABEL.en)[next];
    themeBtn.setAttribute('aria-label', text);
    themeBtn.setAttribute('title', text);
  }

  function setTheme(theme) {
    if (theme === 'light') root.setAttribute('data-ft-theme', 'light');
    else root.removeAttribute('data-ft-theme');
    try { localStorage.setItem('ft-theme', theme); } catch (e) {}
    labelTheme();
  }

  try {
    if (localStorage.getItem('ft-theme') === 'light') root.setAttribute('data-ft-theme', 'light');
  } catch (e) {}

  if (themeBtn) {
    themeBtn.addEventListener('click', function () {
      setTheme(root.getAttribute('data-ft-theme') === 'light' ? 'dark' : 'light');
    });
  }

  function apply(lang) {
    currentLang = lang;
    nodes.forEach(function (el) {
      var v = el.getAttribute('data-' + lang);
      if (v !== null) el.textContent = v;
    });
    buttons.forEach(function (b) {
      b.setAttribute('aria-pressed', String(b.getAttribute('data-lang') === lang));
    });
    root.lang = lang;
    labelTheme();
  }

  buttons.forEach(function (b) {
    b.addEventListener('click', function () { apply(b.getAttribute('data-lang')); });
  });

  /* English always loads first, whoever the visitor is — Slovak is one click away. */
  apply('en');

  var yr = document.getElementById('yr');
  if (yr) yr.textContent = String(new Date().getFullYear());

  /* highlight the section currently in view */
  var links = Array.prototype.slice.call(document.querySelectorAll('.nav a'));
  var byId = {};
  var sections = links.map(function (a) {
    var el = document.querySelector(a.getAttribute('href'));
    if (el) byId[el.id] = a;
    return el;
  }).filter(Boolean);

  if (sections.length && 'IntersectionObserver' in window) {
    var visible = {};
    var io = new IntersectionObserver(function (entries) {
      entries.forEach(function (e) { visible[e.target.id] = e.isIntersecting; });
      var current = null;
      for (var i = 0; i < sections.length; i++) {
        if (visible[sections[i].id]) { current = sections[i].id; break; }
      }
      links.forEach(function (a) { a.classList.remove('active'); });
      if (current && byId[current]) byId[current].classList.add('active');
    }, { rootMargin: '-45% 0px -50% 0px' });
    sections.forEach(function (s) { io.observe(s); });
  }
})();
/* Live journey: contacts enter, split on a decision, wait, and get sent to.
   Pauses when off-screen, when the tab is hidden, and on reduced-motion. */
(function () {
  var canvas = document.getElementById('journey');
  if (!canvas || !canvas.getContext) return;

  var ctx  = canvas.getContext('2d');
  var root = document.documentElement;
  var mq   = window.matchMedia('(prefers-reduced-motion: reduce)');

  var TOP = [[0.04,0.52],[0.26,0.52],[0.43,0.29],[0.62,0.29],[0.80,0.52],[1.04,0.52]];
  var BOT = [[0.04,0.52],[0.26,0.52],[0.43,0.77],[0.60,0.77],[0.80,0.52],[1.04,0.52]];
  var NODES = [
    {p:[0.04,0.52], k:'dot'},    {p:[0.26,0.52], k:'diamond'},
    {p:[0.43,0.29], k:'ring'},   {p:[0.62,0.29], k:'ring'},
    {p:[0.43,0.77], k:'ring'},   {p:[0.60,0.77], k:'ring'},
    {p:[0.80,0.52], k:'send'}
  ];
  var DASH = [[[0.43,0.29],[0.62,0.29]], [[0.43,0.77],[0.60,0.77]]];
  var GATES  = [0.33, 0.60];
  var SEND_AT = 0.78;
  var MAX = 46, SPAWN = 300;

  var W = 0, H = 0, dpr = 1;
  var routes = [], particles = [], pulses = [];
  var colours = {}, running = false, visible = true, raf = 0, last = 0, since = 0;

  function mid(a, b){ return {x:(a.x+b.x)/2, y:(a.y+b.y)/2}; }
  function quad(a, c, b, t){
    var u = 1 - t;
    return {x:u*u*a.x + 2*u*t*c.x + t*t*b.x, y:u*u*a.y + 2*u*t*c.y + t*t*b.y};
  }

  function build(raw){
    var p = raw.map(function(q){ return {x:q[0]*W, y:q[1]*H}; });
    var out = [p[0]];
    for (var i = 1; i < p.length - 1; i++){
      var m0 = (i === 1) ? p[0] : mid(p[i-1], p[i]);
      var m1 = mid(p[i], p[i+1]);
      for (var t = 1; t <= 22; t++) out.push(quad(m0, p[i], m1, t/22));
    }
    out.push(p[p.length-1]);
    var len = [0];
    for (var j = 1; j < out.length; j++){
      var dx = out[j].x - out[j-1].x, dy = out[j].y - out[j-1].y;
      len.push(len[j-1] + Math.sqrt(dx*dx + dy*dy));
    }
    return {pts: out, len: len, total: len[len.length-1]};
  }

  function at(route, d){
    var len = route.len, lo = 0, hi = len.length - 1;
    if (d <= 0) return route.pts[0];
    if (d >= route.total) return route.pts[hi];
    while (lo < hi - 1){
      var m = (lo + hi) >> 1;
      if (len[m] < d) lo = m; else hi = m;
    }
    var seg = len[hi] - len[lo] || 1, t = (d - len[lo]) / seg;
    var a = route.pts[lo], b = route.pts[hi];
    return {x:a.x + (b.x-a.x)*t, y:a.y + (b.y-a.y)*t};
  }

  function readColours(){
    var cs = getComputedStyle(root);
    colours = {
      line:  cs.getPropertyValue('--line').trim()      || '#22303D',
      amber: cs.getPropertyValue('--amber').trim()     || '#E39B3C',
      teal:  cs.getPropertyValue('--teal').trim()      || '#5AA9BC',
      ground:cs.getPropertyValue('--ground').trim()    || '#0A0E13',
      faint: cs.getPropertyValue('--ink-faint').trim() || '#63768A'
    };
  }

  function resize(){
    var r = canvas.getBoundingClientRect();
    if (!r.width || !r.height) return;
    dpr = Math.min(window.devicePixelRatio || 1, 2);
    W = r.width; H = r.height;
    canvas.width  = Math.round(W * dpr);
    canvas.height = Math.round(H * dpr);
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    routes = [build(TOP), build(BOT)];
  }

  function spawn(){
    if (particles.length >= MAX) return;
    var r = Math.random() < 0.56 ? 0 : 1;
    particles.push({
      r: r, d: 0,
      speed: routes[r].total / (11 + Math.random() * 5),
      gate: 0, wait: 0, sent: false,
      a: 0.45 + Math.random() * 0.55
    });
  }

  function step(dt){
    since += dt;
    while (since > SPAWN){ since -= SPAWN; spawn(); }

    for (var i = particles.length - 1; i >= 0; i--){
      var p = particles[i], route = routes[p.r];
      if (p.wait > 0){ p.wait -= dt; continue; }
      p.d += p.speed * (dt / 1000);
      var f = p.d / route.total;
      if (p.gate < GATES.length && f > GATES[p.gate]){
        p.gate++; p.wait = 500 + Math.random() * 900;
      }
      if (!p.sent && f > SEND_AT){
        p.sent = true;
        pulses.push({t:0, x:NODES[6].p[0]*W, y:NODES[6].p[1]*H});
      }
      if (p.d > route.total) particles.splice(i, 1);
    }

    for (var j = pulses.length - 1; j >= 0; j--){
      pulses[j].t += dt;
      if (pulses[j].t > 900) pulses.splice(j, 1);
    }
  }

  function draw(){
    ctx.clearRect(0, 0, W, H);

    /* structure */
    ctx.globalAlpha = 0.5;
    ctx.strokeStyle = colours.faint;
    ctx.lineWidth = 1;
    routes.forEach(function(route){
      ctx.beginPath();
      ctx.moveTo(route.pts[0].x, route.pts[0].y);
      for (var i = 1; i < route.pts.length; i++) ctx.lineTo(route.pts[i].x, route.pts[i].y);
      ctx.stroke();
    });

    ctx.setLineDash([4, 8]);
    ctx.strokeStyle = colours.teal;
    ctx.globalAlpha = 0.6;
    DASH.forEach(function(d){
      ctx.beginPath();
      ctx.moveTo(d[0][0]*W, d[0][1]*H);
      ctx.lineTo(d[1][0]*W, d[1][1]*H);
      ctx.stroke();
    });
    ctx.setLineDash([]);

    /* nodes */
    NODES.forEach(function(n){
      var x = n.p[0]*W, y = n.p[1]*H;
      ctx.globalAlpha = 0.75;
      ctx.strokeStyle = n.k === 'send' ? colours.amber : colours.faint;
      ctx.fillStyle = colours.ground;
      ctx.lineWidth = 1.2;
      ctx.beginPath();
      if (n.k === 'diamond'){
        ctx.moveTo(x, y-9); ctx.lineTo(x+9, y); ctx.lineTo(x, y+9); ctx.lineTo(x-9, y); ctx.closePath();
      } else {
        ctx.arc(x, y, n.k === 'send' ? 8 : 5.5, 0, Math.PI*2);
      }
      ctx.fill(); ctx.stroke();
    });

    /* send pulses */
    pulses.forEach(function(pu){
      var t = pu.t / 900;
      ctx.globalAlpha = (1 - t) * 0.4;
      ctx.strokeStyle = colours.amber;
      ctx.lineWidth = 1.2;
      ctx.beginPath();
      ctx.arc(pu.x, pu.y, 8 + t * 26, 0, Math.PI*2);
      ctx.stroke();
    });

    /* contacts */
    ctx.fillStyle = colours.amber;
    particles.forEach(function(p){
      var pt = at(routes[p.r], p.d);
      var f = p.d / routes[p.r].total;
      var fade = Math.min(1, f * 12) * Math.min(1, (1 - f) * 9);
      ctx.globalAlpha = p.a * fade * 0.95;
      ctx.beginPath();
      ctx.arc(pt.x, pt.y, p.wait > 0 ? 3.6 : 2.8, 0, Math.PI*2);
      ctx.fill();
    });

    ctx.globalAlpha = 1;
  }

  function frame(now){
    if (!running) return;
    var dt = Math.min(now - last, 60);
    last = now;
    step(dt);
    draw();
    raf = requestAnimationFrame(frame);
  }

  function start(){
    if (running || mq.matches) return;
    running = true; last = performance.now();
    raf = requestAnimationFrame(frame);
  }
  function stop(){ running = false; cancelAnimationFrame(raf); }

  function still(){
    /* one calm frame for reduced-motion visitors */
    particles = [];
    for (var i = 0; i < 16; i++){
      var r = i % 2;
      particles.push({r:r, d: routes[r].total * (0.06 + (i/16) * 0.86), wait:0, a:0.8, gate:0, sent:true});
    }
    draw();
  }

  function boot(){
    readColours(); resize();
    if (!routes.length) return;
    if (mq.matches) still(); else start();
  }

  var io = window.IntersectionObserver ? new IntersectionObserver(function(es){
    visible = es[0].isIntersecting;
    if (visible && !mq.matches) start(); else stop();
  }, {threshold:0}) : null;
  if (io) io.observe(canvas);

  document.addEventListener('visibilitychange', function(){
    if (document.hidden) stop(); else if (visible && !mq.matches) start();
  });

  var rt;
  window.addEventListener('resize', function(){
    clearTimeout(rt);
    rt = setTimeout(function(){ resize(); if (mq.matches) still(); }, 180);
  });

  if (mq.addEventListener) mq.addEventListener('change', function(){ stop(); boot(); });

  /* repaint in the new palette when the theme is toggled */
  new MutationObserver(function(){
    readColours();
    if (mq.matches) still();
  }).observe(root, {attributes:true, attributeFilter:['data-ft-theme']});

  boot();
})();
