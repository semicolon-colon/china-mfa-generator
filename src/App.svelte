<script lang="ts">
  import mfaImage from "./assets/China MFA Spokesperson.png";

  let canvas = $state<HTMLCanvasElement>();
  let ctx = $state<CanvasRenderingContext2D | null>(null);
  let img = $state<HTMLImageElement>();

  // Text inputs - using $state rune
  let fullText = $state(`われわれは
日本に告げる
台湾問題で
[火遊びをするな]
火遊びをすれば
必ず身を滅ぼす`);
  let source = $state("中国外交部報道官");

  let currentDate = new Date();
  let year = currentDate.getFullYear();
  let month = currentDate.getMonth() + 1;
  let day = currentDate.getDate();
  let date = $state(`${year}年${month}月${day}日`);

  let fontSize = $state(150);
  let lineHeight = $state(1.4);

  // Initialize canvas and image using $effect
  $effect(() => {
    if (canvas) {
      ctx = canvas.getContext("2d");
      const image = new Image();
      image.src = mfaImage;
      image.onload = async () => {
        img = image;
        // Wait for the font to load before drawing
        await document.fonts.load('150px "Noto Serif JP"');
        drawImage();
      };
    }
  });

  function drawLineWithBrackets(
    ctx: CanvasRenderingContext2D,
    text: string,
    x: number,
    y: number,
  ) {
    // Split the text into parts: normal text and bracketed text
    const parts: { text: string; isBracketed: boolean }[] = [];
    let lastIndex = 0;
    const bracketPattern = /\[([^\]]+)\]/g;
    let match;

    while ((match = bracketPattern.exec(text)) !== null) {
      // Add text before the bracket
      if (match.index > lastIndex) {
        parts.push({
          text: text.substring(lastIndex, match.index),
          isBracketed: false,
        });
      }
      // Add bracketed text (without the brackets)
      parts.push({ text: match[1], isBracketed: true });
      lastIndex = match.index + match[0].length;
    }

    // Add remaining text after last bracket
    if (lastIndex < text.length) {
      parts.push({ text: text.substring(lastIndex), isBracketed: false });
    }

    // Calculate total width to center the text
    ctx.font = `150px "Noto Serif JP", serif`;
    ctx.shadowColor = "black";
    ctx.shadowBlur = 0;
    ctx.shadowOffsetX = 2;
    ctx.shadowOffsetY = 3;
    let totalWidth = 0;
    parts.forEach((part) => {
      totalWidth += ctx.measureText(part.text).width;
    });

    // Start from the left position to center the entire line
    let currentX = x - totalWidth / 2;

    // Draw each part
    parts.forEach((part) => {
      ctx.fillStyle = part.isBracketed ? "#FDB933" : "white";
      ctx.textAlign = "left";
      ctx.fillText(part.text, currentX, y);
      currentX += ctx.measureText(part.text).width;
    });

    // Reset text align
    ctx.textAlign = "center";
  }

  function drawImage() {
    if (!ctx || !img || !canvas) return;

    // Set canvas size to image size
    canvas.width = img.width;
    canvas.height = img.height;

    // Draw the base image
    ctx.drawImage(img, 0, 0);

    // Set text properties
    ctx.textAlign = "center";
    ctx.textBaseline = "middle";

    // Main text
    // Split text into lines
    const lines = `“${fullText}”`.split("\n").filter((line) => line.trim());

    const lineSpacing = fontSize * lineHeight;

    // Calculate total height of all text lines
    const totalTextHeight = lines.length * lineSpacing;

    // Start position to vertically center all text
    let yPosition =
      (canvas.height - totalTextHeight) / 2 + lineSpacing / 2 - 30;

    // Draw each line
    lines.forEach((line) => {
      const trimmedLine = line.trim();
      if (!trimmedLine || !ctx || !canvas) return;

      ctx.font = `150px "Noto Serif JP", serif`;

      // Check if line contains bracketed text
      const bracketPattern = /\[([^\]]+)\]/g;
      if (bracketPattern.test(trimmedLine)) {
        // Line contains brackets - render with color
        drawLineWithBrackets(ctx, trimmedLine, canvas.width / 2, yPosition);
      } else {
        // Normal white text
        ctx.fillStyle = "white";
        ctx.fillText(trimmedLine, canvas.width / 2, yPosition);
      }
      yPosition += lineSpacing;
    });

    // Bottom
    const bottomY = canvas.height - 160;

    ctx.fillStyle = "white";
    ctx.font = `65px "Noto Serif JP", serif`;
    ctx.fillText(`${source}   ${date}`, canvas.width / 2, bottomY);
    yPosition += lineSpacing;

    // URL
    ctx.fillStyle = "white";
    ctx.font = `30px "Noto Serif JP", serif`;
    ctx.globalAlpha = 0.5;
    ctx.fillText(
      "https://china-mfa-generator.vercel.app/",
      canvas.width - 300,
      canvas.height - 50,
    );
    ctx.globalAlpha = 1.0;
  }

  function downloadImage() {
    if (!canvas) return;
    const link = document.createElement("a");
    link.download = "中国外交部.png";
    link.href = canvas.toDataURL();
    link.click();
  }

  // Redraw when any text changes using $effect
  $effect(() => {
    if (canvas && ctx && img) {
      // Track reactive dependencies
      fullText;
      fontSize;
      // Ensure font is loaded before redrawing
      document.fonts.load('150px "Noto Serif JP"').then(() => {
        drawImage();
      });
    }
  });
</script>

<main>
  <h1>一番リアルな中国外交部ジェネレーター</h1>

  <div class="container">
    <div class="controls">
      <h2>設定</h2>

      <div class="input-group">
        <label>
          本文（[]で囲んだ部分が黄色になります）
          <textarea
            bind:value={fullText}
            placeholder="Enter your statement (one line per line)"
            rows="10"
          ></textarea>
        </label>
      </div>

      <div class="input-group">
        <label>
          出典
          <input type="text" bind:value={source} class="highlight-input" />
        </label>
      </div>

      <div class="input-group">
        <label>
          日付
          <input type="text" bind:value={date} />
        </label>
      </div>

      <button class="download-btn" onclick={downloadImage}>
        📥 保存する
      </button>

      <p style="margin-top: 1rem; font-size: 0.9rem; color: #aaa;">
        公序良俗に反さないようにお使いください。<br />
        このジェネレーターを使うことで生じた<br
        />いかなる問題についても責任を負いかねます。
      </p>

      <p style="margin-top: 1rem; font-size: 0.9rem; color: #aaa;">
        ソースコードは
        <a
          href="https://github.com/semicolon-colon/china-mfa-generator"
          target="_blank"
          rel="noopener noreferrer"
        >
          こちら
        </a>
        からご覧いただけます。
      </p>
    </div>

    <div class="preview">
      <h2>プレビュー</h2>
      <canvas bind:this={canvas}></canvas>
    </div>
  </div>
</main>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Noto+Serif+JP:wght@200..900&display=swap");

  :global(body) {
    background: #000;
    color: #fff;
    margin: 0;
    padding: 0;
  }

  main {
    max-width: 1400px;
    margin: 0 auto;
    padding: 2rem;
    font-family: "Noto Serif JP", serif;
  }

  h1 {
    text-align: center;
    color: white;
    margin-bottom: 2rem;
    font-weight: 700;
    font-size: 2.5rem;
    text-shadow: 0 0 20px rgba(200, 16, 46, 0.5);
  }

  .container {
    display: grid;
    grid-template-columns: 420px 1fr;
    gap: 2rem;
    align-items: start;
  }

  .controls {
    background: linear-gradient(145deg, #1a1a1a, #0d0d0d);
    padding: 2rem;
    border-radius: 16px;
    position: sticky;
    top: 2rem;
    border: 1px solid #333;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.5);
  }

  .controls h2 {
    margin-top: 0;
    color: #fff;
    font-size: 1.5rem;
    font-weight: 600;
    margin-bottom: 1.5rem;
  }

  .input-group {
    margin-bottom: 1.5rem;
  }

  .input-group label {
    display: block;
    margin-bottom: 0.7rem;
    font-weight: 500;
    color: #ccc;
    font-size: 0.95rem;
  }

  .input-group input[type="text"],
  .input-group textarea {
    width: 100%;
    padding: 0.8rem;
    border: 2px solid #333;
    border-radius: 8px;
    font-size: 1rem;
    font-family: "Noto Serif JP", serif;
    transition: all 0.3s ease;
    background: #0a0a0a;
    color: #fff;
    box-sizing: border-box;
  }

  .input-group textarea {
    resize: vertical;
    min-height: 200px;
    line-height: 1.6;
  }

  .input-group input[type="text"]:focus,
  .input-group textarea:focus {
    outline: none;
    border-color: #c8102e;
    background: #111;
    box-shadow: 0 0 0 3px rgba(200, 16, 46, 0.1);
  }

  .download-btn {
    width: 100%;
    padding: 1.2rem;
    background: linear-gradient(135deg, #c8102e, #a00d24);
    color: white;
    border: none;
    border-radius: 12px;
    font-size: 1.1rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    margin-top: 1rem;
    font-family: "Noto Serif JP", serif;
    box-shadow: 0 4px 15px rgba(200, 16, 46, 0.3);
  }

  .download-btn:hover {
    background: linear-gradient(135deg, #ff1744, #c8102e);
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(200, 16, 46, 0.5);
  }

  .download-btn:active {
    transform: translateY(0);
  }

  .preview {
    background: linear-gradient(145deg, #1a1a1a, #0d0d0d);
    padding: 2rem;
    border-radius: 16px;
    border: 1px solid #333;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.5);
  }

  .preview h2 {
    margin-top: 0;
    color: #fff;
    font-size: 1.5rem;
    font-weight: 600;
    margin-bottom: 1.5rem;
  }

  canvas {
    max-width: 100%;
    height: auto;
    display: block;
    border-radius: 12px;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.6);
  }

  @media (max-width: 1024px) {
    .container {
      grid-template-columns: 1fr;
    }

    .controls {
      position: static;
    }
  }
</style>
