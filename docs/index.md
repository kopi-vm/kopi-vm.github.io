---
template: home.html
---

<div class="hero">
  <img src="assets/logo_black.svg" alt="Kopi Logo" class="hero-logo">
  <div class="hero-content">
    <h1>Kopi</h1>
    <p class="tagline">Lightning-fast JDK version manager</p>
    <p class="sub-tagline">Automatic version switching · Supports all major JDK distributions · Works seamlessly across bash, zsh, and fish shells</p>
    <a href="getting-started/installation.html" class="cta-button">Get Started →</a>
  </div>
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

```text
winget install kopi

kopi local 21

java --version
```

      </div>
      <div class="tab-pane" id="macos">

```text
brew install kopi-vm/tap/kopi

kopi local 21

java --version
```

      </div>
      <div class="tab-pane" id="debian">

```text
curl -fsSL https://kopi-vm.github.io/install.sh | bash

kopi local 21

java --version
```

      </div>
      <div class="tab-pane" id="other">

```text
cargo install kopi

kopi local 21

java --version
```

      </div>
    </div>

  </div>
</div>
