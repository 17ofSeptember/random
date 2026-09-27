<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Social Media Profile ID Generator</title>
  <style>
    * { box-sizing: border-box; }

    body {
      margin: 0;
      min-height: 100vh;
      font-family: Arial, Helvetica, sans-serif;
      background: #f0f2f5;
      color: #1c1e21;
      padding: 32px 16px;
    }

    .wrap {
      width: min(1100px, 100%);
      margin: 0 auto;
    }

    .header {
      background: #fff;
      border-radius: 14px;
      padding: 24px;
      margin-bottom: 18px;
      box-shadow: 0 4px 18px rgba(0,0,0,.08);
    }

    h1 {
      margin: 0 0 8px;
      font-size: 28px;
    }

    .subtitle {
      margin: 0;
      color: #65676b;
      line-height: 1.5;
    }

    .toolbar {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      margin-top: 18px;
    }

    button {
      border: 0;
      border-radius: 8px;
      padding: 11px 16px;
      font-size: 14px;
      font-weight: 700;
      cursor: pointer;
      background: #1877f2;
      color: #fff;
    }

    button:hover { filter: brightness(.94); }

    .secondary {
      background: #e4e6eb;
      color: #1c1e21;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(310px, 1fr));
      gap: 16px;
    }

    .card {
      background: #fff;
      border-radius: 14px;
      padding: 20px;
      box-shadow: 0 4px 18px rgba(0,0,0,.07);
    }

    .platform {
      display: flex;
      justify-content: space-between;
      gap: 12px;
      align-items: center;
      margin-bottom: 8px;
    }

    .platform h2 {
      margin: 0;
      font-size: 20px;
    }

    .badge {
      font-size: 11px;
      font-weight: 700;
      padding: 5px 8px;
      border-radius: 999px;
      background: #e7f3ff;
      color: #1877f2;
      white-space: nowrap;
    }

    .badge.api {
      background: #f0f0f0;
      color: #555;
    }

    .description {
      color: #65676b;
      font-size: 13px;
      line-height: 1.45;
      min-height: 38px;
      margin-bottom: 13px;
    }

    .value {
      background: #f7f8fa;
      border: 1px solid #dddfe2;
      border-radius: 8px;
      padding: 12px;
      min-height: 46px;
      font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
      font-size: 13px;
      word-break: break-all;
      display: flex;
      align-items: center;
    }

    .linkbox {
      margin-top: 9px;
      min-height: 38px;
      font-size: 13px;
      word-break: break-all;
    }

    .linkbox a {
      color: #1877f2;
      text-decoration: underline;
    }

    .unavailable {
      color: #777;
      font-style: italic;
    }

    .card-actions {
      display: flex;
      gap: 8px;
      margin-top: 12px;
    }

    .card-actions button {
      flex: 1;
    }

    .note {
      margin-top: 18px;
      background: #fff;
      border-radius: 14px;
      padding: 18px 20px;
      color: #65676b;
      font-size: 13px;
      line-height: 1.5;
      box-shadow: 0 4px 18px rgba(0,0,0,.06);
    }
  </style>
</head>

<body>
  <main class="wrap">
    <section class="header">
      <h1>Social Media Profile ID Generator</h1>
      <p class="subtitle">
        Generates structurally plausible account identifiers for Facebook, VK, Steam,
        YouTube, Flickr, X, Instagram, Reddit, TikTok, and Discord.
      </p>

      <div class="toolbar">
        <button id="generateAll">Generate All</button>
        <button id="copyAll" class="secondary">Copy All</button>
      </div>
    </section>

    <section class="grid" id="cards"></section>

    <section class="note">
      These generators reproduce the documented or commonly used identifier structure.
      They do not query the platforms and cannot guarantee that a generated ID belongs
      to a real account. Platforms with a direct ID-based profile route show a clickable
      link. Platforms that normally require a username or authorized API lookup show the
      generated identifier only.
    </section>
  </main>

  <script>
    const cryptoInt = (maxExclusive) => {
      if (!Number.isSafeInteger(maxExclusive) || maxExclusive <= 0) {
        throw new Error("maxExclusive must be a positive safe integer");
      }

      const range = 0x100000000;
      const limit = range - (range % maxExclusive);
      const arr = new Uint32Array(1);

      do {
        crypto.getRandomValues(arr);
      } while (arr[0] >= limit);

      return arr[0] % maxExclusive;
    };

    const randomDigits = (length) => {
      let out = "";
      for (let i = 0; i < length; i++) out += cryptoInt(10);
      return out;
    };

    const randomChars = (chars, length) => {
      let out = "";
      for (let i = 0; i < length; i++) out += chars[cryptoInt(chars.length)];
      return out;
    };

    const randomBigInt = (bits) => {
      const bytes = Math.ceil(bits / 8);
      const arr = new Uint8Array(bytes);
      crypto.getRandomValues(arr);

      let value = 0n;
      for (const b of arr) value = (value << 8n) | BigInt(b);

      const excess = BigInt(bytes * 8 - bits);
      if (excess > 0n) value >>= excess;
      return value;
    };

    const randomTimestamp = (startMs, endMs = Date.now()) => {
      const span = endMs - startMs;
      return startMs + Math.floor(Math.random() * span);
    };

    const uuidV4 = () => {
      if (crypto.randomUUID) return crypto.randomUUID();

      const b = new Uint8Array(16);
      crypto.getRandomValues(b);
      b[6] = (b[6] & 0x0f) | 0x40;
      b[8] = (b[8] & 0x3f) | 0x80;

      const h = [...b].map(x => x.toString(16).padStart(2, "0"));
      return `${h.slice(0,4).join("")}-${h.slice(4,6).join("")}-${h.slice(6,8).join("")}-${h.slice(8,10).join("")}-${h.slice(10).join("")}`;
    };

    // Snowflake generator. The lower 22 bits are randomized.
    const makeSnowflake = (epochMs, startMs) => {
      const t = BigInt(randomTimestamp(startMs));
      const timestampPart = (t - BigInt(epochMs)) << 22n;
      const lower22 = randomBigInt(22);
      return (timestampPart | lower22).toString();
    };

    const platforms = [
      {
        key: "facebook",
        name: "Facebook",
        type: "Direct profile URL",
        description: "Modern-style 15-digit numeric Facebook profile ID.",
        generate: () => "1000" + randomDigits(11),
        link: id => `https://www.facebook.com/profile.php?id=${id}`
      },
      {
        key: "vk",
        name: "VK",
        type: "Direct profile URL",
        description: "Numeric VK user ID used in the /idNUMBER profile route.",
        generate: () => String(1 + cryptoInt(999999999)),
        link: id => `https://vk.com/id${id}`
      },
      {
        key: "steam",
        name: "Steam",
        type: "Direct profile URL",
        description: "SteamID64 generated from Valve's individual-account base plus a 32-bit account number.",
        generate: () => {
          const base = 76561197960265728n;
          const accountId = randomBigInt(32);
          return (base + accountId).toString();
        },
        link: id => `https://steamcommunity.com/profiles/${id}/`
      },
      {
        key: "youtube",
        name: "YouTube",
        type: "Direct channel URL",
        description: "24-character UC-style YouTube channel ID.",
        generate: () => "UC" + randomChars("ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789_-", 22),
        link: id => `https://www.youtube.com/channel/${id}`
      },
      {
        key: "flickr",
        name: "Flickr",
        type: "Direct profile URL",
        description: "Flickr NSID-style identifier used in /people/USER-ID/ URLs.",
        generate: () => {
          const numeric = String(10000000 + cryptoInt(90000000));
          const suffix = String(cryptoInt(100)).padStart(2, "0");
          return `${numeric}@N${suffix}`;
        },
        link: id => `https://www.flickr.com/people/${id}/`
      },
      {
        key: "x",
        name: "X",
        type: "Direct ID redirect",
        description: "Snowflake-style numeric X user ID. The /i/user/ route can redirect to the current username.",
        generate: () => makeSnowflake(
          1288834974657,
          Date.UTC(2016, 0, 1)
        ),
        link: id => `https://x.com/i/user/${id}`
      },
      {
        key: "instagram",
        name: "Instagram",
        type: "API/internal ID",
        description: "Modern Graph API-style numeric IG User ID. Public profile URLs normally use usernames.",
        generate: () => "1784" + randomDigits(13),
        link: null
      },
      {
        key: "reddit",
        name: "Reddit",
        type: "API/internal ID",
        description: "Reddit user Thing ID using the t2_ prefix and a base-36 identifier.",
        generate: () => "t2_" + randomChars("0123456789abcdefghijklmnopqrstuvwxyz", 7 + cryptoInt(5)),
        link: null
      },
      {
        key: "tiktok",
        name: "TikTok",
        type: "API/internal ID",
        description: "TikTok OpenID-style UUID. TikTok profile links are normally returned separately as profile_deep_link.",
        generate: () => uuidV4(),
        link: null
      },
      {
        key: "discord",
        name: "Discord",
        type: "Internal Snowflake",
        description: "Discord Snowflake-style user ID. Discord does not provide a conventional public profile URL for arbitrary IDs.",
        generate: () => makeSnowflake(
          1420070400000,
          Date.UTC(2016, 0, 1)
        ),
        link: null
      }
    ];

    const state = {};

    function renderCards() {
      const cards = document.getElementById("cards");

      cards.innerHTML = platforms.map(p => `
        <article class="card">
          <div class="platform">
            <h2>${p.name}</h2>
            <span class="badge ${p.link ? "" : "api"}">${p.type}</span>
          </div>

          <div class="description">${p.description}</div>

          <div class="value" id="${p.key}-value">Not generated</div>
          <div class="linkbox" id="${p.key}-link"></div>

          <div class="card-actions">
            <button onclick="generateOne('${p.key}')">Generate</button>
            <button class="secondary" onclick="copyOne('${p.key}')">Copy</button>
          </div>
        </article>
      `).join("");
    }

    function generateOne(key) {
      const p = platforms.find(x => x.key === key);
      const id = p.generate();
      const url = p.link ? p.link(id) : null;

      state[key] = { id, url };

      document.getElementById(`${key}-value`).textContent = id;

      const linkBox = document.getElementById(`${key}-link`);
      linkBox.replaceChildren();

      if (url) {
        const a = document.createElement("a");
        a.href = url;
        a.target = "_blank";
        a.rel = "noopener noreferrer";
        a.textContent = url;
        linkBox.appendChild(a);
      } else {
        const span = document.createElement("span");
        span.className = "unavailable";
        span.textContent = "No direct public ID-based profile URL";
        linkBox.appendChild(span);
      }
    }

    function generateAll() {
      for (const p of platforms) generateOne(p.key);
    }

    async function copyOne(key) {
      if (!state[key]) generateOne(key);
      const item = state[key];
      await navigator.clipboard.writeText(item.url || item.id);
    }

    async function copyAll() {
      for (const p of platforms) {
        if (!state[p.key]) generateOne(p.key);
      }

      const text = platforms.map(p => {
        const item = state[p.key];
        return `${p.name}: ${item.id}${item.url ? `\n${item.url}` : ""}`;
      }).join("\n\n");

      await navigator.clipboard.writeText(text);
    }

    document.getElementById("generateAll").addEventListener("click", generateAll);
    document.getElementById("copyAll").addEventListener("click", copyAll);

    renderCards();
  </script>
</body>
</html>
