# Visual Content Library

Memes, diagrams, and images for blog posts.

## Structure

```
visuals/
├── sources/           # Original memes/images you collect
│   ├── image.jpg
│   └── image.meta.md  # Metadata sidecar file
└── adapted/           # Gemini-modified versions
    ├── adapted.jpg
    └── adapted.meta.md
```

## Workflow

### 1. Collect Source

```bash
# Save image to sources/
cp ~/Downloads/funny-meme.jpg .content-seeds/visuals/sources/

# Create metadata (copy template, rename to match image)
cp .content-seeds/visuals/sources/_template.meta.md \
   .content-seeds/visuals/sources/funny-meme.meta.md
```

### 2. Adapt with Gemini

Use `/meme-adapt` skill or manually:

```
# In OpenCode with Gemini
"Take this meme [attach image] and replace the text with:
- Top: 'When the code works'
- Bottom: 'But you don't know why'
Keep the same style and format."
```

### 3. Save Adapted Version

```bash
# Save Gemini output to adapted/
# Create metadata linking back to source
```

### 4. Use in Post

When using in a blog post:
1. Copy to `personal-blog/static/images/posts/YYYY-MM-DD-slug/`
2. Update `used_in_posts` in the adapted metadata
3. Reference in post: `![alt](../images/posts/YYYY-MM-DD-slug/image.jpg)`

## Tagging Convention

### Platform Tags
- `twitter`, `tiktok`, `reddit`, `instagram`

### Format Tags
- `drake-format`, `distracted-boyfriend`, `expanding-brain`, `two-buttons`
- `comparison`, `reaction`, `label-meme`

### Topic Tags
- `programming`, `ai`, `debugging`, `frontend`, `devops`
- `meta`, `philosophy`, `productivity`

## Finding Visuals

```bash
# Search by tag
grep -l "programming" visuals/**/*.meta.md

# Find unused sources
grep -l "times_adapted: 0" visuals/sources/*.meta.md

# Find high-quality adaptations for reuse
grep -l "reuse_potential: high" visuals/adapted/*.meta.md
```
