<script>
  import TrustBar from "$lib/TrustBar.svelte";
  import ToolHeader from "$lib/ToolHeader.svelte";
  import ToolFooter from "$lib/ToolFooter.svelte";

  let input = $state("");
  let output = $state("");
  let valid = $state(null);
  let copied = $state(false);
  let mode = $state("decode");

  function process() {
    if (input.trim() === "") {
      valid = null;
      output = "";
      return;
    }

    try {
      if (mode === "encode") {
        output = btoa(unescape(encodeURIComponent(input)));
      } else {
        output = decodeURIComponent(escape(atob(input.trim())));
      }
      valid = true;
    } catch (e) {
      output = "";
      valid = false;
    }
  }

  function handleInput() {
    process();
  }

  function toggleMode() {
    mode = mode === "encode" ? "decode" : "encode";
    if (output && valid) {
      input = output;
      process();
    } else {
      process();
    }
  }

  function copyOutput() {
    if (!output) return;
    navigator.clipboard
      .writeText(output)
      .then(() => {
        copied = true;
        setTimeout(() => (copied = false), 1500);
      })
      .catch(() => {});
  }

  function clear() {
    input = "";
    output = "";
    valid = null;
    copied = false;
  }

  function loadSample() {
    if (mode === "decode") {
      input =
        "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ";
      process();
    } else {
      input = '{"sub":"1234567890","name":"John Doe","iat":1516239022}';
      process();
    }
  }

  let inputLabel = $derived(mode === "encode" ? "text" : "base64");
  let outputLabel = $derived(mode === "encode" ? "base64" : "text");
  let inputPlaceholder = $derived(
    mode === "encode" ? "Paste text to encode..." : "Paste Base64 to decode...",
  );
  let outputPlaceholder = $derived(
    valid === false
      ? mode === "decode"
        ? "Invalid Base64 input..."
        : "Encoding error..."
      : mode === "encode"
        ? "Encoded Base64 appears here..."
        : "Decoded text appears here...",
  );
  let byteCount = $derived(input.trim() ? new Blob([input]).size : 0);
  let outputByteCount = $derived(output ? new Blob([output]).size : 0);
</script>

<TrustBar sourceUrl="https://github.com/Griffin2/localbase64" />

<div class="page-shell">
  <ToolHeader
    title="Base64 Codec"
    description="Encode and decode Base64 — nothing leaves your browser."
  />

  <div class="tool-area">
    <div class="panel">
      <label class="tool-label" for="input">{inputLabel}</label>
      <textarea
        id="input"
        class="tool-textarea"
        bind:value={input}
        oninput={handleInput}
        placeholder={inputPlaceholder}
        spellcheck="false"
      ></textarea>
    </div>

    <div class="panel">
      <label class="tool-label" for="output">{outputLabel}</label>
      <textarea
        id="output"
        class="tool-textarea"
        class:output-valid={valid === true}
        value={output}
        readonly
        placeholder={outputPlaceholder}
        spellcheck="false"
      ></textarea>
    </div>
  </div>

  <div class="controls">
    <button class="btn btn-primary" onclick={process}>
      {mode === "encode" ? "Encode" : "Decode"}
    </button>
    <button class="btn btn-swap" onclick={toggleMode} title="Swap direction">
      ⇄ {mode === "encode" ? "Decode" : "Encode"}
    </button>
    <button class="btn" onclick={copyOutput}>
      {copied ? "✓ Copied" : "Copy"}
    </button>
    <button class="btn" onclick={clear}>Clear</button>
    <button class="btn btn-sample" onclick={loadSample}>Sample</button>

    {#if byteCount > 0}
      <div class="byte-info">
        <span class="byte-tag">{byteCount.toLocaleString()} B in</span>
        {#if outputByteCount > 0}
          <span class="byte-sep">→</span>
          <span class="byte-tag">{outputByteCount.toLocaleString()} B out</span>
        {/if}
      </div>
    {/if}

    <div class="spacer"></div>

    {#if valid === true}
      <span class="pill pill-ok">
        <span class="pill-dot"></span>
        {mode === "encode" ? "encoded" : "decoded"}
      </span>
    {:else if valid === false}
      <span class="pill pill-err">
        <span class="pill-dot"></span>
        invalid
      </span>
    {/if}
  </div>

  <ToolFooter />
</div>

<style>
  .panel {
    display: flex;
    flex-direction: column;
    gap: 5px;
  }

  .btn-sample {
    font-size: 12px;
    padding: 8px 12px;
    color: var(--text-muted);
  }

  .btn-swap {
    font-size: 12px;
    padding: 8px 12px;
    color: var(--accent);
    border-color: var(--accent-border);
  }

  .btn-swap:hover {
    background: var(--accent-bg);
  }

  .byte-info {
    display: flex;
    align-items: center;
    gap: 5px;
    margin-left: 8px;
  }

  .byte-tag {
    font-family: var(--font-mono);
    font-size: 11px;
    color: var(--text-muted);
    background: var(--bg-input);
    padding: 3px 8px;
    border-radius: var(--radius-sm);
    border: 1px solid var(--border-light);
  }

  .byte-sep {
    font-size: 11px;
    color: var(--text-muted);
  }

  .spacer {
    margin-left: auto;
  }

  @media (max-width: 640px) {
    .byte-info {
      display: none;
    }

    .controls {
      flex-wrap: wrap;
    }
  }
</style>
