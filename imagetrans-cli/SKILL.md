---
name: imagetrans-cli
description: >-
  Guide for operating ImageTrans via command line to OCR, translate, and export
  images and PDFs. Use this skill whenever the user wants to batch process images/PDFs
  through OCR and translation, run automated ImageTrans workflows,
  export translated images/PDFs/markdown, or perform any ImageTrans
  operation from the command line. Triggers on mentions of ImageTrans, image translation
  automation, manga/comics/document translation CLI, batch OCR translation, or
  generating translated PDFs/markdown from images.
---

# ImageTrans CLI Operations

## Overview

ImageTrans is a JavaFX desktop application for OCR, translation, and inpainting of
images and PDFs. It can be operated entirely via command line, making it suitable
for automation and batch processing.

### Core capabilities
- **OCR**: Extract text from images using PaddleOCR (rapid), cloud OCR engines, or specialized manga OCR
- **Translation**: Machine translate extracted text via multiple engines (ChatGPT, DeepSeek, Baidu, Google, DeepL, etc.)
- **Text removal & inpainting**: Remove original text and generate clean backgrounds
- **Export**: Output translated images, searchable PDFs, Markdown, PSD, XLIFF, TXT, etc

### Architecture

ImageTrans works with **projects** (`.itp` files) that reference images and store
text box data (position, source text, translation, style). A **workflow** is a
sequence of operations defined in `custom_workflow.json`.

## Prerequisites

- **Java Runtime**: JRE is bundled inside the ImageTrans installation directory
- **ImageTrans.jar**: The main application JAR in the ImageTrans installation directory
- **Working directory**: All commands must run from the ImageTrans installation directory (where `ImageTrans.jar` lives)

### Determining the ImageTrans Installation Path

Before running any command, you MUST know the ImageTrans installation directory.
The path is **persisted across sessions** — the user only needs to provide it once.

Follow this procedure:

1. **Check persisted memory first.** Look for a memory file named `imagetrans-path` in the project memory directory. If found and the path is still valid (`<path>/ImageTrans.jar` exists), use it directly — no need to ask the user.

2. **If no persisted path, check platform default:**

   | Platform | Default installation path |
   |----------|--------------------------|
   | macOS    | `/Applications/ImageTrans.app/Contents/Resources/ImageTrans` |
   | Windows  | (none) |
   | Linux    | (none) |

   - **macOS**: Try the default path first. If `ImageTrans.jar` exists there, use it and persist it. Only ask the user if the default path doesn't exist.
   - **Windows / Linux**: Ask the user once:
     > "请提供 ImageTrans 的安装目录（完整路径）："

3. **Verify the path** — check that `<path>/ImageTrans.jar` exists. If it doesn't, tell the user and ask them to provide the correct path.

4. **Persist the path** — once verified, write a memory file so future sessions can skip this step. Write to `<project-memory-dir>/imagetrans-path.md`:
   ```markdown
   ---
   name: imagetrans-path
   description: ImageTrans installation directory path
   metadata:
     type: project
   ---
   
   The user's ImageTrans installation directory is: `<VERIFIED_PATH>`
   
   **Why:** Avoid asking the user for the path on every interaction.
   **How to apply:** Before running any ImageTrans command, read this memory. If the path no longer exists (e.g., after an update), re-run the discovery flow.
   ```

5. **Derive JAVA_PATH and JAVAFX_LIB** from `<IMAGETRANS_DIR>`:

   | Platform | `JAVA_PATH` | `JAVAFX_LIB` |
   |----------|-------------|--------------|
   | Windows  | `<IMAGETRANS_DIR>/jre/bin/java.exe` | `<IMAGETRANS_DIR>/jre/javafx/lib` |
   | Linux    | `<IMAGETRANS_DIR>/jre/bin/java` | `<IMAGETRANS_DIR>/jre/javafx/lib` |
   | macOS    | `<IMAGETRANS_DIR>/jdk-23/Contents/Home/bin/java` | `<IMAGETRANS_DIR>/jdk-23/Contents/Home/javafx/lib` |

6. All subsequent commands use `cd <IMAGETRANS_DIR>` first, then launch with the derived `JAVA_PATH` and `JAVAFX_LIB`.

### JVM Launch Command

All CLI invocations share the same base command.

The `-Djava.library.path=./` flag is **required on Linux and macOS** so Java can find
native libraries (OpenCV, ONNX Runtime, etc.) shipped in the ImageTrans directory.

**Windows does NOT need this flag.** Windows searches the current directory for DLLs by
default, and the `-Djava.library.path=./` syntax breaks in PowerShell (`.` is parsed as
a separate token). **Omit `-Djava.library.path=./` on Windows.**

```bash
# Linux / macOS — include -Djava.library.path=./
<JAVA_PATH> \
  -Djava.library.path=./ \
  --module-path <JAVAFX_LIB> \
  --add-modules javafx.base,javafx.controls,javafx.graphics,javafx.web,javafx.swing \
  --add-opens javafx.controls/com.sun.javafx.scene.control.skin=ALL-UNNAMED \
  --add-exports javafx.base/com.sun.javafx.collections=ALL-UNNAMED \
  --add-exports java.desktop/sun.awt=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.bmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.gif=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.wbmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.spi=ALL-UNNAMED \
  --add-opens java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  -jar ImageTrans.jar <ARGS>
```

**Windows command (omit `-Djava.library.path=./`):**
```bash
<JAVA_PATH> \
  --module-path <JAVAFX_LIB> \
  --add-modules javafx.base,javafx.controls,javafx.graphics,javafx.web,javafx.swing \
  --add-opens javafx.controls/com.sun.javafx.scene.control.skin=ALL-UNNAMED \
  --add-exports javafx.base/com.sun.javafx.collections=ALL-UNNAMED \
  --add-exports java.desktop/sun.awt=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.bmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.gif=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.wbmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.spi=ALL-UNNAMED \
  --add-opens java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  -jar ImageTrans.jar <ARGS>
```

**Platform-specific values** (paths relative to `<IMAGETRANS_DIR>`, see [Determining the ImageTrans Installation Path](#determining-the-imagetrans-installation-path)):

| Platform | `JAVA_PATH` | `JAVAFX_LIB` |
|----------|-------------|--------------|
| Windows  | `jre/bin/java.exe` | `jre/javafx/lib` |
| Linux    | `jre/bin/java` | `jre/javafx/lib` |
| macOS    | `jdk-23/Contents/Home/bin/java` | `jdk-23/Contents/Home/javafx/lib` |

**Important**: When building the command for the user, always use the correct paths
for their OS. Verify the Java path exists before running. If the bundled JRE is not
found, fall back to system `java` and check for JavaFX availability.

### Headless Mode (No GUI Windows)

**Important:** The bundled JavaFX is version 23, which does **not** support headless
mode. `-Dglass.platform=headless` requires **JavaFX 26 or later**. Do not use this
flag with the bundled JRE — it will cause errors.

If you are using a custom Java runtime with JavaFX 26+, you can add the flag to
suppress all JavaFX windows:

```
-Dglass.platform=headless
```

Place it alongside the other `-D` flags:

```bash
# Linux / macOS
<JAVA_PATH> \
  -Djava.library.path=./ \
  -Dglass.platform=headless \
  --module-path <JAVAFX_LIB> \
  ... \
  -jar ImageTrans.jar <ARGS>
```

**(Omit `-Djava.library.path=./` on Windows — see [JVM Launch Command](#jvm-launch-command).)**

**When headless mode works (JavaFX 26+ only):**
- Batch processing — no need to see the GUI while images are being processed
- CI/CD pipelines or automated scripts
- Linux servers without X11/Wayland, Docker containers, WSL without an X server
- Windows/macOS when you want the process to run silently in the background

**When NOT to use headless mode:**
- With the bundled JavaFX 23 — not supported, will error
- When you need to visually inspect or manually adjust results
- When using the full GUI for interactive work

## CLI Modes

ImageTrans has several CLI invocation modes, selected by the number and content of arguments.

---

### Mode 1: Template Mode (Recommended for Batch Processing)

```
<JAVA> ... -jar ImageTrans.jar <templateDir> <inputPath>
```

**Arguments:**
- `templateDir`: Path to a template directory containing `.itp` project file, `custom_workflow.json`, and optional `settings.json`
- `inputPath`: An image file (`.jpg`, `.png`, `.bmp`, `.webp`), a `.pdf` file, or a directory of images

**Behavior:**
1. Copies the `.itp` file and supporting files from `templateDir` to the input's parent folder
2. Imports the image(s) or PDF pages (PDF import may require pre-configured options, see [Configuring PDF Import for CLI/Batch Use](#configuring-pdf-import-for-clibatch-use))
3. Applies `settings.json` (source/target language, etc.) and `preferences.conf` if present
4. Runs the default workflow (index 0 from `custom_workflow.json`)
5. Saves the project and exits automatically

**Use this mode when:** You want to process new images/PDFs with zero manual setup.
The template provides all configuration — OCR engine, translation engine, workflow steps.

**Important — Customizing templates:** When you need to customize a template (e.g., add
workflow steps, change settings), do NOT modify the original template files directly.
Instead, copy the template directory to a temporary location, make your changes there,
and use the temporary copy as the `templateDir` argument. This preserves the original
templates for future use. Example:

```bash
# Copy template to a temp directory and customize it
cp -r templates/manga-ja2en /tmp/manga-custom/
# Edit the copy, not the original
# ... edit /tmp/manga-custom/custom_workflow.json ...
# Use the temp copy
... -jar ImageTrans.jar "/tmp/manga-custom" "C:/manga/page01.jpg"
```

**Templates directory:** Templates are stored in `<IMAGETRANS_DIR>/templates/`.
Common built-in templates include `manga` (for manga/comics translation) and `document`
(for document/PDF translation). The `templateDir` argument can be an absolute path or
a path relative to the ImageTrans working directory.

| Template | Path (relative to `<IMAGETRANS_DIR>`) | Use case | Source→Target |
|----------|---------------------------------------|----------|---------------|
| `manga` | `templates/manga` | Manga / comics pages | ja→? |
| `manga-ja2en` | `templates/manga-ja2en` | Japanese to English manga | ja→en |
| `manga-ja2zh` | `templates/manga-ja2zh` | Japanese to Chinese manga | ja→zh |
| `comics` | `templates/comics` | Western comics | en→? |
| `webtoon` | `templates/webtoon` | Korean/Chinese webtoons | ? |
| `document` | `templates/document` | Document / PDF translation | ? |
| `cg` | `templates/cg` | Computer graphics / game screenshots | ? |
| `cg-ja2zh` | `templates/cg-ja2zh` | Japanese to Chinese game screenshots | ja→zh |
| `chinese-manhua` | `templates/chinese-manhua` | Chinese manhua | zh→? |

**Example — Translate a manga page (Windows):**
```bash
cd /path/to/ImageTrans
jre/bin/java.exe \
  --module-path jre/javafx/lib \
  --add-modules javafx.base,javafx.controls,javafx.graphics,javafx.web,javafx.swing \
  --add-opens javafx.controls/com.sun.javafx.scene.control.skin=ALL-UNNAMED \
  --add-exports javafx.base/com.sun.javafx.collections=ALL-UNNAMED \
  --add-exports java.desktop/sun.awt=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.bmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.gif=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.wbmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.spi=ALL-UNNAMED \
  --add-opens java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  -jar ImageTrans.jar "templates/manga" "C:/manga/page01.jpg"
```

**Example — Translate a manga page (Linux):**
```bash
cd /path/to/ImageTrans
jre/bin/java \
  -Djava.library.path=./ \
  --module-path jre/javafx/lib \
  --add-modules javafx.base,javafx.controls,javafx.graphics,javafx.web,javafx.swing \
  --add-opens javafx.controls/com.sun.javafx.scene.control.skin=ALL-UNNAMED \
  --add-exports javafx.base/com.sun.javafx.collections=ALL-UNNAMED \
  --add-exports java.desktop/sun.awt=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.bmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.gif=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.wbmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.spi=ALL-UNNAMED \
  --add-opens java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  -jar ImageTrans.jar "templates/manga" "/home/user/manga/page01.jpg"
```

**Example — Translate a PDF document:**
```bash
cd /path/to/ImageTrans
# Use document template for PDF processing
... -jar ImageTrans.jar "templates/document" "C:/docs/report.pdf"
```

---

### Mode 2: Workflow Mode (Run Workflow on Existing Project)

```
<JAVA> ... -jar ImageTrans.jar <projectPath> <workflowIndex> workflow
```

**Arguments:**
- `projectPath`: Path to an existing `.itp` project file
- `workflowIndex`: Integer index into the `custom_workflow.json` array (0-based)
- `workflow`: The literal string `"workflow"` — **required**

**Behavior:**
1. Opens the existing project
2. Executes the workflow at `workflowIndex` from the project's `custom_workflow.json`
3. Saves the project
4. Logs `customflow done` and exits

**Use this mode when:** You have an existing project and want to run a specific
workflow on it — e.g., re-run translation with different settings, or run an
export-only workflow.

**Example — Run workflow #2 on an existing project:**
```bash
... -jar ImageTrans.jar "C:/manga/project.itp" 2 workflow
```

**Example — Run the first (default) workflow:**
```bash
... -jar ImageTrans.jar "C:/manga/project.itp" 0 workflow
```

---

## Template System

Templates are directories containing:

```
template-name/
├── <name>.itp              # Project file (JSON) — the template project
├── custom_workflow.json    # Workflow definitions (JSON array)
├── settings.json           # Optional: project settings overrides
├── preferences.conf        # Optional: app-level preference overrides
└── setOCRBasedOnLang       # Optional: empty flag file — auto-set OCR engine
```

### Built-in Templates

See the templates table in [Mode 1 (Template Mode)](#mode-1-template-mode-recommended-for-batch-processing) — it lists all built-in
templates with their paths, use cases, and source→target languages.

### Creating a Custom Template

To create a template for a new language pair or use case:

1. **Create the directory structure:**
   ```
   my-template/
   ├── 1.itp
   ├── custom_workflow.json
   └── settings.json
   ```

2. **`settings.json`** — Modify the project settings to set source and target languages, and the api preferences to set key and prompt for ChatGPT translation. Example:
   ```json
   {
      "sourceLang": "ja",
      "targetLang": "en",
      "api": {
        "chatGPT": {"key":"api key","batch_prompt":"prompt text"}
      }
   }
   ```

3. **`custom_workflow.json`** — Define your workflow (see Workflow Configuration below).

4. **`1.itp`** — Copy from `general_template/1.itp` and adjust settings
   (OCR engine, inpainting options, etc.).

5. **`preferences.conf`** (optional) — Override app preferences:
   ```json
   {
     "api": {
       "chatGPT": {"key":"api key","batch_prompt":"prompt text"}
     }
   }
   ```

## Workflow Configuration

Workflows are defined in `custom_workflow.json` as a JSON array. Each entry represents
one named workflow with its sequence of operations.

### Workflow Object Structure

```json
[
  {
    "name": "default",
    "flow": [
      "Text detection",
      "Merge areas",
      "Sort",
      "Translation",
      "Generate translated pictures"
    ],
    "ocrInterval": 100,
    "detectionMethod": 0,
    "mergeOperation": 2,
    "engine": "google",
    "type": "MT",
    "scale": 0.5,
    "auto_remove_low_confidence_areas": true,
    "panel_detection_method": 0,
    "play_sound": false
  }
]
```

### Top-level Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Display name for this workflow |
| `flow` | string[] | Ordered list of operation names |
| `ocrInterval` | int | Delay in ms between OCR API calls (throttling) |
| `detectionMethod` | int | Text detection method (see below) |
| `mergeOperation` | int/string | Merge strategy for text areas |
| `engine` | string | Default MT engine id (e.g., `"google"`, `"baidu"`, `"deepl"`) |
| `type` | string | Translation type — usually `"MT"` |
| `scale` | float | Default box scaling factor |
| `panel_detection_method` | int | Panel/comic frame detection method |
| `auto_remove_low_confidence_areas` | bool | Auto-remove low-confidence OCR results |
| `play_sound` | bool | Play sound on completion |

### Detection Methods (`detectionMethod`)

| Value | Method | Description |
|-------|--------|----------|
| `0` | OCR-based | Use OCR engines' text detection capabilities and recognize text at the same time. Text recognition workflow is no longer needed. |
| `1` | Heuristic | Use connected-components to find text regions (not recommended) |
| `2` | Scene text detection | Use local scene text detection server (not recommended) |
| `3` | Balloon detection | Manga/comics speech bubbles or text lines according to different object detection models |

### Workflow Operations Reference

Operations are referenced by their **English** or **Chinese** names. The code
normalizes by looking up the source-language equivalent, so either works.

#### Text Operations

| English Name | Chinese Name | Description |
|-------------|-------------|-------------|
| `Text detection` | 文字检测 | Detect text areas on all images |
| `Text detection (N)` | 文字检测 (N) | Text detection with specific method N |
| `Text recognition` | 文字识别 | OCR all detected text boxes |
| `Text recognition (engine,langIndex)` | 文字识别 (engine,langIndex) | OCR with specific engine and language |
| `Recognize long text areas` | 识别长条区域 | Detect long horizontal text regions |
| `Recognize textless areas mergable with areas with text` | 识别可以与有文字区域合并的无文字区域 | Find mergeable text-free areas |
| `Append OCR result` | 用OCR给现有文本框添加文字 | Re-OCR existing boxes |
| `Get text area confidence` | 获取文字区域置信度 | Score OCR confidence per area |
| `Remove low confidence areas` | 去除低置信度区域 | Strip low-confidence results |
| `Remove areas without source text for all pictures` | 去除所有图片的无原文区域 | Clean empty boxes |
| `Remove non-text areas (OCR) for all pictures` | 去除所有图片的非文字区域（OCR） | Filter non-text boxes via OCR |
| `Clear source` | 清除原文 | Remove all source text |
| `Clear target` | 清除译文 | Remove all translations |
| `Clear target position` | 清除译文位置 | Reset translation positions |

#### Translation

| English Name | Chinese Name | Description |
|-------------|-------------|-------------|
| `Translation` | 翻译 | Machine translate all texts |

#### Area Manipulation

| English Name | Chinese Name | Description |
|-------------|-------------|-------------|
| `Merge areas` | 合并区域 | Merge overlapping/adjacent text areas |
| `Sort` | 排序 | Sort text areas by reading order |
| `Scale` | 缩放 | Scale text boxes |
| `Expand areas` | 拓展区域 | Expand text area boundaries |
| `Shrink areas` | 缩小区域 | Shrink text area boundaries |
| `Expand areas for target` | 拓展译文区域 | Expand target text areas |
| `Shrink areas for target` | 缩小译文区域 | Shrink target text areas |
| `Remove outer areas` | 去除外部区域 | Remove areas outside image bounds |
| `Remove areas outside the image` | 去除图片外区域 | Remove out-of-bounds areas |
| `Merge text areas within panels` | 合并分镜内的区域 | Merge boxes within same panel |

#### Panel/Frame Detection

| English Name | Chinese Name | Description |
|-------------|-------------|-------------|
| `Panel detection` | 分镜检测 | Detect comic panels/frames |
| `Sort panels` | 分镜排序 | Sort panels by reading order |
| `Remove areas outside panels` | 去除分镜外区域 | Remove text outside panels |
| `Remove areas within image and table panels` | 去除图表类分镜内的区域 | Remove text in image/table panels |

#### Color & Style

| English Name | Chinese Name | Description |
|-------------|-------------|-------------|
| `Detect font color` | 识别文字颜色 | Detect text foreground color |
| `Detect background color` | 识别背景颜色 | Detect text background color |
| `Use the detected text color as the stroke color` | 使用识别的文字颜色做为描边颜色 | Apply text color to outline |
| `Set the text color based on the depth of the stroke color` | 根据描边颜色深浅设置文字颜色 | Derive text color from stroke |
| `Set the stroke color based on the depth of the text color` | 根据文字颜色深浅设置描边颜色 | Derive stroke from text |
| `Set font style based on text color` | 根据文字颜色匹配样式 | Match style via text color |
| `Set font style based on stroke color` | 根据描边颜色匹配样式 | Match style via stroke color |
| `Detect text direction and set the corresponding style` | 检测文字方向并设置对应样式 | Auto-detect text orientation |
| `Clear styles` | 清除样式 | Remove all style data |

#### Image Generation & Export

| English Name | Chinese Name | Description |
|-------------|-------------|-------------|
| `Generate translated pictures` | 生成成品图 | Render translated images |
| `Generate mask images for all` | 生成所有图片的掩膜 | Create inpainting masks |
| `Generate text-removed images for all` | 生成所有图片的去文字图片 | Create clean background images |
| `Generate mask and text-removed pictures for all` | 生成所有图片的掩膜和去文字图片 | Create both masks and clean images |
| `Export PDF` | 导出PDF | Export as searchable PDF |
| `Export markdown` | 导出Markdown | Export as Markdown file |
| `Remove text mask outside of text areas for all pictures` | 去除所有图片的框外掩膜 | Clean masks beyond box bounds |
| `Resize according to mask` | 根据掩膜调整大小 | Adjust box size to mask |

#### Other Operations

| English Name | Chinese Name | Description |
|-------------|-------------|-------------|
| `Detect rotation degree` | 识别旋转角度 | Detect text rotation angle |
| `Rotate images by text area degrees` | 根据文字旋转角度旋转图像 | Rotate images to straighten text |
| `Super resolution` | 超分辨率 | Upscale images |
| `Transliterate` | 注音 | Add furigana/ruby text |
| `Detect language in images to set OCR` | 检测图片中的语言以设置OCR | Auto-detect language for OCR |
| `Detect the language of the text and set the project's source language` | 根据文本检测并设置项目源语言 | Auto-detect source language |
| `Fix spelling of source text areas` | 修正所有区域的原文的拼写 | Spell-check source text |
| `Fix spelling of target text areas` | 修正所有区域的译文的拼写 | Spell-check translations |
| `Use balloon detection to remove excess areas and add missing areas` | 用气泡检测去除多余区域并补充缺失区域 | Refine text areas using balloon detection |
| `Create text areas using panels` | 使用分镜生成文字区域 | Generate text boxes from panels |
| `Delete mask images` | 删除掩膜图像 | Clean up mask files |
| `Delete text-removed images` | 删除无文字图像 | Clean up text-removed files |
| `Restore UI state` | 还原UI状态 | Restore previous UI state |

### Workflow Design Patterns

**Minimal OCR + Translate:**
```json
{
  "flow": [
    "Text detection",
    "Text recognition",
    "Translation"
  ]
}
```

**Full pipeline with export (manga/comics):**
```json
{
  "flow": [
    "Text detection (3)",
    "Text recognition (manga-ocr,0)",
    "Merge areas",
    "Sort",
    "Translation",
    "Detect text direction and set the corresponding style",
    "Generate translated pictures"
  ]
}
```

**Document translation with PDF export:**
```json
{
  "flow": [
    "Text detection",
    "Panel detection",
    "Sort panels",
    "Remove areas without source text for all pictures",
    "Remove areas within image and table panels",
    "Merge text areas within panels",
    "Sort",
    "Translation",
    "Export PDF"
  ]
}
```

**Export-only workflow (run on already-translated project):**
```json
{
  "flow": [
    "Generate translated pictures",
    "Export PDF",
    "Export markdown"
  ]
}
```

## Project File Format

The `.itp` file is a JSON file containing:

```json
{
  "settings": {
    "sourceLang": "ja",
    "targetLang": "en",
    "defaultInpainter": 0,
    "precisionMode": 0,
    "output_format": 0,
    ...
  },
  "images": {
    "page01.jpg": {
      "boxes": [
        {
          "x": 100, "y": 200, "width": 300, "height": 50,
          "text": "こんにちは",
          "target": "Hello",
          "class": "speech",
          ...
        }
      ],
      "panels": [...]
    }
  }
}
```

Key project settings:
- `sourceLang`: Source language code (e.g., `"ja"`, `"en"`, `"zh"`, `"ko"`)
- `targetLang`: Target language code
- `defaultInpainter`: Inpainting engine (0 = built-in)
- `output_format`: Output image format (0 = same as source)
- `precisionMode`: Higher quality rendering (0 or 1)

## Export Formats

ImageTrans supports these export formats, all usable from workflows:

| Format | Workflow Operation | Description |
|--------|-------------------|-------------|
| **Translated images** | `Generate translated pictures` | Rendered images with translated text in place |
| **PDF** | `Export PDF` | Searchable PDF with text layer and optional image layer |
| **Markdown** | `Export markdown` | Markdown with extracted text, supports page numbering |
| **TXT** | (GUI) | Plain text export |
| **XLSX** | (GUI) | Spreadsheet with source/target columns |
| **DOCX** | (GUI) | Word document with table |
| **XLIFF** | (GUI) | XLIFF translation exchange format |
| **TMX** | (GUI) | Translation memory exchange |

### Configuring Exports for CLI/Batch Use

When running via CLI (template mode or workflow mode), exports need proper configuration
to avoid GUI dialogs blocking the process:

**Translated images** — add `Generate translated pictures` to the workflow's `flow` array.
No additional settings are required.

**PDF export** — two things are needed:

1. Add `Export PDF` to the workflow's `flow` array.
2. In the project's `settings.json` (or template's), set `enable_pdf_export_options` to `true`
   and provide the options in `pdf_export_options`:
   ```json
   {
     "enable_pdf_export_options": true,
     "pdf_export_options": "{\"textMode\":1,\"imageMode\":0,\"fontPath\":\"\",\"lookupFontFile\":false,\"displayText\":false,\"addBookmarks\":false,\"enableImageCompression\":false,\"imageFormat\":0,\"jpegQuality\":85,\"adaptive\":false,\"removeNonTextPart\":false,\"byTextArea\":false}"
   }
   ```
   Note: `pdf_export_options` is a **JSON string** (serialized JSON), not a nested object.
   If `enable_pdf_export_options` is `false` or the options string is invalid/missing,
   a GUI dialog will pop up and block the CLI process.

**Markdown export** — same pattern as PDF:

1. Add `Export markdown` to the workflow's `flow` array.
2. In the project's `settings.json`, set `enable_markdown_export_options` to `true`
   and provide the options in `markdown_export_options`:
   ```json
   {
     "enable_markdown_export_options": true,
     "markdown_export_options": "{\"textMode\":1,\"addPageNumber\":false,\"convertSpecificClassesToImages\":false,\"mergePages\":true}"
   }
   ```
   Note: `markdown_export_options` is a **JSON string** (serialized JSON), not a nested object.
   If `enable_markdown_export_options` is `false` or the options string is invalid/missing,
   a GUI dialog will pop up and block the CLI process.

If these settings are not provided in CLI mode, the export steps will either show a GUI
dialog (blocking) or fail silently.

### Configuring PDF Import for CLI/Batch Use

When importing a PDF file via CLI (template mode), you need to configure the import
options beforehand to avoid GUI dialogs blocking the process:

In the project's `settings.json` (or template's), set `enable_pdf_import_options` to `true`
and provide the options in `pdf_import_options`:

```json
{
  "enable_pdf_import_options": true,
  "pdf_import_options": "{\"extractMode\":false,\"dpi\":300,\"extractText\":true,\"wordLevel\":false,\"pageStart\":1,\"pageEnd\":10}"
}
```

Note: `pdf_import_options` is a **JSON string** (serialized JSON), not a nested object.
If `enable_pdf_import_options` is `false` or the options string is invalid/missing,
a GUI dialog will pop up and block the CLI process.

#### PDF Import Options Reference

The `pdf_import_options` value is a JSON string containing:

| Option | Type | Values | Description |
|--------|------|--------|-------------|
| `extractMode` | bool | `true` | **Extract mode** — extract text and images directly from PDF without rendering pages as images. Text is obtained from the PDF's internal text stream. |
| | | `false` | **Render mode** — render PDF pages as images first, then run OCR on the images. Use this for scanned PDFs or when text extraction is unreliable. |
| `dpi` | int | e.g., `300` | Resolution (DPI) for rendering PDF pages to images. Only applies in render mode (`extractMode: false`). Higher values give better quality but larger images and slower processing. Typical range: 150–600. |
| `extractText` | bool | `true`/`false` | Whether to extract text from the PDF. In extract mode, this gets text from the PDF text stream. In render mode, this controls whether OCR is run on the rendered images. |
| `wordLevel` | bool | `true`/`false` | **Word-level extraction** — extract text at word granularity instead of line/block level. Only applicable when `extractText` is `true`. Gives finer-grained text boxes. |
| `pageStart` | int | e.g., `1` | First page to import (1-based). If omitted or `0`, starts from the first page. |
| `pageEnd` | int | e.g., `10` | Last page to import (1-based). If omitted or less than `pageStart`, imports to the last page. |

**Example — Render mode for scanned PDFs (no text layer):**
```json
{
  "enable_pdf_import_options": true,
  "pdf_import_options": "{\"extractMode\":false,\"dpi\":300,\"extractText\":false}"
}
```

**Example — Extract mode for digital PDFs with text layer:**
```json
{
  "enable_pdf_import_options": true,
  "pdf_import_options": "{\"extractMode\":true,\"extractText\":true,\"wordLevel\":false}"
}
```

### PDF Export Options Reference

The `pdf_export_options` value is a JSON string (see previous section for setup) containing:

| Option | Type | Values | Description |
|--------|------|--------|-------------|
| `textMode` | int | `0` | **Source text** — overlay original OCR text as a searchable text layer |
| | | `1` | **Target text** — overlay translated text as a searchable text layer |
| | | `2` | **No text layer** — no searchable text overlay |
| `imageMode` | int | `0` | **Original image** — use the source image as the page background |
| | | `1` | **Translated image** — use the rendered translated image |
| | | `2` | **Text-removed image** — use the inpainted (clean) image |
| | | `3` | **No image** — text layer only, no background image |
| `fontPath` | string | file path | Path to a `.ttf` or `.ttc` font file for the PDF text layer. Saved to `PDFFontPath` project setting for reuse. |
| `lookupFontFile` | bool | `true`/`false` | Map each text box's font style to a corresponding font file via `FontHelper`. When disabled, all text uses the single `fontPath` font. |
| `displayText` | bool | `true`/`false` | **Display text mode** — converts half-width characters to full-width and applies character replacement rules. Helps vertical text render correctly. |
| `addBookmarks` | bool | `true`/`false` | Add PDF bookmark outline from `bookmarks.txt` in the project directory (format: `filename: title`, one per line). |
| `enableImageCompression` | bool | `true`/`false` | Enable image re-compression before embedding in PDF. When disabled, the original image is used as-is. |
| `imageFormat` | int | `0` | **1-bit PNG** — binarize image (threshold/Otsu/adaptive), output as 1-bit PNG for smallest file size |
| | | `1` | **8-bit grayscale PNG** — convert to 8-bit grayscale PNG |
| | | `2` | **JPEG** — compress as JPEG with configurable quality |
| `jpegQuality` | int | `1`–`100` | JPEG quality level when `imageFormat` is `2` |
| `adaptive` | bool | `true`/`false` | Use **adaptive thresholding** for binarization (instead of Otsu or fixed threshold). Only applies when `imageFormat` is `0`. |
| `removeNonTextPart` | bool | `true`/`false` | **Remove non-text parts** — crop the image to only show text regions, leaving the rest white. Useful for clean text-only PDFs. |
| `byTextArea` | bool | `true`/`false` | **Threshold by text area** — apply binarization only within detected text regions, keeping non-text areas as-is. Only applies when `imageFormat` is `0`.

**Markdown Export Options** — a JSON string (see previous section for setup) containing:

| Option | Type | Values | Description |
|--------|------|--------|-------------|
| `textMode` | int | `0` | **Source only** — export original text |
| | | `1` | **Target only** — export translated text |
| | | `2` | **Source + Target** — export both |
| `addPageNumber` | bool | `true`/`false` | Insert `--- Page N ---` markers between pages |
| `convertSpecificClassesToImages` | bool | `true`/`false` | Convert panels whose class is in `panel_classes_to_convert_to_image` to inline images instead of text |
| `mergePages` | bool | `true`/`false` | Merge all pages into a single `.md` file. When `false`, each page gets its own file. |

**Note**: When `mergePages` is `true`, the CLI will show a save dialog unless a path
is pre-configured. When `false`, each page gets its own `.md` file in the `out` folder.

## Recipes: Common Tasks

### Recipe 1: Translate a Manga Chapter from Japanese to English

**Preparation** — the default `manga-ja2en` template workflow only includes
`Text detection`, `Merge areas`, `Sort`, and `Translation`. It does **not** include
`Generate translated pictures`. Before running the CLI, copy the template to a
temporary directory and edit the copy:

```bash
cp -r templates/manga-ja2en /tmp/manga-ja2en-custom/
```

Then edit `/tmp/manga-ja2en-custom/custom_workflow.json` and add
`"Generate translated pictures"` to the end of the `flow` array:

```json
"flow": [
  "Text detection",
  "Merge areas",
  "Sort",
  "Translation",
  "Generate translated pictures"
]
```

Then use the temporary copy as the template path in the CLI command below.

**Windows:**
```bash
cd /path/to/ImageTrans
jre/bin/java.exe \
  --module-path jre/javafx/lib \
  --add-modules javafx.base,javafx.controls,javafx.graphics,javafx.web,javafx.swing \
  --add-opens javafx.controls/com.sun.javafx.scene.control.skin=ALL-UNNAMED \
  --add-exports javafx.base/com.sun.javafx.collections=ALL-UNNAMED \
  --add-exports java.desktop/sun.awt=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.bmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.gif=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.wbmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.spi=ALL-UNNAMED \
  --add-opens java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  -jar ImageTrans.jar "/tmp/manga-ja2en-custom" "C:/manga/chapter01/"
```

**Linux:**
```bash
cd /path/to/ImageTrans
jre/bin/java \
  -Djava.library.path=./ \
  --module-path jre/javafx/lib \
  --add-modules javafx.base,javafx.controls,javafx.graphics,javafx.web,javafx.swing \
  --add-opens javafx.controls/com.sun.javafx.scene.control.skin=ALL-UNNAMED \
  --add-exports javafx.base/com.sun.javafx.collections=ALL-UNNAMED \
  --add-exports java.desktop/sun.awt=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.bmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.gif=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.wbmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.spi=ALL-UNNAMED \
  --add-opens java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  -jar ImageTrans.jar "/tmp/manga-ja2en-custom" "/home/user/manga/chapter01/"
```

This imports all images from the directory, runs the manga-ja2en workflow, and exits.
Translated images are saved to an `out` folder inside the project directory
(e.g., `C:/manga/chapter01/out/`).

### Recipe 2: Translate a PDF Document and Export as Searchable PDF

**Preparation** — the default `document` template workflow may not include `Export PDF`.
Before running the CLI, copy the template to a temporary directory and edit the copy:

```bash
cp -r templates/document /tmp/document-custom/
```

Then:

1. Edit `/tmp/document-custom/custom_workflow.json` and add `"Export PDF"` to the end of
   the `flow` array (same pattern as Recipe 1).

2. Edit `/tmp/document-custom/settings.json` and add PDF import and export options so the CLI runs
   without popping up a GUI dialog:
   ```json
   {
     "enable_pdf_import_options": true,
     "pdf_import_options": "{\"extractMode\":true,\"extractText\":true,\"wordLevel\":false}",
     "enable_pdf_export_options": true,
     "pdf_export_options": "{\"textMode\":1,\"imageMode\":1,\"fontPath\":\"\",\"lookupFontFile\":false,\"displayText\":false,\"addBookmarks\":false,\"enableImageCompression\":false,\"imageFormat\":0,\"jpegQuality\":85,\"adaptive\":false,\"removeNonTextPart\":false,\"byTextArea\":false}"
   }
   ```
   See [PDF Import Options Reference](#pdf-import-options-reference) and [PDF Export Options Reference](#pdf-export-options-reference) for details on each option.

Then use the temporary copy as the template path in the CLI command below.

**Windows:**
```bash
cd /path/to/ImageTrans
jre/bin/java.exe \
  --module-path jre/javafx/lib \
  --add-modules javafx.base,javafx.controls,javafx.graphics,javafx.web,javafx.swing \
  --add-opens javafx.controls/com.sun.javafx.scene.control.skin=ALL-UNNAMED \
  --add-exports javafx.base/com.sun.javafx.collections=ALL-UNNAMED \
  --add-exports java.desktop/sun.awt=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.bmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.gif=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.wbmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.spi=ALL-UNNAMED \
  --add-opens java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  -jar ImageTrans.jar "/tmp/document-custom" "C:/docs/japanese_report.pdf"
```

**Linux:**
```bash
cd /path/to/ImageTrans
jre/bin/java \
  -Djava.library.path=./ \
  --module-path jre/javafx/lib \
  --add-modules javafx.base,javafx.controls,javafx.graphics,javafx.web,javafx.swing \
  --add-opens javafx.controls/com.sun.javafx.scene.control.skin=ALL-UNNAMED \
  --add-exports javafx.base/com.sun.javafx.collections=ALL-UNNAMED \
  --add-exports java.desktop/sun.awt=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.bmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.gif=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.plugins.wbmp=ALL-UNNAMED \
  --add-exports java.desktop/com.sun.imageio.spi=ALL-UNNAMED \
  --add-opens java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
  -jar ImageTrans.jar "/tmp/document-custom" "/home/user/docs/japanese_report.pdf"
```

The output PDF is saved to the `out` folder inside the project directory
(e.g., `C:/docs/japanese_report/japanese_report.pdf`).

### Recipe 3: Export Translated Images and Markdown from Existing Project

Create or modify `custom_workflow.json` in the project directory:

```json
[{
  "name": "export-only",
  "flow": [
    "Generate translated pictures",
    "Export markdown"
  ],
  "ocrInterval": 0,
  "detectionMethod": 0,
  "engine": "google",
  "type": "MT"
}]
```

Then run:
```bash
cd /path/to/ImageTrans
... -jar ImageTrans.jar "C:/project/project.itp" 0 workflow
```

### Recipe 4: Batch Process Multiple PDFs

For each PDF, use the template mode.

**Windows PowerShell:**
```powershell
$pdfs = Get-ChildItem "C:/pdfs/*.pdf"
foreach ($pdf in $pdfs) {
  & jre/bin/java.exe `
    --module-path jre/javafx/lib `
    --add-modules javafx.base,javafx.controls,javafx.graphics,javafx.web,javafx.swing `
    --add-opens javafx.controls/com.sun.javafx.scene.control.skin=ALL-UNNAMED `
    --add-exports javafx.base/com.sun.javafx.collections=ALL-UNNAMED `
    --add-exports java.desktop/sun.awt=ALL-UNNAMED `
    --add-exports java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED `
    --add-exports java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED `
    --add-exports java.desktop/com.sun.imageio.plugins.bmp=ALL-UNNAMED `
    --add-exports java.desktop/com.sun.imageio.plugins.gif=ALL-UNNAMED `
    --add-exports java.desktop/com.sun.imageio.plugins.wbmp=ALL-UNNAMED `
    --add-exports java.desktop/com.sun.imageio.spi=ALL-UNNAMED `
    --add-opens java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED `
    -jar ImageTrans.jar "templates/document" $pdf.FullName
}
```

**Linux bash:**
```bash
cd /path/to/ImageTrans
for pdf in /home/user/pdfs/*.pdf; do
  jre/bin/java \
    -Djava.library.path=./ \
    --module-path jre/javafx/lib \
    --add-modules javafx.base,javafx.controls,javafx.graphics,javafx.web,javafx.swing \
    --add-opens javafx.controls/com.sun.javafx.scene.control.skin=ALL-UNNAMED \
    --add-exports javafx.base/com.sun.javafx.collections=ALL-UNNAMED \
    --add-exports java.desktop/sun.awt=ALL-UNNAMED \
    --add-exports java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
    --add-exports java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED \
    --add-exports java.desktop/com.sun.imageio.plugins.bmp=ALL-UNNAMED \
    --add-exports java.desktop/com.sun.imageio.plugins.gif=ALL-UNNAMED \
    --add-exports java.desktop/com.sun.imageio.plugins.wbmp=ALL-UNNAMED \
    --add-exports java.desktop/com.sun.imageio.spi=ALL-UNNAMED \
    --add-opens java.desktop/com.sun.imageio.plugins.jpeg=ALL-UNNAMED \
    -jar ImageTrans.jar "templates/document" "$pdf"
done
```

### Recipe 5: Create a Project with Custom Settings (Workflow Mode)

When you need custom settings not covered by templates:

1. Copy a template directory:
   ```
   general_template/ → C:/my_project/
   ```

2. Edit `settings.json`:
   ```json
   {
     "sourceLang": "ko",
     "targetLang": "en"
   }
   ```

3. Edit `custom_workflow.json` with your workflow steps.

4. Import images via the GUI once, or use template mode first.

5. Run subsequent workflows via workflow mode:
   ```bash
   ... -jar ImageTrans.jar "C:/my_project/1.itp" 0 workflow
   ```

## Troubleshooting

### Common Issues

**"Java not found" or JRE path wrong:**
Verify the bundled JRE exists:
- Windows: `jre/bin/java.exe`
- Linux: `jre/bin/java`
- macOS: `<IMAGETRANS_DIR>/jdk-23/Contents/Home/bin/java` (default: `/Applications/ImageTrans.app/Contents/Resources/ImageTrans`)

If missing, install a Java 11+ JDK with JavaFX and use system `java`.

**Native library loading fails (UnsatisfiedLinkError):**
- **Linux / macOS**: Ensure `-Djava.library.path=./` is included in the JVM arguments.
- **Windows**: This flag is not needed (Windows searches the current directory for DLLs
  by default). Verify that native libraries (`.dll`) exist in the ImageTrans
  installation directory.

**Headless mode issues:**
`-Dglass.platform=headless` requires **JavaFX 26 or later**. The bundled JRE ships
with JavaFX 23, which does not support this flag. If you see errors like
`java.awt.HeadlessException` or `GraphicsEnvironment.isHeadless()`, remove the
`-Dglass.platform=headless` flag. Only use it if you have manually upgraded to
JavaFX 26+. On Linux servers without a display, use `xvfb-run` instead.

**Application exits with error code 1:**
Check the log output. Common causes:
- Invalid template path
- Missing `custom_workflow.json` in project directory
- OCR engine not configured (check `preferences.conf`)
- MT engine credentials not set

**Workflow runs but no output:**
Ensure the workflow includes an export step (`Generate translated pictures`, `Export PDF`, or `Export markdown`).
Without these, only the `.itp` project file is updated.

**"customflow done" does not appear:**
The application may have crashed or hung. Check if:
- All images in the project exist on disk
- The OCR engine is responsive
- Network is available (for cloud MT/OCR engines)

**PDF import takes too long:**
Large PDFs are processed page by page. Consider splitting into smaller files,
or increase the OCR interval (`ocrInterval`) to avoid API rate limiting.

### Finding Output Files

- **Translated images**: In the project's output directory, files with the configured output format
- **PDF export**: `File.GetName(projectPath) & ".pdf"` in the project's output directory
- **Markdown export**: `File.GetName(projectPath) & ".md"` in the project's output directory
- **Project file**: The `.itp` file is updated in place

By default the output directory is the `out` directory in the input directory, unless overridden in `settings.json`. It supports recursive subdirectories.

### Debug Mode

To see more verbose output, run the command and capture stdout/stderr. The application
logs progress information, including workflow steps being executed and any errors.

For workflow debugging, check that the operation names in `custom_workflow.json`
match the expected names exactly. The operations list in this document shows both
English and Chinese variants.
