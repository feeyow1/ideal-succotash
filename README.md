<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Image SEO Tags</title>
  <style>
    :root {
      --bg: #0f1218;
      --panel: #171c26;
      --line: #2a3344;
      --text: #e8edf5;
      --muted: #93a0b5;
      --accent: #5b8cff;
      --good: #3ecf8e;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: ui-sans-serif, system-ui, Segoe UI, Roboto, Helvetica, Arial;
      background: var(--bg);
      color: var(--text);
      min-height: 100vh;
    }
    header {
      padding: 20px 24px;
      border-bottom: 1px solid var(--line);
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 12px;
    }
    h1 { margin: 0; font-size: 1.15rem; }
    .wrap { max-width: 1100px; margin: 0 auto; padding: 20px; display: grid; gap: 16px; }
    .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
    @media (max-width: 860px) { .grid { grid-template-columns: 1fr; } }
    .card {
      background: var(--panel);
      border: 1px solid var(--line);
      border-radius: 14px;
      padding: 16px;
    }
    label { display: block; font-size: 12px; color: var(--muted); margin: 10px 0 6px; }
    input, textarea, select {
      width: 100%;
      background: #10151e;
      color: var(--text);
      border: 1px solid var(--line);
      border-radius: 10px;
      padding: 10px 12px;
      font: inherit;
    }
    textarea { min-height: 88px; resize: vertical; }
    button {
      background: var(--accent);
      color: #fff;
      border: 0;
      border-radius: 10px;
      padding: 10px 14px;
      font-weight: 600;
      cursor: pointer;
    }
    button.secondary { background: #263044; }
    .row { display: flex; gap: 8px; flex-wrap: wrap; margin-top: 12px; }
    .preview {
      width: 100%;
      min-height: 180px;
      border: 1px dashed var(--line);
      border-radius: 12px;
      display: grid;
      place-items: center;
      overflow: hidden;
      background: #10151e;
      color: var(--muted);
    }
    .preview img { max-width: 100%; max-height: 280px; display: block; }
    .out {
      background: #10151e;
      border: 1px solid var(--line);
      border-radius: 10px;
      padding: 10px 12px;
      font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
      font-size: 13px;
      white-space: pre-wrap;
      word-break: break-word;
    }
    .ok { color: var(--good); font-size: 12px; min-height: 16px; }
    small { color: var(--muted); }
  </style>
</head>
<body>
  <header>
    <h1>Image SEO Tags</h1>
    <small>Local-only · works offline 24/7</small>
  </header>
  <div class="wrap">
    <div class="grid">
      <section class="card">
        <div class="preview" id="preview">Drop or choose an image</div>
        <label>Image file</label>
        <input type="file" id="file" accept="image/*" />
        <label>Page / product topic</label>
        <input id="topic" placeholder="e.g. handmade oak dining table" />
        <label>Primary keyword</label>
        <input id="keyword" placeholder="e.g. oak dining table" />
        <label>Brand / site name (optional)</label>
        <input id="brand" placeholder="e.g. Northwood Furniture" />
        <label>Location (optional)</label>
        <input id="place" placeholder="e.g. Portland, Oregon" />
        <label>What is in the image?</label>
        <textarea id="desc" placeholder="e.g. close-up of a rustic oak table in a sunlit kitchen"></textarea>
        <label>Style</label>
        <select id="style">
          <option value="ecommerce">Ecommerce / product</option>
          <option value="blog">Blog / editorial</option>
          <option value="local">Local business</option>
        </select>
        <div class="row">
          <button id="generate">Generate SEO tags</button>
          <button class="secondary" id="clear">Clear</button>
        </div>
      </section>

      <section class="card">
        <label>Suggested filename</label>
        <div class="out" id="filename"></div>
        <div class="row"><button class="secondary" data-copy="filename">Copy</button></div>

        <label>Alt text</label>
        <div class="out" id="alt"></div>
        <div class="row"><button class="secondary" data-copy="alt">Copy</button></div>

        <label>Title attribute</label>
        <div class="out" id="titleAttr"></div>
        <div class="row"><button class="secondary" data-copy="titleAttr">Copy</button></div>

        <label>Caption</label>
        <div class="out" id="caption"></div>
        <div class="row"><button class="secondary" data-copy="caption">Copy</button></div>

        <label>HTML snippet</label>
        <div class="out" id="html"></div>
        <div class="row"><button class="secondary" data-copy="html">Copy</button></div>

        <label>Open Graph / social meta</label>
        <div class="out" id="og"></div>
        <div class="row"><button class="secondary" data-copy="og">Copy</button></div>

        <p class="ok" id="status"></p>
      </section>
    </div>
  </div>

  <script>
    const $ = id => document.getElementById(id);
    let objectUrl = "";
    let currentName = "image";

    function slug(s) {
      return (s || "")
        .toLowerCase()
        .normalize("NFKD")
        .replace(/[\u0300-\u036f]/g, "")
        .replace(/[^a-z0-9]+/g, "-")
        .replace(/^-+|-+$/g, "")
        .slice(0, 70) || "image";
    }

    function clip(s, n) {
      s = (s || "").replace(/\s+/g, " ").trim();
      if (s.length <= n) return s;
      return s.slice(0, n - 1).replace(/\s+\S*$/, "") + "…";
    }

    $("file").addEventListener("change", e => {
      const file = e.target.files[0];
      if (!file) return;
      currentName = file.name.replace(/\.[^.]+$/, "");
      if (objectUrl) URL.revokeObjectURL(objectUrl);
      objectUrl = URL.createObjectURL(file);
      $("preview").innerHTML = `<img alt="preview" src="${objectUrl}">`;
      if (!$("desc").value) $("desc").value = currentName.replace(/[-_]+/g, " ");
    });

    $("generate").addEventListener("click", () => {
      const topic = $("topic").value.trim();
      const keyword = $("keyword").value.trim() || topic || currentName.replace(/[-_]+/g, " ");
      const brand = $("brand").value.trim();
      const place = $("place").value.trim();
      const desc = $("desc").value.trim() || keyword;
      const style = $("style").value;

      const bits = [keyword];
      if (style === "local" && place) bits.push(place);
      if (brand) bits.push(brand);
      const filename = slug(bits.join(" ")) + ".jpg";

      let alt = desc;
      if (!alt.toLowerCase().includes(keyword.toLowerCase())) alt = `${keyword}: ${desc}`;
      if (style === "local" && place && !alt.toLowerCase().includes(place.toLowerCase())) alt += ` in ${place}`;
      alt = clip(alt, 125);

      const titleAttr = clip([topic || keyword, brand].filter(Boolean).join(" | "), 60);
      const caption = clip(
        style === "ecommerce"
          ? `${keyword}${brand ? " by " + brand : ""}${place ? " — " + place : ""}`
          : desc,
        160
      );

      const html = `<img src="/images/${filename}" alt="${alt.replace(/"/g, "&quot;")}" title="${titleAttr.replace(/"/g, "&quot;")}" width="1200" height="800" loading="lazy">`;

      const ogTitle = clip(topic || keyword, 60);
      const ogDesc = clip(desc, 155);
      const og = `<meta property="og:image" content="https://example.com/images/${filename}">
<meta property="og:image:alt" content="${alt.replace(/"/g, "&quot;")}">
<meta property="og:title" content="${ogTitle.replace(/"/g, "&quot;")}">
<meta property="og:description" content="${ogDesc.replace(/"/g, "&quot;")}">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:image" content="https://example.com/images/${filename}">
<meta name="twitter:image:alt" content="${alt.replace(/"/g, "&quot;")}">`;

      $("filename").textContent = filename;
      $("alt").textContent = alt;
      $("titleAttr").textContent = titleAttr;
      $("caption").textContent = caption;
      $("html").textContent = html;
      $("og").textContent = og;
      $("status").textContent = "Generated. Replace example.com and dimensions as needed.";
    });

    $("clear").addEventListener("click", () => {
      ["topic","keyword","brand","place","desc","file"].forEach(id => $(id).value = "");
      ["filename","alt","titleAttr","caption","html","og","status"].forEach(id => $(id).textContent = "");
      $("preview").textContent = "Drop or choose an image";
      if (objectUrl) URL.revokeObjectURL(objectUrl);
      objectUrl = "";
    });

    document.querySelectorAll("[data-copy]").forEach(btn => {
      btn.addEventListener("click", async () => {
        const text = $(btn.dataset.copy).textContent;
        if (!text) return;
        await navigator.clipboard.writeText(text);
        $("status").textContent = "Copied.";
      });
    });
  </script>
</body>
</html>
