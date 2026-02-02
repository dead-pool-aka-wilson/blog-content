---
name: meme-adapt
description: Adapt memes and visuals for blog posts using Gemini. Activate when user wants to customize a meme, add text overlays, or modify images for content. 밈/이미지를 블로그용으로 수정. 밈 커스터마이징, 텍스트 오버레이 추가 시 활성화.
---

# Meme Adaptation Skill

Source memes → Gemini adaptation → Blog-ready visuals.

## When to Activate

- User says "이 밈 수정해줘" / "adapt this meme"
- User wants to add text to an image
- User needs visual content for a blog post
- User says "밈 만들어줘" / "make a meme for this"

## Visual Library Location

```
~/Dev/.content-seeds/visuals/
├── sources/      # Original memes (user collected)
└── adapted/      # Gemini-modified versions
```

## Adaptation Workflow

### Step 1: Identify Source

```bash
# List available source memes
ls ~/Dev/.content-seeds/visuals/sources/

# Check metadata for a source
cat ~/Dev/.content-seeds/visuals/sources/[name].meta.md
```

Or user provides image directly.

### Step 2: Understand Context

Before adapting, gather:
- **Target post topic**: What is the blog post about?
- **Tone**: Serious, humorous, sarcastic, educational?
- **Text needed**: What should the meme say?
- **Constraints**: Keep format? Change colors? Add elements?

### Step 3: Gemini Adaptation

**Use Gemini with image input:**

```
/gemini or use look_at tool with Gemini

Prompt structure:
"I have this meme [describe or attach]. 
For a blog post about [topic], modify it to:
- [specific change 1]
- [specific change 2]
Keep the original meme format and style.
Output as [format: same/png/jpg]."
```

**Common adaptation types:**

| Type | Prompt Pattern |
|------|----------------|
| Text replacement | "Replace the text. Top: '[new top]', Bottom: '[new bottom]'" |
| Label meme | "Change the labels to: [A] = '[meaning]', [B] = '[meaning]'" |
| Reaction context | "Keep the reaction image, add context: '[situation]'" |
| Style transfer | "Redraw this in [style] while keeping the message" |
| Diagram annotation | "Add arrows/labels pointing to [elements]" |

### Step 4: Save & Track

```bash
# Save adapted image
# Move Gemini output to:
~/Dev/.content-seeds/visuals/adapted/[descriptive-name].jpg

# Create metadata
cp ~/Dev/.content-seeds/visuals/adapted/_template.meta.md \
   ~/Dev/.content-seeds/visuals/adapted/[descriptive-name].meta.md

# Fill in:
# - source_file: link to original
# - adaptation_prompt: what you asked Gemini
# - tags: for future retrieval
```

### Step 5: Use in Post

```bash
# Copy to blog static folder
cp ~/Dev/.content-seeds/visuals/adapted/[name].jpg \
   ~/Dev/personal-blog/static/images/posts/YYYY-MM-DD-slug/

# Update adapted metadata with usage
# Add to used_in_posts, increment times_used
```

## Prompt Templates

### Drake Format
```
Modify this Drake meme:
- Top panel (disapproval): "[thing to reject]"
- Bottom panel (approval): "[thing to prefer]"
Topic: [your topic]
Keep Drake's expressions exactly as-is.
```

### Distracted Boyfriend
```
Modify this distracted boyfriend meme:
- Boyfriend label: "[who is distracted]"  
- Girlfriend label: "[what they should focus on]"
- Other woman label: "[the distraction]"
Context: [your situation]
```

### Two Buttons / Hard Choice
```
Modify this two buttons meme:
- Button 1: "[option 1]"
- Button 2: "[option 2]"
- Sweating person: "[who faces this choice]"
```

### Expanding Brain
```
Create expanding brain meme:
- Small brain: "[basic approach]"
- Medium brain: "[better approach]"  
- Large brain: "[advanced approach]"
- Galaxy brain: "[your punchline/absurd take]"
```

### Custom Diagram
```
Create a simple diagram showing:
- [Element A] → [Element B] → [Element C]
- Label the arrows: "[relationship 1]", "[relationship 2]"
Style: minimal, clean, blog-appropriate
Colors: [specify or "match my blog theme"]
```

## Quality Checklist

- [ ] Source meme identified or provided
- [ ] Target post context understood
- [ ] Adaptation prompt crafted with specifics
- [ ] Gemini output reviewed for quality
- [ ] Text readable and correctly placed
- [ ] Saved to adapted/ with metadata
- [ ] Source metadata updated (times_adapted++)
- [ ] Ready for blog integration

## Tips

1. **Be specific**: "Replace top text" > "modify the meme"
2. **Preserve format**: Ask to keep the original meme structure
3. **Check readability**: Ensure text is legible at blog size
4. **Track everything**: Metadata enables reuse and learning
5. **Iterate**: If first attempt isn't right, refine the prompt

## Anti-patterns

- Don't generate generic AI images when a real meme fits better
- Don't over-modify - the recognizable format IS the joke
- Don't forget attribution in metadata (source_url)
- Don't skip metadata - future you will thank present you

---

**Remember**: Good meme adaptation enhances the original format with your content. The meme should feel native to its format while delivering your message.
