# Tree-sitter Grammar for GDL (GNU Data Language)

> **Goal**: Create a tree-sitter grammar for GDL/IDL to enable AST-based code parsing for fusion and astronomy scientific code.

**Repository**: `iterorganization/tree-sitter-gdl` (forked to `simon-mcintosh/tree-sitter-gdl` for development)  
**PyPI Package**: `tree-sitter-gdl`  
**Grammar Name**: `gdl`

## Naming Decision

We use **GDL** (GNU Data Language) rather than IDL because:
1. GDL is the open-source implementation - no trademark concerns
2. `tree-sitter-idl` already exists for OMG IDL (CORBA Interface Definition Language)
3. PyPI package `tree-sitter-gdl` is available
4. GDL is 100% syntax-compatible with IDL - same `.pro` files

## Motivation

GDL/IDL (Interactive Data Language) is widely used in:
- **Fusion research**: Data analysis at TCV, ASDEX, DIII-D
- **Astronomy**: FITS file processing, image analysis
- **Earth science**: Satellite data processing

Currently, no tree-sitter grammar exists for GDL/IDL. This project would:
1. Enable AST-based code chunking in imas-codex
2. Benefit the broader scientific Python and tree-sitter communities
3. Be a contribution opportunity for the fusion community

## Discovering GDL/IDL Syntax

GDL/IDL terminology, syntax patterns, and sample files can be discovered from the EPFL facility using the **imas-codex MCP server**:

```python
# Using the python() REPL tool

# Find .pro files
print(ssh("fd -e pro /home --max-depth 4 2>/dev/null | head -20"))

# Search for specific constructs
print(ssh("rg -l 'mdsopen|mdsvalue' /home -g '*.pro' 2>/dev/null | head -10"))

# Read sample file
print(ssh("cat /home/bgeiger/write_namelist.pro"))

# Find common patterns
print(ssh("rg '^pro |^function ' /home -g '*.pro' 2>/dev/null | head -20"))
```

This enables iterative grammar development with real-world test cases from fusion research code.

## Feasibility Assessment

### IDL Syntax Complexity

| Feature | Complexity | Notes |
|---------|------------|-------|
| Procedures/Functions | Low | Simple `pro`/`function`...`end` blocks |
| Control structures | Medium | `if/then/else`, `for/do`, `case/of` |
| Operators | Medium | Word operators (`eq`, `ne`, `and`, `or`) |
| Array indexing | Medium | `[start:end:step]`, `[*]` |
| Keywords | Low | `/keyword`, `keyword=value` |
| Strings | Low | Single and double quoted |
| Comments | Low | `;` to end of line |
| Line continuation | Low | `$` at end of line |
| Object-oriented | Medium | `obj->method` syntax |

**Estimated complexity**: Similar to MATLAB grammar (~500-800 lines of JavaScript)

### Comparison to Existing Grammars

| Grammar | Lines | Complexity |
|---------|-------|------------|
| tree-sitter-matlab | ~800 | Similar syntax family |
| tree-sitter-fortran | ~1200 | More complex |
| tree-sitter-python | ~1500 | More complex |
| tree-sitter-bash | ~1000 | Comparable |

**Estimate**: GDL grammar would be ~600-900 lines.

## GDL/IDL Syntax Overview

### 1. Procedures and Functions

```idl
pro procedure_name, arg1, arg2, keyword=value
  ; procedure body
  print, 'Hello'
end

function function_name, arg1, arg2, keyword=value
  ; function body
  result = arg1 + arg2
  return, result
end
```

### 2. Control Structures

```idl
; If/then/else
if condition then statement
if condition then begin
  ; block
endif else begin
  ; else block
endelse

; For loop
for i = 0, n-1 do begin
  ; loop body
endfor

; Foreach
foreach element, array do begin
  ; element processing
endforeach

; While
while condition do begin
  ; loop body
endwhile

; Repeat/until
repeat begin
  ; loop body
endrep until condition

; Case/switch
case expression of
  value1: statement
  value2: begin
    ; block
  end
  else: default_statement
endcase
```

### 3. Operators

```idl
; Arithmetic
a + b, a - b, a * b, a / b, a ^ b, a mod b

; Comparison (word operators!)
a eq b    ; equal
a ne b    ; not equal
a lt b    ; less than
a gt b    ; greater than
a le b    ; less or equal
a ge b    ; greater or equal

; Logical
a and b, a or b, a xor b, not a

; Array
arr[0], arr[0:10], arr[*], arr[0:*:2]
arr # brr   ; matrix multiply
arr ## brr  ; transpose multiply
```

### 4. Keywords and Special Syntax

```idl
; Keyword shorthand
plot, x, y, /xlog        ; equivalent to xlog=1
plot, x, y, color=255

; Line continuation
result = long_function_name( $
  arg1, arg2, $
  arg3)

; Batch include
@startup_file

; Common blocks
common block_name, var1, var2, var3
```

### 5. Data Types

```idl
; Numbers
42, 42L, 42LL          ; integer, long, 64-bit
3.14, 3.14d0           ; float, double
1e10, 1d10             ; scientific notation
complex(1.0, 2.0)      ; complex

; Strings
'single quoted'
"double quoted"

; Arrays
[1, 2, 3, 4]
intarr(10), fltarr(10, 10)

; Structures
{tag1: value1, tag2: value2}
{name, tag1: value1}    ; named structure
```

## Implementation Plan

### Phase 1: Core Grammar (2-3 days)

**Week 1 Deliverables:**
- [ ] Repository setup with tree-sitter tooling
- [ ] Basic expressions (numbers, strings, identifiers)
- [ ] Operators (arithmetic, comparison, logical)
- [ ] Comments (`;` to end of line)
- [ ] Line continuation (`$`)

**Grammar file structure:**
```javascript
// grammar.js
module.exports = grammar({
  name: 'gdl',

  extras: $ => [
    /\s/,
    $.comment,
  ],

  rules: {
    source_file: $ => repeat($._statement),
    
    comment: $ => seq(';', /.*/),
    
    // ... rest of grammar
  }
});
```

### Phase 2: Statements and Control Flow (2-3 days)

**Deliverables:**
- [ ] Procedure definitions (`pro`...`end`)
- [ ] Function definitions (`function`...`end`)
- [ ] If/then/else (both inline and block forms)
- [ ] For loops
- [ ] While loops
- [ ] Case/switch statements

### Phase 3: Expressions and Advanced Features (2-3 days)

**Deliverables:**
- [ ] Array indexing (`[start:end:step]`)
- [ ] Function/procedure calls with keywords
- [ ] Object method calls (`obj->method`)
- [ ] Structure definitions and access
- [ ] Common blocks
- [ ] Batch includes (`@file`)

### Phase 4: Testing and Integration (2-3 days)

**Deliverables:**
- [ ] Corpus tests with real GDL/IDL code from EPFL
- [ ] Python bindings package (`tree-sitter-gdl`)
- [ ] Integration with tree-sitter-language-pack (PR)
- [ ] Integration with imas-codex

## Repository Structure

```
tree-sitter-gdl/
├── grammar.js           # Main grammar definition
├── src/
│   ├── parser.c         # Generated parser
│   ├── scanner.c        # Custom scanner (if needed)
│   └── tree_sitter/
│       └── parser.h
├── bindings/
│   ├── python/          # Python bindings
│   │   └── tree_sitter_gdl/
│   ├── rust/            # Rust bindings
│   └── node/            # Node.js bindings
├── queries/
│   ├── highlights.scm   # Syntax highlighting
│   ├── injections.scm   # Embedded languages
│   └── locals.scm       # Local definitions
├── test/
│   └── corpus/          # Test cases
├── package.json
├── Cargo.toml
├── pyproject.toml
├── LICENSE              # MIT
└── README.md
```

## Testing Strategy

### Corpus Tests

```text
================================================================================
Procedure definition
================================================================================

pro hello_world, name
  print, 'Hello, ' + name
end

--------------------------------------------------------------------------------

(source_file
  (procedure_definition
    name: (identifier)
    parameters: (parameter_list (identifier))
    body: (statement_list
      (call_expression
        function: (identifier)
        arguments: (argument_list
          (binary_expression
            left: (string)
            operator: (plus)
            right: (identifier)))))))
```

### Real-World Test Files

Use IDL files from EPFL for integration testing:
- `/home/bgeiger/write_namelist.pro`
- `/home/cordaro/script_analisi/idl/*.pro`

## Integration with imas-codex

### 1. Python Package Installation

```toml
# pyproject.toml (tree-sitter-gdl)
[project]
name = "tree-sitter-gdl"
version = "0.1.0"
dependencies = ["tree-sitter>=0.20"]

[build-system]
requires = ["tree-sitter"]
```

### 2. imas-codex Integration

```python
# imas_codex/code_examples/parsers.py

def get_parser(language: str) -> Parser:
    """Get tree-sitter parser for language."""
    if language == 'gdl':
        try:
            import tree_sitter_gdl
            return Parser(tree_sitter_gdl.language())
        except ImportError:
            logger.warning("tree-sitter-gdl not installed, using regex fallback")
            return None
    # ... other languages via tree-sitter-language-pack
```

### 3. Chunking Integration

```python
# Once grammar is ready, update discovery.yaml
tree_sitter_languages:
  - python
  - fortran
  - c
  - cpp
  - matlab
  - julia
  - bash
  - gdl  # Add when tree-sitter-gdl is ready
```

## Effort Estimate

| Phase | Effort | Dependencies |
|-------|--------|--------------|
| Phase 1: Core | 2-3 days | None |
| Phase 2: Statements | 2-3 days | Phase 1 |
| Phase 3: Advanced | 2-3 days | Phase 2 |
| Phase 4: Testing | 2-3 days | Phase 3 |
| **Total** | **8-12 days** | |

## Resources

### Tree-sitter Documentation
- [Tree-sitter Guide](https://tree-sitter.github.io/tree-sitter/creating-parsers)
- [Grammar DSL Reference](https://tree-sitter.github.io/tree-sitter/creating-parsers#the-grammar-dsl)

### GDL/IDL Documentation
- [IDL Language Reference](https://www.nv5geospatialsoftware.com/docs/idl_language.html)
- [GDL Source Code](https://github.com/gnudatalanguage/gdl) - Primary reference for grammar
- [GDL Documentation](https://gnudatalanguage.github.io/gdl-documentation/)

### Reference Grammars
- [tree-sitter-matlab](https://github.com/acristoffers/tree-sitter-matlab) - Most similar syntax
- [tree-sitter-fortran](https://github.com/stadelmanma/tree-sitter-fortran) - Scientific language
- [tree-sitter-python](https://github.com/tree-sitter/tree-sitter-python) - High-quality example

## Decision Points

### 1. Scope

**Option A**: Full IDL coverage (including OOP, all built-ins)
- Pros: Complete, community-ready
- Cons: More effort, may delay imas-codex integration

**Option B**: Core syntax only (what we need for chunking)
- Pros: Faster, focused on our use case
- Cons: Less useful to community

**Recommendation**: Start with Option B, extend to Option A over time.

### 2. Repository Location

**Chosen**: Separate repo (`iterorganization/tree-sitter-gdl`)
- Community contribution, reusable
- Published to PyPI as `tree-sitter-gdl`
- Development fork: `simon-mcintosh/tree-sitter-gdl`

### 3. Custom Scanner

IDL has some context-sensitive features (e.g., `end` keyword meaning depends on context). We may need a custom scanner in C.

**Likely needed for:**
- Context-sensitive `end` (endfor, endif, endelse, etc.)
- String interpolation edge cases

## Next Steps

1. **Repository created**: `github.com/iterorganization/tree-sitter-gdl`
2. **Development fork**: `github.com/simon-mcintosh/tree-sitter-gdl`
3. **Discover more samples**: Use MCP `python()` tool to SSH and explore EPFL `.pro` files
4. **Start Phase 1**: Core grammar implementation

## Appendix: Sample IDL Code

```idl
;+
; NAME:
;   ANALYZE_SHOT
;
; PURPOSE:
;   Analyze TCV shot data and generate plots
;
; CALLING SEQUENCE:
;   analyze_shot, shot_number, /verbose
;-
pro analyze_shot, shot, verbose=verbose, output_dir=output_dir

  ; Set defaults
  if n_elements(output_dir) eq 0 then output_dir = '~/analysis'
  
  ; Open MDSplus tree
  mdsopen, 'tcv_shot', shot
  
  ; Get time base
  time = mdsvalue('\results::r_axis:time')
  
  ; Get plasma parameters
  r_axis = mdsvalue('\results::r_axis')
  z_axis = mdsvalue('\results::z_axis')
  ip = mdsvalue('\magnetics::ip')
  
  ; Filter data
  good = where(time gt 0 and time lt 2.0, ngood)
  if ngood eq 0 then begin
    print, 'No valid data found'
    return
  endif
  
  ; Plot results
  if keyword_set(verbose) then begin
    window, 0
    plot, time[good], ip[good], $
      xtitle='Time [s]', ytitle='Ip [A]', $
      title='Shot ' + string(shot, format='(i6)')
  endif
  
  ; Save results
  save, time, r_axis, z_axis, ip, $
    filename=output_dir + '/shot_' + string(shot, format='(i6)') + '.sav'
  
  mdsclose
  
end
```

This sample demonstrates most IDL constructs we need to parse.
