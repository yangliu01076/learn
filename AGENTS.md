# AGENTS.md

## Repo type

Personal Java backend study notes. **No build system, no tests, no lint, no codegen.** Content is Markdown (`.md`) and PlantUML diagrams (`.puml`) only — do not look for or create build files.

## Language

All notes are written in **Chinese**. Write new content in Chinese to match.

## Structure

Topic-organized directories, each holding standalone note files:
- `java/`, `spring/`, `mybatis/`, `dubbo/`, `netty/`, `sentinel/` — frameworks
- `mysql/`, `redis/`, `elastic/`, `database/` — storage
- `kafka/` — messaging
- `算法/`, `设计模式/`, `计算机基础/`, `工程思想/`, `代码质量/` — fundamentals
- `大语言模型/` — LLM resources

No package boundaries or app entrypoints — every file is an independent note.

## Git conventions

- Commit message is always: `好好学习，天天向上`
- No `.gitignore` — all files are tracked and committed.

## Editing notes

- Add new notes as `.md` files in the matching topic directory; create the directory if a new topic is needed.
- Diagrams use `.puml` (PlantUML) and live alongside the `.md` that references them.
- Preserve existing Chinese formatting and heading style within each file.
