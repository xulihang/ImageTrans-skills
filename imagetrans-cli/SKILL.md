---
name: imagetrans-cli
description: >-
  Guide for operating ImageTrans via command line to OCR, translate, and export
  images and PDFs. Use this skill whenever the user wants to batch process images/PDFs
  through OCR and translation, run automated ImageTrans workflows, start ImageTrans
  in server mode, export translated images/PDFs/markdown, or perform any ImageTrans
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
- **OCR**: Extract text from images using Tesseract, cloud OCR engines, or specialized manga OCR
- **Translation**: Machine translate extracted text via multiple engines (Baidu, Google, DeepL, etc.)
- **Text removal & inpainting**: Remove original text and generate clean backgrounds
- **Export**: Output translated images, searchable PDFs, Markdown, DOCX, XLIFF, TXT

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

All CLI invocations share the same base command. The `-Djava.library.path=./` flag
is **required on all platforms** so Java can find native libraries (OpenCV, ONNX Runtime, etc.)
shipped in the ImageTrans directory.

```bash
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

To run ImageTrans without showing any GUI windows — useful for batch processing,
server environments, or automated pipelines — add the JavaFX headless platform flag:

```
-Dglass.platform=headless
```

This suppresses all JavaFX windows. The CLI process runs purely in the background
and exits when the workflow completes. Place it alongside the other `-D` flags:

```bash
<JAVA_PATH> \
  -Djava.library.path=./ \
  -Dglass.platform=headless \
  --module-path <JAVAFX_LIB> \
  ... \
  -jar ImageTrans.jar <ARGS>
```

**When to use headless mode:**
- Batch processing — no need to see the GUI while images are being processed
- CI/CD pipelines or automated scripts
- Linux servers without X11/Wayland, Docker containers, WSL without an X server
- Windows/macOS when you want the process to run silently in the background

**When NOT to use headless mode:**
- When you need to visually inspect or manually adjust results
- When using the full GUI for interactive work
- Server Mode may not function fully in headless mode since JavaFX windows are used internally

Note: headless mode works on **all platforms** (Windows, Linux, macOS). It works
particularly well with Template Mode and Workflow Mode where no user interaction is needed.

## CLI Modes

ImageTrans has four CLI invocation modes, selected by the number and content of arguments.

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
2. Imports the image(s) or PDF pages
3. Applies `settings.json` (source/target language, etc.) and `preferences.conf` if present
4. Runs the default workflow (index 0 from `custom_workflow.json`)
5. Saves the project and exits automatically

**Use this mode when:** You want to process new images/PDFs with zero manual setup.
The template provides all configuration — OCR engine, translation engine, workflow steps.

**Example — Translate a manga page (Windows):**
```bash
cd /path/to/ImageTrans
jre/bin/java.exe \
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
  -jar ImageTrans.jar "Objects/templates/manga" "C:/manga/page01.jpg"
```

**Example — Translate a manga page (Linux, headless):**
```bash
cd /path/to/ImageTrans
jre/bin/java \
  -Djava.library.path=./ \
  -Dglass.platform=headless \
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
  -jar ImageTrans.jar "Objects/templates/manga" "/home/user/manga/page01.jpg"
```

**Example — Translate a PDF document:**
```bash
cd /path/to/ImageTrans
# Use document template for PDF processing
... -jar ImageTrans.jar "Objects/templates/document" "C:/docs/report.pdf"
```

---

### Mode 2: Workflow Mode (Run Workflow on Existing Project)

```
<JAVA> ... -jar ImageTrans.jar <projectPath> <workflowIndex>
```

**Arguments:**
- `projectPath`: Path to an existing `.itp` project file
- `workflowIndex`: Integer index into the `custom_workflow.json` array (0-based)

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
... -jar ImageTrans.jar "C:/manga/project.itp" 2
```

**Example — Run the first (default) workflow:**
```bash
... -jar ImageTrans.jar "C:/manga/project.itp" 0
```

---

### Mode 3: Server Mode (WebSocket API)

```
<JAVA> ... -jar ImageTrans.jar <projectPath> server
```

**Arguments:**
- `projectPath`: Path to an existing `.itp` project file
- `server`: The literal string `"server"`

**Behavior:**
1. Opens the project
2. Starts a WebSocket server at `ws://127.0.0.1:51042/imagetrans`
3. Also exposes HTTP at `http://127.0.0.1:51042`
4. The process remains running, accepting WebSocket commands

**Use this mode when:** You need programmatic control, real-time interaction, or
integration with external tools (e.g., browser extensions, OCR capture workflows).

#### WebSocket Protocol

The server communicates via JSON messages over WebSocket.

**Sending commands (client → server):**
The server receives commands by listening for function calls from connected clients.
Commands are dispatched to `wsh_<EventName>(Messages As List)` handlers.

**Core commands:**

| Command | Parameters | Description |
|---------|-----------|-------------|
| `TranslateRegion` | `[src, sourceLang, targetLang]` | OCR + translate an image region |
| `Translate` | `[...]` | Full translation pipeline for an image |
| `heartbeatReceived` | `[...]` | Keep-alive acknowledgment |

**Server events (server → client):**

| Event | Data | Description |
|-------|------|-------------|
| `set_running` | `{running: bool}` | Translation state change |
| `set_translated` | `{success, output, imgMap}` | Translation completed |
| `set_name_and_password` | `{name, password}` | Server identity on connect |
| `close_server` | `{success}` | Server shutdown notification |
| `keep_alive` | `{...}` | Periodic keep-alive ping |

**Web Client:**
A web UI is available for server mode:
- Local: `https://local.basiccat.org:51043`
- Remote (if configured): `https://service.basiccat.org:51043`

#### Connecting to the Server

When writing automation scripts, connect via WebSocket to `ws://127.0.0.1:51042/imagetrans`,
then send JSON messages:

```json
{
  "type": "event",
  "event": "TranslateRegion",
  "params": {
    "src": "image_path_or_url",
    "sourceLang": "ja",
    "targetLang": "zh"
  }
}
```

The server responds with events on the same WebSocket connection.


### Mode 0: Open Project (1 Argument, GUI Mode)

```
<JAVA> ... -jar ImageTrans.jar <projectPath>
```

Opens the project in the GUI. Not typically used for automation but useful for
manual inspection after batch processing.

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

ImageTrans ships with these templates in `Objects/templates/`:

| Template | Use Case | Source→Target |
|----------|----------|---------------|
| `manga` | Japanese manga translation | ja→? |
| `manga-ja2en` | Japanese to English manga | ja→en |
| `manga-ja2zh` | Japanese to Chinese manga | ja→zh |
| `comics` | Western comics | en→? |
| `webtoon` | Korean/Chinese webtoons | ? |
| `document` | Document translation with panel detection | ? |
| `cg` | Computer graphics / game screenshots | ? |
| `cg-ja2zh` | Japanese to Chinese game screenshots | ja→zh |
| `chinese-manhua` | Chinese manhua | zh→? |

### Creating a Custom Template

To create a template for a new language pair or use case:

1. **Create the directory structure:**
   ```
   my-template/
   ├── 1.itp
   ├── custom_workflow.json
   └── settings.json
   ```

2. **`settings.json`** — Set source and target languages:
   ```json
   {
     "sourceLang": "ja",
     "targetLang": "en"
   }
   ```

3. **`custom_workflow.json`** — Define your workflow (see Workflow Configuration below).

4. **`1.itp`** — Copy from `Objects/general_template/1.itp` and adjust settings
   (OCR engine, inpainting options, etc.).

5. **`preferences.conf`** (optional) — Override app preferences:
   ```json
   {
     "api": {
       "chatGPT": {"key":"api key"}
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
| `0` | OCR-based | Use OCR engines' text detection capabilities and recognize text at the same time |
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

**PDF Export Options** (set in project settings `pdf_export_options`):
- `textMode`: 0=source+target, 1=target only, 2=no text layer
- `imageMode`: 0=original image, 1=translated image, 2=text-removed image
- `fontPath`: Path to a font file for the text layer
- `enableImageCompression`: JPEG compression for images
- `jpegQuality`: JPEG quality (1-100)

**Markdown Export Options** (set in project settings `markdown_export_options`):
- `textMode`: 0=source only, 1=target only, 2=source+target
- `addPageNumber`: Insert page number markers
- `convertSpecificClassesToImages`: Convert certain panel classes to images
- `mergePages`: Merge all pages into a single markdown file

## Recipes: Common Tasks

### Recipe 1: Translate a Manga Chapter from Japanese to English

**Windows:**
```bash
cd /path/to/ImageTrans
jre/bin/java.exe \
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
  -jar ImageTrans.jar "Objects/templates/manga-ja2en" "C:/manga/chapter01/"
```

**Linux (headless):**
```bash
cd /path/to/ImageTrans
jre/bin/java \
  -Djava.library.path=./ \
  -Dglass.platform=headless \
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
  -jar ImageTrans.jar "Objects/templates/manga-ja2en" "/home/user/manga/chapter01/"
```

This imports all images from the directory, runs the manga-ja2en workflow, and exits.
Output files (translated images) will be in the same directory.

### Recipe 2: Translate a PDF Document and Export as Searchable PDF

```bash
cd /path/to/ImageTrans
# Step 1: Use document template to process the PDF
jre/bin/java.exe \
  ... \
  -jar ImageTrans.jar "Objects/templates/document" "C:/docs/japanese_report.pdf"
```

If the workflow includes `Export PDF`, the output PDF is generated automatically.
Otherwise, edit the template's `custom_workflow.json` to add `Export PDF` to the flow.

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
... -jar ImageTrans.jar "C:/project/project.itp" 0
```

### Recipe 4: Batch Process Multiple PDFs

For each PDF, use the template mode.

**Windows PowerShell:**
```powershell
$pdfs = Get-ChildItem "C:/pdfs/*.pdf"
foreach ($pdf in $pdfs) {
  & jre/bin/java.exe `
    -Djava.library.path=./ `
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
    -jar ImageTrans.jar "Objects/templates/document" $pdf.FullName
}
```

**Linux bash (headless):**
```bash
cd /path/to/ImageTrans
for pdf in /home/user/pdfs/*.pdf; do
  jre/bin/java \
    -Djava.library.path=./ \
    -Dglass.platform=headless \
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
    -jar ImageTrans.jar "Objects/templates/document" "$pdf"
done
```

### Recipe 5: Start Server Mode for External Tool Integration

```bash
cd /path/to/ImageTrans
... -jar ImageTrans.jar "C:/project/project.itp" server
```

The server starts and stays running. External tools can then connect to
`ws://127.0.0.1:51042/imagetrans` to send translation requests.

### Recipe 6: Create a Project with Custom Settings (Workflow Mode)

When you need custom settings not covered by templates:

1. Copy a template directory:
   ```
   Objects/general_template/ → C:/my_project/
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
   ... -jar ImageTrans.jar "C:/my_project/1.itp" 0
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
Ensure `-Djava.library.path=./` is included in the JVM arguments, and verify that
native libraries (`.dll` on Windows, `.so` on Linux, `.dylib` on macOS) exist in
the ImageTrans installation directory.

**Headless mode issues:**
If you see errors about `java.awt.HeadlessException` or `GraphicsEnvironment.isHeadless()`,
add `-Dglass.platform=headless` to the JVM arguments. This applies to all platforms.
Conversely, if Server Mode fails to start in headless mode, remove this flag and ensure
a display is available (e.g., with `xvfb-run` on Linux servers).

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
