# pandoc

Using pandoc to generate documents from markdown including plantuml diagrams.

## dependencies

- [pandoc](https://pandoc.org/) last known version which works 3.1.13
- [pandoc lua-filters](https://github.com/pandoc/lua-filters?tab=readme-ov-file)
- [plantuml](https://plantuml.com/)
  - java as required by plantuml.jar
 
## installation (linux)

pandoc: download tar.gz and extract into ~/.local/
pandoc-lua-filters: extract distribution file in pandoc data-dir.
plantuml.jar: download and place in appropriate location.
java: use package manager.

plantuml might need other depencies like dot and grapviz. Check the versions.

## generating a doc file from various markdown files containing also plantuml diagrams

The markdownfiles:
- 01_Introduction.md
- 02_Product views.md
- 03_Software_requirements_and_specifications.md
- 04_Verification.md

The reference docx file used for style:
 - style.docx

```bash
$ pandoc ./01_Introduction.md ./02_Product views.md ./03_Software_requirements_and_specifications.md ./04_Verification.md -f markdown -t docx --reference-doc=./style.docx --toc --embed-resources --standalone --lua-filter=~/.local/share/pandoc/filters/diagram-generator.lua --metadata=plantumlPath:"<path-to>/plantuml.jar" -o output.docx
```

The pandoc documentation states the `--lua-filter=..` looks into the `--data-dir` (pandoc -v). But seems not working, therefore use full path.
