---
title: Home
description: Digital forensics, CTF write-ups, and field references.
---

<div class="home" id="__skip" tabindex="-1">
  <section class="home-hero" aria-labelledby="home-title">
    <div class="home-intro">
      <p class="home-eyebrow"><span></span> fx5 / digital forensics &amp; incident response</p>
      <h1 id="home-title">Follow the traces.<br><em>Understand the story.</em></h1>
      <p class="home-lead">A working notebook for the evidence left behind. Forensic articles, practical references, and lessons from CTF labs, kept in the open.</p>
      <div class="home-actions">
        <a class="home-button" href="articles/">Explore the articles <span aria-hidden="true">↗</span></a>
        <a class="home-text-link" href="about/">Who’s behind this? <span aria-hidden="true">→</span></a>
      </div>
    </div>
    <div class="home-evidence" aria-label="Investigation workflow: collect artifacts, correlate traces, reconstruct activity">
      <div class="home-evidence-header"><span>THE INVESTIGATION LOOP</span><span aria-hidden="true">[ fx5 ]</span></div>
      <div class="home-trace"><span class="home-step">01</span><div><strong>Collect</strong><code>Security.evtx · History · memory.raw</code></div><span class="home-dot" aria-hidden="true"></span></div>
      <div class="home-trace"><span class="home-step">02</span><div><strong>Correlate</strong><code>timestamps → processes → activity</code></div><span class="home-dot" aria-hidden="true"></span></div>
      <div class="home-trace"><span class="home-step">03</span><div><strong>Reconstruct</strong><code>observations → a testable explanation</code></div><span class="home-dot" aria-hidden="true"></span></div>
      <p class="home-evidence-footer"><span aria-hidden="true">&gt;_</span> Every conclusion needs a trail.</p>
    </div>
  </section>

  <section class="home-library" aria-labelledby="home-library-title">
    <div class="home-section-heading"><div><p class="home-eyebrow">ON THE WORKBENCH</p><h2 id="home-library-title">Start with the evidence.</h2></div><a class="home-text-link" href="articles/">All articles <span aria-hidden="true">↗</span></a></div>
    <div class="home-grid">
      <a class="home-card home-card-featured" href="guides/browser-artifacts/">
        <div class="home-card-meta"><span>01 / USER ACTIVITY</span><span aria-hidden="true">↗</span></div>
        <div class="home-artifact" aria-hidden="true"><span>History</span><span>Sessions</span><span>Cache</span><span class="home-artifact-line"></span></div>
        <div><p class="home-kind">IN-DEPTH</p><h3>Browser artifacts</h3><p>Find profiles, trace browsing activity, and understand what survives when history disappears.</p></div>
        <span class="home-card-bottom">Chromium · Firefox · Safari <span aria-hidden="true">→</span></span>
      </a>
      <a class="home-card" href="cheat-sheet/volatility3/">
        <div class="home-card-meta"><span>02 / MEMORY</span><span aria-hidden="true">↗</span></div>
        <div><p class="home-kind">IN-DEPTH</p><h3>Volatility 3</h3><p>Process trees, network state, and injected code. Plugin syntax to keep close during memory analysis.</p></div>
        <span class="home-card-bottom"><code>windows.pslist</code><span aria-hidden="true">→</span></span>
      </a>
      <a class="home-card" href="cheat-sheet/windows-event-ids/">
        <div class="home-card-meta"><span>03 / EVENT LOGS</span><span aria-hidden="true">↗</span></div>
        <div><p class="home-kind">REFERENCE</p><h3>Windows Event IDs</h3><p>Logons, process creation, services, and cleared logs. The events worth a second look, organized by channel.</p></div>
        <span class="home-card-bottom"><code>4624 · 4688 · 1102</code><span aria-hidden="true">→</span></span>
      </a>
      <a class="home-card" href="guides/sqlite-and-the-wal/">
        <div class="home-card-meta"><span>04 / STORAGE</span><span aria-hidden="true">↗</span></div>
        <div><p class="home-kind">SHORT ARTICLE</p><h3>SQLite and the WAL</h3><p>The database may not tell the whole story. Read the companion log and preserve the evidence it holds.</p></div>
        <span class="home-card-bottom"><code>database + database-wal</code><span aria-hidden="true">→</span></span>
      </a>
    </div>
  </section>

  <section class="home-notebook" aria-labelledby="home-notebook-title">
    <span class="home-notebook-mark" aria-hidden="true">~/</span>
    <div><p class="home-eyebrow">A NOTE FROM THE AUTHOR</p><h2 id="home-notebook-title">Useful enough to keep. Open enough to correct.</h2><p>These pages started as notes to myself: commands worth saving, artifacts worth understanding, and reasoning worth retracing. The notebook grows as the lab work does.</p></div>
    <a class="home-text-link" href="about/">About the notebook <span aria-hidden="true">↗</span></a>
  </section>
</div>
