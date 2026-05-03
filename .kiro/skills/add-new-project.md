# Adding a New Project to the Portfolio

Follow this guide exactly when adding a new project entry to `_projects/`.

---

## Step 1: Create the Project Folder and File

Create a new folder inside `_projects/` named after the project. Then create `index.md` inside it.

```
_projects/
└── Your Project Name/
    ├── index.md
    └── (image files)
```

---

## Step 2: Add the Front Matter

The front matter goes at the very top of `index.md`. No spaces before the `---` delimiters — this causes a Jekyll build error.

```yaml
---
layout: post
title: Full Project Title Here
description: >
  One to three sentence summary of the project. This appears on the main portfolio
  page card and at the top of the project page. Be concise but informative.
skills:
  - Skill 1
  - Skill 2
  - Skill 3
main-image: /your-main-image.png
---
```

- `main-image` path is relative to the project folder (just `/filename.ext`)
- `skills` are short tags shown on the project card

---

## Step 3: Write the Project Body

Use the following section structure. All sections use `##` (H2) as top-level headers. Use `###` (H3) for subsections.

---

### Required Sections and Order

#### 1. Project Overview (optional intro block)
A brief institutional/context line if relevant. Example:
```markdown
**Institution**: University Name – Lab Name  
**Role**: Lead Researcher  
**Tools**: Python, Pyomo, Gurobi  
```

---

#### 2. Objective
```markdown
# Objective
State the main goal of the project clearly. What problem does it solve? What was the system or framework being developed?
```

---

#### 3. Methodology
```markdown
# Methodology
- Describe the approach, tools, and workflow used.
- Use numbered lists for sequential steps.
- Use bullet points for parallel items.
1. Step one
2. Step two
```

---

#### 4. Assumptions
```markdown
# Assumptions
- List key simplifications or boundary conditions used in the analysis.
- Be specific with values where relevant (e.g., "Diesel price: 0.67 USD/L").
```

---

#### 5. Results
```markdown
# Results
- Summarize the key quantitative and qualitative outcomes.
- Use subsections (##) for different result categories if needed.
- Include tables for sizing results or comparison data.

## Subsection Title (e.g., Optimal Sizing Results)
<br>

| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| value    | value    | value    |
```

---

#### 6. Significance & Impact
```markdown
# Significance & Impact
- Why does this project matter?
- What skills or insights does it demonstrate?
- What real-world applications or implications does it have?
```

---

#### 7. Key Takeaway
```markdown
# Key Takeaway
- Concise bullet points summarizing the most important lessons or conclusions.
- What would someone remember from this project?
```

---

#### 8. Key Figures
```markdown
# Key Figures
## Figure Group Title (e.g., System Performance)
### Individual Figure Title

![](/_projects/Your Project Folder Name/image-filename.png){:height="400px"}

<br>
Caption or brief description of what the figure shows.
<br>
```

**This is the correct image syntax.** Always use the full project folder path:
```markdown
![](/_projects/Your Project Folder Name/image.png){:height="400px"}
```

For multiple images side by side, place them on consecutive lines:
```markdown
![](/_projects/Your Project Folder Name/img1.png){:height="400px"}
![](/_projects/Your Project Folder Name/img2.png){:height="400px"}
```

Do NOT use `{% include image-gallery.html %}` — it does not render correctly.

---

#### 9. Code (if applicable)
```markdown
# [Language] Code
## Section Title (e.g., Main Optimization Script)

```python
# your code here
```

## Another Section (e.g., Parameters)

```python
# parameters here
```
```

Use the correct language tag for syntax highlighting: `python`, `matlab`, `javascript`, `r`, etc.

---

## Step 4: Add Images

Place all image files in the same project folder as `index.md`. Reference them using the path:
- In `image-gallery` include: `/filename.ext` (just the filename with leading slash)
- In inline markdown: `/_projects/Your Project Folder Name/filename.ext`

---

## Complete Template

```markdown
---
layout: post
title: Project Title
description: >
  Short description for the portfolio card and project page header.
skills:
  - Skill 1
  - Skill 2
main-image: /main-image.png
---

**Institution**: Name (if applicable)  
**Role**: Your role  
**Tools**: Tools used  

# Objective
Describe the goal of the project.

# Methodology
- Approach overview
1. Step 1
2. Step 2

# Assumptions
- Assumption 1
- Assumption 2 with value (e.g., efficiency: 95%)

# Results
Summary of outcomes.

## Key Results Table
<br>

| Metric | Value |
|--------|-------|
| ...    | ...   |

# Significance & Impact
- Why this matters
- Skills demonstrated

# Key Takeaway
- Main conclusion 1
- Main conclusion 2

# Key Figures
## Figure Section Title
### Figure Title

![](/_projects/Your Project Folder Name/image.png){:height="400px"}

<br>
Caption here.
<br>

# Python Code
## Script Name

```python
# code here
```
```

---

## Reference Projects

For real examples of the expected format and depth, refer to:
- `_projects/sonos-teardown/index.md` — most complete example with tables, figures, and full sections
- `_projects/Microgrid Optimization and Sizing using Pyomo & Gurobi/index.md` — example with extensive Python code blocks
- `_projects/demo-project/index.md` — example with MATLAB code
- `_projects/biomass-based-hydrogen/index.md` — example of a report/thesis-style project without code
