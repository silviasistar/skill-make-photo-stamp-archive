# Make Photo Stamp Archive

把照片转成克制、清爽的档案拼接作品：一侧忠实保留原照片，另一侧使用暖白纸张与定制手工图章。

A Codex Skill for transforming photos into restrained archival composites: a faithfully preserved photograph joined directly to a warm-white paper panel with a custom hand-pressed seal.

## 特点 / Features

- 忠实保留原照片内容、人物、建筑、文字与色彩关系
- 默认采用横向左右直拼，保持清晰笔直的分界
- 根据主体设计圆形、方形、拱形或异形图章
- 使用克制的旧纸纹理、干墨、磨损网点与少量套色偏差
- 支持多张照片分别输出，以及图章形状、大小、位置和纸张质感的迭代修改

- Preserves the source photograph faithfully
- Uses a clean direct-splice archival composition
- Creates subject-specific circular, square, arched, panoramic, or irregular seals
- Supports iterative changes to seal shape, scale, position, ink color, and paper age

## 目录 / Structure

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    └── prompt-template.md
```

## 安装 / Installation

```bash
git clone https://github.com/Dlcccc71913/skill-make-photo-stamp-archive.git
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R skill-make-photo-stamp-archive "${CODEX_HOME:-$HOME/.codex}/skills/make-photo-stamp-archive"
```

## 使用 / Usage

```text
使用 $make-photo-stamp-archive，把这张照片制作成带定制图章的档案拼接作品。
```

```text
Use $make-photo-stamp-archive to turn this photo into a direct-splice archival artwork with a custom seal.
```

## License

[MIT](LICENSE)
