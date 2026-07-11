# ImageTrans-skills

Agent skills for ImageTrans. You can use the skills in an AI agent like Claude Code, Codex to use ImageTrans to translate images, perform OCR and convert PDF to markdown, etc.

## Installation

```
npx skills add https://github.com/xulihang/ImageTrans-skills
```

## Example Prompts

### English

```
Translate Japanese manga images in the directory to Chinese. Use Deepseek for translation. ImageTrans directory: C:\Users\HP\Documents\GitHub\ImageTrans\Objects
```

```
Convert PDFs in the directory to a single markdown file, add page number information, only need the original text, no translation needed. Don't OCR, just extract text from PDF (word level), then use panel detection for layout analysis to determine paragraph order. Use document template /imagetrans-cli ImageTrans directory: C:\Users\HP\Documents\GitHub\ImageTrans\Objects
```

### Chinese

```
将目录的日语漫画图片翻译成中文。使用Deepseek进行翻译。ImageTrans目录：C:\Users\HP\Documents\GitHub\ImageTrans\Objects
```

```
将目录的pdf转换成单个markdown文件，添加页码信息，只需要原文，不需要翻译。不要OCR，只需提取pdf中的文本（单词级别），然后使用分镜检测做布局分析，确定段落的顺序。使用文档模板 /imagetrans-cli ImageTrans目录：C:\Users\HP\Documents\GitHub\ImageTrans\Objects
```