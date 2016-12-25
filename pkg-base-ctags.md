# ctags

### Descriptions:
A C programming language indexing and/or cross-reference tool

### Changes on AnNyung:
* Exuberant CTags patch 적용
 * Added support for new "attached" and "detachable" keywords
 * Fixed parsing of comments after import statements and other tags
 * Fixed regular expressions for Ant so they won't span multiple tags
 * Fixed infinite loop with malformed Makefiles
 * Fixed Verilog parameter parsing
 * Fixed problem detecting function definitions using circumflexes for MS.NET managed types.
 * Fixed bug caused by use of strlen() where strings overlapped.
 * Enabled Large File System support
* PHP namespace 지원


### Sub packages:
* **ctags-etags** - Exuberant Ctags for emacs tag format