---
template: home.html
---

<div class="hero">
  <img src="assets/logo_black.svg" alt="Kopi Logo" class="hero-logo">
  <div class="hero-content">
    <h1>Kopi</h1>
    <p class="tagline">⚡Lightning-fast JDK version manager</p>
    <a href="getting-started/installation.html" class="cta-button">Get Started →</a>
  </div>
</div>

<div class="features">
  <div class="feature">
    <h2>🚀 Automatic Version Switching</h2>
    <p>Seamlessly switches JDK versions based on your project configuration</p>
  </div>
  <div class="feature">
    <h2>☕ All Major JDK Distributions</h2>
    <p>Supports Oracle, OpenJDK, Amazon Corretto, Eclipse Temurin, and more</p>
  </div>
  <div class="feature">
    <h2>🐚 Universal Shell Support</h2>
    <p>Works seamlessly across all major shells including bash, zsh, fish, cmd, and PowerShell</p>
  </div>
</div>

<div class="try-it-out">
  <h2>Try it out!</h2>
</div>

<div class="code-example">
  <div class="tab-container">
    <div class="tab-buttons">
      <button class="tab-button" data-tab="windows">Windows</button>
      <button class="tab-button" data-tab="macos">macOS</button>
      <button class="tab-button" data-tab="debian">Debian/Ubuntu</button>
      <button class="tab-button" data-tab="other">Other</button>
    </div>
    <div class="tab-content">
      <div class="tab-pane" id="windows">
<pre><code>winget install kopi

kopi local 21

java --version</code></pre>

</div>
<div class="tab-pane" id="macos">

<pre><code>brew install kopi-vm/tap/kopi

kopi local 21

java --version</code></pre>

      </div>
      <div class="tab-pane" id="debian">

<pre><code>curl -fsSL https://kopi-vm.github.io/install.sh | bash

kopi local 21

java --version</code></pre>

      </div>
      <div class="tab-pane" id="other">

<pre><code>cargo install kopi

kopi local 21

java --version</code></pre>

      </div>
    </div>

  </div>
</div>
