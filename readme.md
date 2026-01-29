# Autodocs

## Setters (@set)

### `/home/hadean/Desktop/Bin/autodocs:4`
> Configure scan directory and output path

> from CLI arguments with field delimiters

```sh
SCAN_DIR="${1:-.}"
OUTPUT="${2:-readme.md}"
SCAN_DIR="${SCAN_DIR%/}"
SCAN_DIR=$(cd "$SCAN_DIR" && pwd)
TAB=$(printf '\t')
US=$(printf '\037')
```

### `/home/hadean/Desktop/Bin/autodocs:242`
> Initialize per-file state machine variables

> `_get_lang` sets `_gl` via result-variable pattern

> avoiding `$()` subshell fork, read back as `_pf_lang`

```sh
    _pf_path="$1"
    _pf_rel="$_pf_path"
    _get_lang "$_pf_path"
    _pf_lang="$_gl"
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

### `/home/hadean/Desktop/Bin/autodocs:41`
> Test whether a line contains any documentation tag

```sh
has_tag() {
    case "$1" in
        *'@set'*|*'@ass'*|*'@cal'*|*'@rai'*) return 0 ;;
        *) return 1 ;;
    esac
}
```

### `/home/hadean/Desktop/Bin/autodocs:49`
> Classify a tagged line into SET, ASS, CAL, or RAI

```sh
get_tag() {
    case "$1" in
        *'@set'*) _gt='SET' ;;
        *'@ass'*) _gt='ASS' ;;
        *'@cal'*) _gt='CAL' ;;
        *'@rai'*) _gt='RAI' ;;
    esac
}
```

### `/home/hadean/Desktop/Bin/autodocs:108`
> Detect comment style from a source line

> supporting hash, dslash, cblock, html, dquote, squote, ddash

```sh
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

### `/home/hadean/Desktop/Bin/autodocs:190`
> Map file extension to fenced code block language

> falling back to shebang detection for extensionless files


### `/home/hadean/Desktop/Bin/autodocs:565`
> Verify tagged files were discovered


### `/home/hadean/Desktop/Bin/autodocs:584`
> Verify extraction produced results


## Callers (@cal)

### `/home/hadean/Desktop/Bin/autodocs:59`
> Extract the subject line count from @tag:N syntax

> parsing leading digits after the colon into _gsc

```sh
get_subject_count() {
    _gsc=0
    case "$1" in
        *'@set:'[0-9]*) _gsc_r="${1#*@set:}" ;;
        *'@ass:'[0-9]*) _gsc_r="${1#*@ass:}" ;;
        *'@cal:'[0-9]*) _gsc_r="${1#*@cal:}" ;;
        *'@rai:'[0-9]*) _gsc_r="${1#*@rai:}" ;;
        *) return ;;
    esac
    _gsc=""
    while :; do
        case "$_gsc_r" in
            [0-9]*) _gsc="${_gsc}${_gsc_r%"${_gsc_r#?}"}"; _gsc_r="${_gsc_r#?}" ;;
            *) break ;;
        esac
    done
    [ -z "$_gsc" ] && _gsc=0 || :
}
```

### `/home/hadean/Desktop/Bin/autodocs:88`
> Remove @tag or @tag:N syntax from comment text

> delegates to _strip_tag_num for :N variants

```sh
strip_tags() {
    case "$1" in
        *'@set:'[0-9]*) _strip_tag_num "$1" "@set" ;;
        *'@set '*)      _st="${1%%@set *}${1#*@set }" ;;
        *'@set'*)       _st="${1%%@set*}${1#*@set}" ;;
        *'@ass:'[0-9]*) _strip_tag_num "$1" "@ass" ;;
        *'@ass '*)      _st="${1%%@ass *}${1#*@ass }" ;;
        *'@ass'*)       _st="${1%%@ass*}${1#*@ass}" ;;
        *'@cal:'[0-9]*) _strip_tag_num "$1" "@cal" ;;
        *'@cal '*)      _st="${1%%@cal *}${1#*@cal }" ;;
        *'@cal'*)       _st="${1%%@cal*}${1#*@cal}" ;;
        *'@rai:'[0-9]*) _strip_tag_num "$1" "@rai" ;;
        *'@rai '*)      _st="${1%%@rai *}${1#*@rai }" ;;
        *'@rai'*)       _st="${1%%@rai*}${1#*@rai}" ;;
        *)              _st="$1" ;;
    esac
}
```

### `/home/hadean/Desktop/Bin/autodocs:124`
> Strip comment delimiters and extract inner text

> for all styles including block continuations


### `/home/hadean/Desktop/Bin/autodocs:239`
> Walk one file as a line-by-line state machine

> extracting tagged comments into tab-delimited records


### `/home/hadean/Desktop/Bin/autodocs:260`
> Emit a documentation record or defer for subject capture

```sh
    _emit() {
        if [ -n "$_pf_tag" ] && [ -n "$_pf_text" ]; then
            _trim "$_pf_text"
            if [ -n "$_tr" ]; then
                if [ "$_pf_nsubj" -gt 0 ] 2>/dev/null; then
                    _pf_lang_f="$_pf_lang"
                    [ -z "$_pf_lang_f" ] && _pf_lang_f="-" || :
                    _pf_pending="${_pf_tag}${TAB}${_pf_rel}:${_pf_start}${TAB}${_tr}${TAB}${_pf_lang_f}"
                    _pf_cap_want="$_pf_nsubj"
                    _pf_subj=""
                else
                    printf '%s\t%s:%s\t%s\t%s\n' "$_pf_tag" "$_pf_rel" "$_pf_start" "$_tr" "$_pf_lang"
                fi
            fi
        fi
        _pf_in=""
        _pf_tag=""
        _pf_start=""
        _pf_text=""
        _pf_nsubj=0
    }
```

### `/home/hadean/Desktop/Bin/autodocs:283`
> Flush deferred record with captured subject lines

```sh
    _flush_pending() {
        if [ -n "$_pf_pending" ]; then
            printf '%s\t%s\n' "$_pf_pending" "$_pf_subj"
            _pf_pending=""
            _pf_subj=""
            _pf_capture=0
        fi
    }
```

### `/home/hadean/Desktop/Bin/autodocs:517`
> Render intermediate records into grouped markdown

> with blockquotes for text and fenced code blocks for subjects

```sh
render_markdown() {
    _rm_data="$1"
    printf '# Autodocs\n\n'

    _render_section() {
        _rs_prefix="$1"
        _rs_title="$2"
        _rs_label="$3"

        _rs_entries=$(printf '%s\n' "$_rm_data" | grep "^${_rs_prefix}${TAB}" 2>/dev/null || true)
        [ -z "$_rs_entries" ] && return || :

        printf '## %s (%s)\n\n' "$_rs_title" "$_rs_label"
        printf '%s\n' "$_rs_entries" | while IFS="$TAB" read -r _rs_tag _rs_loc _rs_text _rs_lang _rs_subj; do
            printf "### \`%s\`\n" "$_rs_loc"
            printf '%s' "$_rs_text" | tr '\037' '\n' | while IFS="" read -r _rs_line || [ -n "$_rs_line" ]; do
                _trim "$_rs_line"
                [ -n "$_tr" ] && printf '> %s\n\n' "$_tr" || :
            done
            if [ -n "$_rs_subj" ]; then
                if [ -n "$_rs_lang" ] && [ "$_rs_lang" != "-" ]; then
                    printf '```%s\n' "$_rs_lang"
                else
                    printf '```\n'
                fi
                printf '%s' "$_rs_subj" | tr '\037' '\n' | while IFS="" read -r _rs_line || [ -n "$_rs_line" ]; do
                    printf '%s\n' "$_rs_line"
                done
                printf '```\n'
            fi
            printf '\n'
        done
    }

    _render_section "SET" "Setters" "@set"
    _render_section "ASS" "Asserts" "@ass"
    _render_section "CAL" "Callers" "@cal"
    _render_section "RAI" "Raisers" "@rai"
}
```

### `/home/hadean/Desktop/Bin/autodocs:560`
> Discover files containing documentation tags

```sh
    _m_files=$(grep -rl -I \
        -e '@set' -e '@ass' -e '@cal' -e '@rai' \
        "$SCAN_DIR" 2>/dev/null) || true
```

### `/home/hadean/Desktop/Bin/autodocs:576`
> Process all discovered files into intermediate records

```sh
    _m_intermediate=$(
        printf '%s\n' "$_m_files" | while IFS="" read -r _m_fp; do
            case "$_m_fp" in */"$_m_out_base") continue ;; esac
            process_file "$_m_fp"
        done
    )
```

### `/home/hadean/Desktop/Bin/autodocs:593`
> Render documentation and write output file

```sh
    render_markdown "$_m_intermediate" > "$OUTPUT"
    printf 'autodocs: wrote %s\n' "$OUTPUT" >&2
```

### `/home/hadean/Desktop/Bin/autodocs:598`
> Entry point

```sh
main
```

## Raisers (@rai)

### `/home/hadean/Desktop/Bin/autodocs:567`
> Handle missing tagged files

> with empty output and stderr warning

```sh
        printf '# Autodocs\n\nNo tagged documentation found.\n' > "$OUTPUT"
        printf 'autodocs: no tags found under %s\n' "$SCAN_DIR" >&2
        return 0
```

### `/home/hadean/Desktop/Bin/autodocs:586`
> Handle extraction failure

> with empty output and stderr warning

```sh
        printf '# Autodocs\n\nNo tagged documentation found.\n' > "$OUTPUT"
        printf 'autodocs: tags found but no extractable docs under %s\n' "$SCAN_DIR" >&2
        return 0
```

