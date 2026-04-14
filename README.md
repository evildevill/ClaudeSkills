# ClaudeSkills

A collection of professional, production-ready Claude Code skills for AI-augmented development. Three domain-specific skills designed to enhance your coding workflow across document generation, frontend design, and full-stack development.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Skills: 3](https://img.shields.io/badge/Skills-3-blue)]()
[![Version: 1.0.0](https://img.shields.io/badge/Version-1.0.0-green)]()

## Overview

ClaudeSkills provides structured instruction packages that extend Claude's capabilities in three critical domains:

| Skill | Purpose | Use When |
|---|---|---|
| **docx** | Word document creation, editing, and analysis | Creating reports, memos, letters, or manipulating .docx files with formatting |
| **frontend-design** | Production-grade UI/UX component and page design | Building web components, landing pages, dashboards, or applications |
| **nextjs-16-stack** | Full-stack Next.js 16 development with TypeScript, MongoDB, and more | Building scalable, production-ready web applications |

Each skill includes:
- Detailed `SKILL.md` with complete instructions and best practices
- Structured YAML frontmatter with metadata
- MIT license with full terms in `LICENSE.txt`
- Compatible with Claude Code, Codex, Cursor, Gemini CLI, and more

## Quick Start

### Installation

#### For Claude Code
```bash
# Clone the repository
git clone https://github.com/evildevill/ClaudeSkills.git
cd ClaudeSkills

# Copy skills to Claude's skill directory
cp -r .github/skills/* ~/.claude/skills/
```

#### For Cursor
```bash
cp -r .github/skills/* ~/.cursor/skills/
```

#### For Codex CLI
```bash
cp -r .github/skills/* ~/.codex/skills/
```

#### For Gemini CLI
```bash
cp -r .github/skills/* ~/.gemini/skills/
```

### Verify Installation
```bash
# Claude Code
ls ~/.claude/skills/  # Should show: docx, frontend-design, nextjs-16-stack

# Cursor
ls ~/.cursor/skills/  # Should show: docx, frontend-design, nextjs-16-stack
```

## Skills Documentation

### 🗂️ DOCX — Word Document Generation & Editing

**Version:** 1.0.0 | **License:** MIT

Professional Word document (.docx) creation, editing, and analysis. Perfect for generating reports, memos, letters, proposals, and programmatically manipulating document content.

#### Quick Reference
```bash
# Read/analyze content
pandoc --track-changes=all document.docx -o output.md

# Create new document with JavaScript
npm install -g docx
node create-doc.js

# Convert .doc to .docx
python scripts/office/soffice.py --headless --convert-to docx document.doc

# Validate document structure
python scripts/office/validate.py document.docx
```

#### When to Use
- ✅ Create professional reports with formatting, tables of contents, headers/footers
- ✅ Edit existing .docx files programmatically
- ✅ Extract and manipulate document content with tracked changes
- ✅ Convert between .doc and .docx formats
- ✅ Generate memos, letters, contracts, and templates

#### When NOT to Use
- ❌ PDF generation (use PDF libraries instead)
- ❌ Google Docs operations
- ❌ General spreadsheet work (use Excel skills)

**[→ Full docx Skill Documentation](.github/skills/docx/SKILL.md)**

---

### 🎨 Frontend Design — Production-Grade UI/UX

**Version:** 1.0.0 | **License:** MIT

Create distinctive, production-grade frontend interfaces with exceptional design quality. Avoids generic AI-generated aesthetics and produces visually striking, memorable web components and pages.

#### Design Philosophy
Before building, understand the **purpose**, commit to a **bold aesthetic direction**, and execute with **precision**:
- **Typography**: Distinctive font choices (avoid generic Inter, Arial, Roboto)
- **Color & Theme**: Cohesive palettes with dominant colors and sharp accents
- **Motion**: CSS animations and micro-interactions that delight
- **Spatial Composition**: Unexpected layouts, asymmetry, depth, and breathing room
- **Visual Details**: Contextual effects, textures, gradients, and atmosphere

#### When to Use
- ✅ Build web components and reusable UI elements
- ✅ Design landing pages, marketing sites, and portfolios
- ✅ Create dashboards, admin panels, and data visualization interfaces
- ✅ Develop React/Vue/Svelte single-page applications
- ✅ Style HTML/CSS layouts with exceptional visual quality

#### When NOT to Use
- ❌ Simple CRUD interfaces (focus on substance, not style)
- ❌ Generic e-commerce checkouts (use template systems)
- ❌ Designs that need to match existing corporate branding exactly

**[→ Full Frontend Design Skill Documentation](.github/skills/frontend-design/SKILL.md)**

---

### ⚡ Next.js 16 Stack — Full-Stack Development

**Version:** 1.0.0 | **License:** MIT

Professional Next.js 16 development guidance with TypeScript, TailwindCSS, shadcn/ui, MongoDB, and Nodemailer. Build scalable, production-ready applications with best practices in system design, performance optimization, and architecture.

#### Core Technologies
- **Framework**: Next.js 16 with App Router and React Server Components
- **Language**: TypeScript for type safety
- **Styling**: TailwindCSS + shadcn/ui components
- **Database**: MongoDB with Mongoose ODM
- **Email**: Nodemailer integration
- **Validation**: Zod schemas
- **Auth**: Customizable authentication patterns

#### Project Structure
```
src/
├── app/                    # App Router (Next.js 16)
│   ├── (auth)/            # Route groups
│   ├── api/               # API routes
│   ├── layout.tsx         # Root layout
│   └── page.tsx           # Home page
├── components/            # Reusable components
│   ├── ui/               # shadcn/ui components
│   ├── forms/            # Form components
│   └── shared/           # Shared components
├── lib/                  # Utilities
│   ├── db/              # Database utilities
│   ├── email/           # Email setup
│   └── validations/     # Zod schemas
├── types/               # TypeScript definitions
├── hooks/               # Custom React hooks
└── actions/             # Server Actions
```

#### When to Use
- ✅ Building scalable full-stack web applications
- ✅ Next.js 16 projects requiring production architecture
- ✅ Full-stack development with TypeScript
- ✅ Implementing authentication, databases, and email services
- ✅ Performance optimization and caching strategies
- ✅ System design and scalability planning
- ✅ Real-world algorithms and data structures

#### When NOT to Use
- ❌ Static site generation (use simpler tools)
- ❌ Prototypes or throwaway experiments
- ❌ Projects not using Next.js or React

**[→ Full Next.js 16 Stack Skill Documentation](.github/skills/nextjs-16-stack/SKILL.md)**

---

## Repository Structure

```
ClaudeSkills/
├── .github/
│   └── skills/
│       ├── docx/
│       │   ├── SKILL.md          # Complete docx skill documentation
│       │   └── LICENSE.txt       # MIT License
│       ├── frontend-design/
│       │   ├── SKILL.md          # Complete frontend design documentation
│       │   └── LICENSE.txt       # MIT License
│       └── nextjs-16-stack/
│           ├── SKILL.md          # Complete Next.js 16 documentation
│           └── LICENSE.txt       # MIT License
├── README.md                      # This file
└── LICENSE                        # Repository license (MIT)
```

## How Skills Work

Each skill is a structured instruction package designed to enhance Claude's capabilities in a specific domain:

1. **SKILL.md** — The core instruction document with:
   - YAML frontmatter (metadata, version, tags, compatible tools)
   - Detailed guidance and best practices
   - Code examples and patterns
   - Decision frameworks and workflows
   - References and related resources

2. **LICENSE.txt** — Complete license terms (MIT)

3. **Metadata** — Version, author, tags, and compatible tools defined in frontmatter

### Supported Tools
All skills work with:
- Claude Code (native)
- Codex CLI
- Cursor
- Gemini CLI
- OpenCode
- Antigravity

## Usage Examples

### Example 1: Generate a Professional Report
```
Using the docx skill, create a quarterly business report with:
- Title page with company logo
- Executive summary
- Three sections with charts and tables
- Table of contents
- Tracked changes for review
```

### Example 2: Build a Modern Landing Page
```
Using the frontend-design skill, create a landing page for a SaaS product with:
- Bold, unique aesthetic (minimalist or maximalist, your choice)
- Exceptional typography
- Smooth scroll animations
- Mobile-responsive design
- Call-to-action sections
```

### Example 3: Scaffold a Full-Stack Application
```
Using the nextjs-16-stack skill, build a user authentication system with:
- MongoDB database schema
- API routes for login/register
- TypeScript type safety
- Zod validation schemas
- Email verification via Nodemailer
- shadcn/ui components
```

## Metadata

Each skill includes standardized metadata in YAML frontmatter:

```yaml
---
name: skill-name
description: Clear, actionable description
version: 1.0.0
author: claude-skills-collective
license: MIT
tags: [tag1, tag2, tag3]
compatible_tools: [claude-code, codex-cli, cursor, gemini-cli, opencode, antigravity]
---
```

## Contributing

We welcome contributions! To add new skills or improve existing ones:

1. **Fork** this repository
2. **Create** a feature branch (`git checkout -b feature/new-skill`)
3. **Structure** your skill following the [skill template](.github/skills/)
4. **Test** with Claude Code, Cursor, or other compatible tools
5. **Document** thoroughly in SKILL.md
6. **Submit** a pull request with a clear description

### Skill Requirements
- Complete YAML frontmatter with all metadata fields
- Comprehensive SKILL.md with examples and best practices
- MIT LICENSE.txt file
- Clear, actionable descriptions
- Tested with at least one supported tool
- No external dependencies (or clearly documented)

## License

All skills and documentation are licensed under the **MIT License**.

See individual [LICENSE.txt](.github/skills/docx/LICENSE.txt) files in each skill directory for complete terms.

```
MIT License

Copyright (c) 2026 Waseem Akram (evildevill)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```

## Related Projects

- **[alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills)** — Comprehensive library of 230+ production-ready Claude Code skills
- **[Claude Code](https://claude.ai/)** — AI-powered coding assistant
- **[Cursor](https://cursor.sh/)** — AI code editor
- **[Gemini CLI](https://github.com/google/gemini)** — Google's AI coding tool

## Support & Documentation

- **Claude Code Docs**: https://claude.ai/
- **Next.js Documentation**: https://nextjs.org/docs
- **TailwindCSS**: https://tailwindcss.com/docs
- **shadcn/ui**: https://shadcn-ui.com/
- **MongoDB**: https://docs.mongodb.com/

## Changelog

### Version 1.0.0 (April 2026)
- ✅ Initial release with 3 production-ready skills
- ✅ DOCX skill for Word document generation
- ✅ Frontend Design skill for UI/UX development
- ✅ Next.js 16 Stack skill for full-stack development
- ✅ Comprehensive documentation and examples

## FAQ

**Q: Can I use these skills with OpenAI Codex?**
A: Yes! All skills are compatible with Codex, Cursor, Gemini CLI, and other supported tools.

**Q: Do the scripts require external dependencies?**
A: Skills minimize external dependencies. Some features (like document validation) may require system tools like LibreOffice, but this is clearly documented in each skill.

**Q: Can I modify skills for my team?**
A: Yes, absolutely! The MIT license allows you to fork, modify, and distribute skills with proper attribution.

**Q: How do I report a bug or suggest improvements?**
A: Please open an issue on GitHub with:
- Skill name and version
- Detailed description of the issue
- Steps to reproduce (if applicable)
- Your tool and environment

**Q: How often are skills updated?**
A: Skills are maintained and updated based on community feedback, tool updates, and best practice changes. Check the changelog for details.

---

**Created with ❤️ by the ClaudeSkills community**

Questions? Issues? Ideas? [Open an issue](https://github.com/evildevill/ClaudeSkills/issues) or check the skill documentation for detailed guidance.
