# Autodocs

## Setters (@set)

### `/home/user/Desktop/Bin/autodocs:4`
> Configure scan directory and output path
> from CLI arguments with field delimiters
```
SCAN_DIR="${1:-.}"
OUTPUT="${2:-readme.md}"
SCAN_DIR="${SCAN_DIR%/}"
SCAN_DIR=$(cd "$SCAN_DIR" && pwd)
TAB=$(printf '\t')
US=$(printf '\037')
```

### `/home/user/Desktop/Bin/autodocs:192`
> Initialize file path and state machine variables
> for tracking comment blocks and subject capture
```
    _pf_path="$1"
    _pf_rel="$_pf_path"
    _pf_ln=0
    _pf_in=""
    _pf_tag=""
    _pf_start=""
    _pf_text=""
    _pf_nsubj=0
    _pf_cap_want=0
    _pf_capture=0
    _pf_subj=""
    _pf_pending=""
```

## Asserts (@ass)

### `/home/user/Desktop/Bin/autodocs:506`
> Verify tagged files were discovered

### `/home/user/Desktop/Bin/autodocs:525`
> Verify extraction produced results

## Callers (@cal)

### `/home/user/Desktop/Bin/autodocs:41`
> Test whether a line contains any documentation tag
```
has_tag() {
    case "$1" in
        *'@set'*|*'@ass'*|*'@cal'*|*'@rai'*) return 0 ;;
        *) return 1 ;;
    esac
}
```

### `/home/user/Desktop/Bin/autodocs:49`
> Classify a tagged line into SET, ASS, CAL, or RAI
```
get_tag() {
    case "$1" in
        *'@set'*) _gt='SET' ;;
        *'@ass'*) _gt='ASS' ;;
        *'@cal'*) _gt='CAL' ;;
        *'@rai'*) _gt='RAI' ;;
    esac
}
```

### `/home/user/Desktop/Bin/autodocs:59`
> Extract the subject line count from @tag:N syntax
> parsing leading digits after the colon into _gsc

### `/home/user/Desktop/Bin/autodocs:88`
> Remove @tag or @tag:N syntax from comment text

### `/home/user/Desktop/Bin/autodocs:107`
> Detect comment style from a source line
> supporting hash, dslash, cblock, html, dquote, squote, ddash
```
detect_style() {
    _trim_lead "$1"
    case "$_tl" in
        '<!--'*)    _ds='html'   ;;
        '/*'*)      _ds='cblock' ;;
        '//'*)      _ds='dslash' ;;
        '#'*)       _ds='hash'   ;;
        '"""'*)     _ds='dquote' ;;
        "'''"*)     _ds='squote' ;;
        '--'*)      _ds='ddash'  ;;
        *)          _ds='none'   ;;
    esac
}
```

### `/home/user/Desktop/Bin/autodocs:123`
> Strip comment delimiters and extract inner text
> for all styles including block continuations

### `/home/user/Desktop/Bin/autodocs:189`
> Walk one file as a line-by-line state machine
> extracting tagged comments into tab-delimited records

### `/home/user/Desktop/Bin/autodocs:207`
> Emit a documentation record or defer for subject capture

### `/home/user/Desktop/Bin/autodocs:228`
> Flush deferred record with captured subject lines

### `/home/user/Desktop/Bin/autodocs:462`
> Render intermediate records into grouped markdown
> with blockquotes for text and fenced code blocks for subjects

### `/home/user/Desktop/Bin/autodocs:501`
> Discover files containing documentation tags
```
    _m_files=$(grep -rl -I \
        -e '@set' -e '@ass' -e '@cal' -e '@rai' \
        "$SCAN_DIR" 2>/dev/null) || true
```

### `/home/user/Desktop/Bin/autodocs:517`
> Process all discovered files into intermediate records
```
    _m_intermediate=$(
        printf '%s\n' "$_m_files" | while IFS="" read -r _m_fp; do
            case "$_m_fp" in */"$_m_out_base") continue ;; esac
            process_file "$_m_fp"
        done
    )
```

### `/home/user/Desktop/Bin/autodocs:534`
> Render documentation and write output file
```
    render_markdown "$_m_intermediate" > "$OUTPUT"
    printf 'autodocs: wrote %s\n' "$OUTPUT" >&2
```

### `/home/user/Desktop/Bin/autodocs:539`
> Entry point
```
main
```

## Raisers (@rai)

### `/home/user/Desktop/Bin/autodocs:508`
> Handle missing tagged files
> with empty output and stderr warning
```
        printf '# Autodocs\n\nNo tagged documentation found.\n' > "$OUTPUT"
        printf 'autodocs: no tags found under %s\n' "$SCAN_DIR" >&2
        return 0
```

### `/home/user/Desktop/Bin/autodocs:527`
> Handle extraction failure
> with empty output and stderr warning
```
        printf '# Autodocs\n\nNo tagged documentation found.\n' > "$OUTPUT"
        printf 'autodocs: tags found but no extractable docs under %s\n' "$SCAN_DIR" >&2
        return 0
```

