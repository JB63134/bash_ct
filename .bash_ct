#!/usr/bin/env bash
#-----------------------------------------------------------------------------------------------
# Command Trace aka ct                                                                    
# Purpose: Analyzes a Bash command, alias, builtin, keyword,  function, or binary
#          Trace command resolution, with shadowed command detection.        
#-----------------------------------------------------------------------------------------------
# Copyright (c) 2025 John Blair
# Licensed under the MIT License. See LICENSE file for details.        
#-----------------------------------------------------------------------------------------------
# BEHAVIORAL NOTES:
#
# • This script is intended to be SOURCED into an interactive Bash shell.
# • This script does NOT use `set -euo pipefail`.
# • PATH is optionally extended during execution to improve discovery of
#   administrative and user-level command locations. PATH is restored on exit.
#
# • Shell option `extglob` may be enabled temporarily and is restored afterward.
#
# • Kernel-level inspection may occur for PATH executables, including:
#     - symlink resolution
#     - ELF interpreter detection
#     - shebang parsing
#     - alternatives system detection
#
# • Bare command names only. Paths (e.g. /bin/ls) are rejected.
#-----------------------------------------------------------------------------------------------
# AUTO-EXTENDED DISCOVERY (JSON: auto_extended)
#
# When ct is invoked without -x/--extend and a command is not
# found in the user's PATH, ct will temporarily include known
# administrative paths (e.g. /usr/sbin, /sbin) to locate it.
#
# This behavior:
#   • does NOT modify PATH permanently
#   • does NOT execute the command
#   • is intended for interactive discovery
#
# When auto-extension occurs, JSON output includes:
#
#   "auto_extended": true
#
# Explicit -x/--extend disables auto-extension semantics and
# guarantees exhaustive analysis regardless of resolution.
#-----------------------------------------------------------------------------------------------
if [[ -z "${BASH_VERSION:-}" ]]; then
    printf "ct must be sourced or executed by bash.\n" >&2
    return 1
fi

__CT_version=4.2.10
__CT_needed_deps=(grep file cut head readlink readelf awk)
__CT_optional_deps=(tput)
__CT_extglob_was=0

complete -F _ct_completion ct

#------------------------------------------------------------------------
# Function: ct
# Purpose : User-facing entry point.
#
# Behavior:
# • Parses command-line options
# • Rejects path-based input
# • Dispatches to human-readable or JSON output
#
# Exit codes:
# • 0  → successful resolution
# • 1  → command not found or invalid input
#------------------------------------------------------------------------
ct() {
    local __CT_JSON_OUTPUT=0
    local __CT_EXTEND_PATH=0
    local __CT_AUTO_EXTEND=0

    trap '_ct_exit' RETURN

    while [[ $# -gt 0 ]]; do
        local arg="$1"

        case "$arg" in
            --help)
                _ct_usage
                return 0
                ;;
            --version)
                _ct_ver
                return 0
                ;;
            --json)
                __CT_JSON_OUTPUT=1
                shift
                ;;
            --extend)
                __CT_EXTEND_PATH=1
                shift
                ;;
            --json--extend|--extend--json)  
                __CT_JSON_OUTPUT=1
                __CT_EXTEND_PATH=1
                shift
                ;;
            -*)  # short flags, possibly combined like -jx
                local i
                for (( i=1; i<${#arg}; i++ )); do
                    local flag="${arg:i:1}"
                    case "$flag" in
                        j) __CT_JSON_OUTPUT=1 ;;
                        x) __CT_EXTEND_PATH=1 ;;
                        h)
                            _ct_usage
                            return 0
                            ;;
                        v)
                            _ct_ver
                            return 0
                            ;;
                        *)
                            printf "Unknown option: -%s\n" "$flag" >&2
                            return 1
                            ;;
                    esac
                done
                shift
                ;;
            *)
                break  # first non-option argument (command)
                ;;
        esac
    done

    # Auto-extend PATH if command not found but exists in admin paths
    if [[ -n "$1" && $__CT_EXTEND_PATH -eq 0 ]]; then
        if ! command -v -- "$1" &>/dev/null; then
            if _ct_found_in_admin_paths "$1"; then
                __CT_EXTEND_PATH=1
                __CT_AUTO_EXTEND=1
            fi
        fi
    fi

    # ---- Startup ----  
    _ct_startup "$__CT_EXTEND_PATH"
    
    # Validate input
    [[ -z "$1" ]] && _ct_usage && return 0
    
    # JSON mode rejects path input
    if [[ $__CT_JSON_OUTPUT == 1 ]] && [[ "$1" == */* ]]; then
        local path_type
        if [[ -d "$1" ]]; then
            path_type="directory"
        elif [[ -f "$1" ]]; then
            path_type="file"
        else 
            path_type="" 
        fi
        
        if [[ -n $path_type ]]; then
            printf '{ "error": true, "type": "invalid_input", "message": "%s is a %s. Command Trace only works with bare command names." }\n' "$1" "$path_type"
        else
            printf '{ "error": true, "type": "invalid_input", "message": "Unknown or invalid command: %s" }\n' "$1"
            return 1
        fi
        return 1 
    fi
    
    # Human-readable mode rejects path input with explanation
    if [[ "$1" == */* ]]; then
        if [[ -d "$1" ]]; then
            printf "├─ ${__CT_RED}%s is a directory${__CT_RESET}\n" "$1"
            printf "    ↳ Please provide the command name only, not a path.\n\n"
            return 1
        elif [[ -f "$1" ]]; then
            printf "├─ ${__CT_RED}%s is a file${__CT_RESET}\n" "$1"
            printf "    ↳ Command Trace only works with bare command names.\n\n"
            return 1
        fi 
    fi
    
    # Dispatch output
    if (( __CT_JSON_OUTPUT )); then
        _ct_json_output "$1" 
    else
        _ct_print_trace "$1"
    fi
}

#------------------------------------------------------------------------
# Function: _ct_resolve
# Purpose : Core command resolution engine.
#
# Inputs:
#   $1 → command name (bare token, no path)
#   $2 → nameref to associative array for resolution results (RES)
#   $3 → nameref to indexed array for PATH scan results (PATH_ENTRIES)
#
# Outputs (RES associative array):
#   command          → original command string
#   bash_type        → alias|function|keyword|builtin|path|notfound
#   shadowed_fs      → true if filesystem executables are unreachable
#
#   alias_*          → alias detection and definition
#   function_*       → function location and line
#   builtin_*        → builtin state (enabled/disabled)
#   keyword_found    → keyword match
#
#   kernel_*         → execution target details (PATH only)
#
# Notes:
# • Resolution precedence follows Bash rules, not filesystem order.
# • PATH scanning records *all* candidates, not just the winner.
#------------------------------------------------------------------------
# shellcheck disable=SC2154
_ct_resolve() {

__CT_POSIX_SPECIAL_BUILTINS=(
    :
    .
    break
    continue
    eval
    exec
    exit
    export
    readonly
    return
    set
    shift
    times
    trap
    unset
)

    local cmd="$1"
    local -n R="$2"      # associative result
    local -n P="$3"      # indexed PATH entries
    local alias_out
    
    # shellcheck disable=SC2034  
    R=()
    P=()

    RES[command]="$cmd"
    RES[bash_type]="notfound"
    RES[shadowed_fs]=false
    RES[function_shadowed]=false
    RES[builtin_shadowed]=false
    RES[keyword_shadowed_alias]=false
    RES[alias_shadowed]=false
    RES[posix_mode]=false
    
    # ------------------------------------------------------------
    # Edge cases
    # ------------------------------------------------------------
    case "$cmd" in
        '{'|'}'|'('|')'|'[['|']]'|']'|'.')
            RES[keyword_found]=true
            RES[bash_type]="keyword"
            ;;
    esac

    # ------------------------------------------------------------
    # Alias
    # ------------------------------------------------------------
    if alias "$cmd" &>/dev/null; then

        alias_out=$(alias -- "$cmd" 2>/dev/null) || return

        RES[alias_found]=true
        RES[alias_def]="$alias_out"
        RES[bash_type]="alias"
    else
        RES[alias_found]=false
    fi

# ------------------------------------------------------------
# POSIX special builtin (pre-function precedence)
# ------------------------------------------------------------
if (( __CT_POSIX_MODE )) && compgen -b | grep -Fxq -- "$cmd"; then
    if _ct_is_posix_special_builtin "$cmd"; then
        RES[builtin_found]=true
        RES[builtin_state]="enabled"
        RES[bash_type]="builtin"
        RES[bash_winner]="$cmd"
        RES[posix_special]=true
    fi
fi

if (( __CT_POSIX_MODE )); then
    RES[posix_mode]=true
fi
  
    # ------------------------------------------------------------
    # Function (skip if POSIX special builtin already resolved)
    # ------------------------------------------------------------
    if [[ ${RES[posix_special]} != true ]] && declare -F "$cmd" &>/dev/null; then
        R[function_found]=true

        local line file
        shopt -q extdebug || { shopt -s extdebug; local _ext=1; }
        read -r _ line file <<<"$(declare -F "$cmd")"
        [[ $_ext ]] && shopt -u extdebug

        R[function_file]="${file:-interactive shell}"
        R[function_line]="$line"

        # Mark shadowed if higher precedence exists (alias or keyword)
        if [[ ${R[bash_type]} =~ ^(alias|keyword|builtin)$ ]]; then
            R[function_shadowed]=true
        else
            R[bash_type]="function"
            R[function_shadowed]=false
        fi
    else
        R[function_found]=false
    fi

    # ------------------------------------------------------------
    # Keyword (higher precedence than builtin)
    # ------------------------------------------------------------
    if compgen -k | grep -Fxq -- "$cmd"; then
        RES[keyword_found]=true
        RES[bash_type]="keyword"
    else
        RES[keyword_found]=false
    fi

    if [[ ${RES[keyword_found]} == true && ${RES[alias_found]} == true ]]; then
        RES[keyword_shadowed_alias]=true
    fi
    
    # ------------------------------------------------------------
    # Builtin
    # ------------------------------------------------------------
    if compgen -b | grep -Fxq -- "$cmd"; then
        RES[builtin_found]=true

        # enabled/disabled state
        if enable -n | grep -Fxq -- "enable -n $cmd"; then
            RES[builtin_state]="disabled"
        else
            RES[builtin_state]="enabled"
        fi

        # shadowed only if enabled and something higher-precedence exists
if [[ ${RES[builtin_state]} == "enabled" && \
      ${RES[bash_type]} =~ ^(alias|function|keyword)$ && \
      ( __CT_POSIX_MODE -eq 0 || ! $(_ct_is_posix_special_builtin "$cmd") ) ]]; then
    RES[builtin_shadowed]=true
fi



        # builtin wins only if enabled and not shadowed, and nothing else resolved yet
        if [[ ${RES[builtin_state]} == "enabled" && ${RES[builtin_shadowed]} == false && ${RES[bash_type]} == "notfound" ]]; then
            RES[bash_type]="builtin"
            RES[bash_winner]="$cmd"
        fi
    else
        RES[builtin_found]=false
        RES[builtin_state]=null
        RES[builtin_shadowed]=false
    fi

# ------------------------------------------------------------
# POSIX special builtin precedence
# ------------------------------------------------------------
if (( __CT_POSIX_MODE )) && compgen -b | grep -Fxq -- "$cmd"; then
    if _ct_is_posix_special_builtin "$cmd"; then
        RES[builtin_found]=true
        RES[builtin_state]="enabled"
        RES[bash_type]="builtin"
        RES[bash_winner]="$cmd"

        # POSIX special builtin shadows function
        if [[ ${RES[function_found]} == true ]]; then
            RES[function_shadowed]=true
        fi
    fi
fi


    
    # Builtin only wins if no higher-precedence Bash entity already resolved
    if [[ ${RES[builtin_found]} == true &&
        ${RES[builtin_state]} == "enabled" &&
        ${RES[builtin_shadowed]} == false &&
        ${RES[bash_type]} == "notfound" ]]; then
        RES[bash_type]="builtin"
        RES[bash_winner]="$cmd"
    fi

    # ------------------------------------------------------------
    # PATH scan
    # PATH entry encoding format (PATH_ENTRIES):
    #   "dir|state|target|shadow"
    #
    #   dir     → PATH directory
    #   state   → notfound | file | symlink
    #   target  → resolved target (for symlink), empty otherwise
    #   shadow  → true if this entry is unreachable by bare invocation
    # ------------------------------------------------------------
    local found_path=""
    local path_winner_seen=0

    IFS=: read -ra _dirs <<<"$PATH"
    for d in "${_dirs[@]}"; do
        local p="$d/$cmd"
        
        # Skip if not executable
        if [[ ! -x "$p" || ! -f "$p" ]]; then
            P+=("$d|notfound||false")
            continue
        fi

        local state=file target="$p"
        if [[ -L "$p" ]]; then
            state=symlink
            target="$(readlink "$p")"
        fi

        local shadow=false
        # Mark as shadowed if a higher-priority Bash resolution exists
        if [[ ${RES[bash_type]} =~ ^(alias|function|builtin|keyword)$ ]]; then
            shadow=true
            RES[shadowed_fs]=true
        elif (( path_winner_seen )); then
            shadow=true
        else
            path_winner_seen=1
            # Only set bash_type=path if no higher-priority type is set
            [[ ${RES[bash_type]} == "notfound" ]] && RES[bash_type]="path"
            found_path="$p"
        fi

        P+=("$d|$state|$target|$shadow")
    done

    # ------------------------------------------------------------
    # Kernel execution details + alternatives detection
    # ------------------------------------------------------------
    if [[ ${RES[bash_type]} == "path" && -n $found_path ]]; then
        RES[kernel_path]="$found_path"
        RES[kernel_canonical]="$(readlink -f "$found_path")"
        RES[elf_interp]=""
        RES[shebang]=""

        # ELF interpreter (if ELF binary)
        if file "${RES[kernel_canonical]}" 2>/dev/null | grep -q ELF; then
            RES[elf_interp]="$(readelf -l "${RES[kernel_canonical]}" 2>/dev/null \
                          | awk '/interpreter/ {gsub(/[][]/, "", $NF); print $NF}')"
        fi

        # Shebang (if script)
        if head -1 "${RES[kernel_canonical]}" 2>/dev/null | grep -q '^#!'; then
            RES[shebang]="$(head -1 "${RES[kernel_canonical]}" | cut -c3-)"
        fi

        # Symlink chain (safe string for JSON)
        local chain=()
        local x="$found_path"
        while [[ -L "$x" ]]; do
            local y
            y="$(readlink "$x")"
            chain+=("$x -> $y")
            x="$y"
        done
        RES[kernel_chain]=$(printf '%s\n' "${chain[@]}")  # newline-delimited string

        # Alternatives detection
        RES[alternatives]=false
        RES[alternatives_link]=null
        RES[alternatives_target]=null

        x="$found_path"
        while [[ -L "$x" ]]; do
            if [[ "$x" == /etc/alternatives/* ]]; then
                RES[alternatives]=true
                RES[alternatives_link]="$x"
                RES[alternatives_target]="$(readlink -f "$x")"
                break
            fi
            x="$(readlink -f "$x")"
        done
    fi
  
    # special edge case
    if [[ $cmd == "." ]]; then
        RES[bash_type]="keyword"
        RES[keyword_found]=true
        RES[builtin_shadowed]=false
    fi
    
    : "${RES[bash_type]:=notfound}"
    return 0
}

#------------------------------------------------------------------------
# Function: _ct_is_posix_special_builtin
# Purpose: check if command is a posix special builtin
#------------------------------------------------------------------------
_ct_is_posix_special_builtin() {
    local b
    for b in "${__CT_POSIX_SPECIAL_BUILTINS[@]}"; do
        [[ $b == "$1" ]] && return 0
    done
    return 1
}

#------------------------------------------------------------------------
# Function: _ct_found_in_admin_paths
# Purpose: check if command exists outside of user path
#------------------------------------------------------------------------
_ct_found_in_admin_paths() {
    local cmd="$1"
    local d
    
    _CT_ADMIN_PATHS=(
    "/flatpak/bin"
    "/snap/bin"
    "/usr/local/sbin"
    "/usr/sbin"
    "/sbin"
    "/opt/sbin"
    )
    
    for d in "${_CT_ADMIN_PATHS[@]}"; do
        [[ -x "$d/$cmd" && -f "$d/$cmd" ]] && return 0
    done
    return 1
}

#------------------------------------------------------------------------
# Function: _ct_print_trace
# Purpose: Command Resolution Trace human display
#------------------------------------------------------------------------
# Human Output:
#
# • "Bash Resolution Target"
#     What Bash resolves the command to, based on shell semantics.
#
# • "Kernel Execution Target"
#     What the kernel actually executes (filesystem path, interpreter).
#
# These are intentionally separated to highlight shadowing and indirection.
#------------------------------------------------------------------------
_ct_print_trace() {
    local cmd="$1"

    declare -A RES
    declare -a PATH_ENTRIES

    _ct_resolve "$cmd" RES PATH_ENTRIES || return 1
    
    if [[ ${RES[bash_type]} == "notfound" ]]; then
          printf "${__CT_RED}Unknown or invalid command: ${__CT_CYAN}%s${__CT_RESET}\n" "$cmd"
          printf "  ↳ Check your spelling and try again\n\n"
         return 1
     fi

    printf "Command Trace of ${__CT_CYAN}%s${__CT_RESET}\n" "$cmd"

    if (( __CT_AUTO_EXTEND )); then 
        printf "\n"
        printf "Not in \$USER \$PATH\nFound in system \$PATH %s(auto-extended)%s:\n" "${__CT_YELLOW}" "${__CT_RESET}"
    fi
        # Dispatch output
    if (( __CT_POSIX_MODE )); then
        printf "\n"
        printf "Bash is in POSIX mode:\nPOSIX special builtins cannot be shadowed by functions;\nconflicting function names are disallowed.\n"
  #      printf "The shell will not execute a function whose name contains one or more slashes.\n" 
    fi
    printf "\n"

    # ------------------------------------------------------------
    # Keyword
    # ------------------------------------------------------------
    if [[ ${RES[keyword_found]} == true ]]; then
        printf "Keyword:  -  ${__CT_CYAN}%s → found${__CT_RESET}\n" "$cmd"
    else
        printf "Keyword:  -  not found\n"
    fi

    # ------------------------------------------------------------
    # Alias
    # ------------------------------------------------------------
    if [[ ${RES[alias_found]} == true ]]; then 
            printf "Alias:    -  ${__CT_CYAN}%s → found → %s${__CT_RESET}\n" "$cmd" "${RES[alias_def]}"   
    else
        printf "Alias:    -  not found\n"
    fi
    
    # ------------------------------------------------------------
    # Function
    # ------------------------------------------------------------
    if [[ ${RES[function_found]} == true ]]; then
        if [[ ${RES[function_shadowed]} == true ]]; then
            printf "Function: -  ${__CT_GREY}%s → found → Function %s (%s : line %s) [shadowed]${__CT_RESET}\n" \
            "$cmd" "$cmd" "${RES[function_file]}" "${RES[function_line]}"
        else
            printf "Function: -  ${__CT_CYAN}%s → found → Function %s (%s : line %s)${__CT_RESET}\n" \
            "$cmd" "$cmd" "${RES[function_file]}" "${RES[function_line]}"
        fi
    else
        printf "Function: -  not found\n"
    fi

    # ------------------------------------------------------------
    # Builtin
    # ------------------------------------------------------------
    if [[ ${RES[builtin_found]} == true ]]; then
        if [[ ${RES[builtin_state]} == "enabled" ]]; then
            if [[ ${RES[builtin_shadowed]} == true ]]; then
                printf "Builtin:  -  ${__CT_GREY}%s → found → %s [shadowed]${__CT_RESET}\n" \
                       "$cmd" "${RES[builtin_state]}"      
            else
                printf "Builtin:  -  ${__CT_CYAN}%s → found → %s${__CT_RESET}\n" \
                   "$cmd" "${RES[builtin_state]}"
            fi           
        else
            printf "Builtin:  -  ${__CT_GREY}%s → found → %s${__CT_RESET}\n" \
            "$cmd" "${RES[builtin_state]}"       
        fi                   
    else
        printf "Builtin:  -  not found\n"
    fi
    printf "\n"   
    
    # ------------------------------------------------------------
    # PATH scan rendering
    #
    # Shows every PATH entry in order, not just the winner.
    # Shadowed entries indicate commands that exist but cannot
    # be reached via bare invocation.
    # ------------------------------------------------------------
    printf "\$PATH in order:\n"

    for entry in "${PATH_ENTRIES[@]}"; do
        IFS='|' read -r dir state target shadow <<<"$entry"

        local usr_merged=""
        # Detect usr-merged directories by comparing inode to kernel_path
        if [[ -n "${RES[kernel_path]}" && -e "$dir/$cmd" ]]; then
            local inode1 inode2
            inode1=$(stat -c %i "$dir/$cmd" 2>/dev/null || echo "")
            inode2=$(stat -c %i "${RES[kernel_path]}" 2>/dev/null || echo "")
            [[ "$inode1" == "$inode2" && "$dir/$cmd" != "${RES[kernel_path]}" ]] && usr_merged=" [usr-merged]"
        fi

        case "$state" in
            notfound)
                printf "  ↳ %-24s - not found\n" "$dir"
                ;;
            file)
                if [[ $shadow == true ]]; then
                    printf "  ↳ %-24s - ${__CT_GREY}%s [shadowed]%s${__CT_RESET}\n" "$dir" "$cmd" "$usr_merged"
                else
                    printf "  ↳ %-24s - ${__CT_CYAN}%s found${__CT_RESET}%s\n" "$dir" "$cmd" "$usr_merged"
                fi
                ;;
            symlink)
                if [[ $shadow == true ]]; then
                    printf "  ↳ %-24s - ${__CT_GREY}%s (symlink) [shadowed]%s${__CT_RESET}\n" "$dir" "$cmd" "$usr_merged"
                else
                    printf "  ↳ %-24s - ${__CT_CYAN}%s${__CT_RESET} → ${__CT_YELLOW}%s${__CT_RESET}%s\n" \
                    "$dir" "$cmd" "$target" "$usr_merged"
                fi
                ;;
        esac    
    done

    printf "\n"

    # ------------------------------------------------------------
    # Bash resolution target
    # ------------------------------------------------------------
    printf "Bash Resolution Target:\n"

    case "${RES[bash_type]}" in
        keyword) winner_display="${__CT_CYAN}Keyword → ${RES[command]}${__CT_RESET}" ;;
        alias)    winner_display="${__CT_CYAN}Alias → ${RES[command]} → ${RES[alias_def]}${__CT_RESET}" ;;
        function) winner_display="${__CT_CYAN}Function → ${RES[command]} → ${RES[function_file]} : line ${RES[function_line]}${__CT_RESET}" ;;
        builtin)  winner_display="${__CT_CYAN}Builtin → ${RES[command]} → ${RES[builtin_state]}${__CT_RESET}" ;;
        path)     winner_display="${__CT_CYAN}Filesystem → ${RES[kernel_path]}${__CT_RESET}" ;;
        *)        printf "not found\n" ;;
    esac
    printf "  ↳ Resolved to: %s\n" "$winner_display"

    # Check for secondary entity if keyword won and a shadowed alias or builtin exists
    if [[ "${RES[bash_type]}" == "keyword" ]]; then
        if [[ ${RES[alias_found]} == true ]]; then
            printf "          ↳ And: ${__CT_CYAN}Alias → %s${__CT_RESET}\n" "${RES[alias_def]}"  
        fi
    
        if [[ ${RES[builtin_state]} == "enabled" ]]; then
            printf "          ↳ And: ${__CT_CYAN}Builtin → %s → %s${__CT_RESET}\n"  "${RES[command]}" "${RES[builtin_state]}"
        elif [[ ${RES[builtin_state]} == "disabled" ]]; then
            printf "          ↳ And: ${__CT_GREY}Builtin → %s → %s${__CT_RESET}\n"  "${RES[command]}" "${RES[builtin_state]}"
            printf "  ↳ Note: Builtin implementation is disabled; execution will fail.\n"

        fi    

        if [[ ${RES[alias_found]} == true || ${RES[function_found]} == true || ${RES[path_found]} == true ]]; then
            printf "  ↳ Note: Keyword ${__CT_CYAN}'%s'${__CT_RESET} always wins in its reserved context; other commands are ignored.\n" \
        "${RES[command]}"
        fi
    fi
    
    if [[ ${RES[function_found]} == true && ${RES[function_shadowed]} == true ]]; then
        printf "  ↳ Note: Function ${__CT_CYAN}'%s'${__CT_RESET} is unreachable by bare invocation.\n" "$cmd"
    fi
    
    if [[ ${RES[builtin_found]} == true && ${RES[builtin_shadowed]} == true ]]; then
        printf "  ↳ Note: Builtin ${__CT_CYAN}'%s'${__CT_RESET} is unreachable by bare invocation.\n" "$cmd"
    fi
    
    # Only print shadowed note for non-builtin
    if [[ ${RES[shadowed_fs]} == true ]]; then
        printf "  ↳ Note: Filesystem executables named ${__CT_CYAN}'%s'${__CT_RESET} are unreachable by bare invocation.\n" "$cmd"
    fi

    printf "\n"

    # ------------------------------------------------------------
    # Kernel execution target
    #
    # Only applies when bash_type == path.
    # ------------------------------------------------------------
    printf "Kernel Execution Target:\n"

    if [[ ${RES[bash_type]} == path ]]; then
        if [[ "${RES[kernel_path]}" != "${RES[kernel_canonical]}" ]]; then
            printf "  ↳ Executable → ${__CT_BPURP}%s${__CT_RESET}\n" "${RES[kernel_canonical]}"
        else
            printf "  ↳ Executable → ${__CT_CYAN}%s${__CT_RESET}\n" "${RES[kernel_canonical]}"
        fi

        [[ -n ${RES[elf_interp]} ]] && \
        printf "  ↳ ELF interpreter: ${__CT_CYAN}%s${__CT_RESET}\n" "${RES[elf_interp]}"

        [[ -n ${RES[shebang]} ]] && \
        printf "  ↳ Shebang: ${__CT_CYAN}%s${__CT_RESET}\n" "${RES[shebang]}"

        # symlink chain
        if [[ -n ${RES[kernel_chain]} ]]; then
            printf "  ↳ Symlink chain:\n"
            IFS=$'\n' read -r -d '' -a chain_lines <<<"${RES[kernel_chain]}" || true

            local last_index=$(( ${#chain_lines[@]} - 1 ))
            for i in "${!chain_lines[@]}"; do
                local line="${chain_lines[i]}"
                local src="${line%% -> *}"
                local tgt="${line##* -> }"
    
                if (( i == last_index )); then
                    # Final target (kernel executable)
                    printf "       → ${__CT_YELLOW}%s${__CT_RESET} -> ${__CT_BPURP}%s${__CT_RESET}\n" "$src" "$tgt"
                else
                    # intermediate symlink
                    printf "       → ${__CT_CYAN}%s${__CT_RESET} -> ${__CT_YELLOW}%s${__CT_RESET}\n" "$src" "$tgt"
                fi
            done
        fi  
    else
        printf "  ↳ NONE\n"
    fi

    printf "\n"
}

#------------------------------------------------------------------------
# Function: _ct_startup
# Purpose : One-time setup per ct invocation
#
# Ensures environment consistency and dependency availability.
#------------------------------------------------------------------------
_ct_startup() {
    local __CT_EXTEND_PATH="$1"
    # Save PATH if not already saved 
    if [[ -z "$__CT_OLD_PATH" ]]; then
        __CT_OLD_PATH="$PATH"
    fi
    
    _ct_bash_posix_mode
    
    _ct_bash_ver_check
    _ct_check_dependencies __CT_needed_deps __CT_optional_deps 1  || return 1 # 1 = exit if required
    (( __CT_EXTEND_PATH )) && _ct_add_admin_paths
    _ct_enable_extglob
    _ct_setup_colors # Setup tput or ansi
}

#------------------------------------------------------------------------
# Function: _ct_bash_posix_mode
# Purpose : Check if bash posix mode to apply posix rules if needed
#------------------------------------------------------------------------
_ct_bash_posix_mode() {
    __CT_POSIX_MODE=0
    
    shopt -qo posix && __CT_POSIX_MODE=1
    
}

#------------------------------------------------------------------------
# Function: _ct_exit
# Purpose : Cleanup handler
#
# Registered via trap to guarantee restoration of shell state.
#------------------------------------------------------------------------
_ct_exit() {
    _ct_restore_path
    __CT_RESET_extglob
    __CT_edgecase=0
    __CT_EXTEND_PATH=0
    __CT_AUTO_EXTEND=0
    __CT_POSIX_MODE=0
}

#------------------------------------------------------------------------
# Function: _ct_usage
# Purpose : Display usage information and examples for the `h` command analyzer.
#------------------------------------------------------------------------
_ct_usage() {
    cat <<'EOF'

    Usage:
      ct [option] command

    Command Trace (ct) analyzes how Bash resolves a command name 
    and what the kernel ultimately executes.

    The command must be provided as a bare command name.
    Paths such as /bin/ls or ./script are rejected.

    Options:
      -h, --help        Show this help text
      -v, --version     Show version and license information
      -j, --json        Emit JSON output suitable for scripting

EOF
}

#------------------------------------------------------------------------
# Function: _ct_ver
# Purpose : Display version information.
#------------------------------------------------------------------------
_ct_ver() {

    printf "\n     ct v%s  — Command Trace\n" "$__CT_version"
    cat <<'EOF'
    
     MIT License
     
     Copyright (c) 2025 John Blair

     Permission is hereby granted, free of charge, to any person obtaining a copy
     of this software and associated documentation files (the "Software"), to deal
     in the Software without restriction, including without limitation the rights
     to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
     copies of the Software, and to permit persons to whom the Software is
     furnished to do so, subject to the following conditions:

     The above copyright notice and this permission notice shall be included in all
     copies or substantial portions of the Software.

     THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
     IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
     FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
     AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
     LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
     OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
     SOFTWARE.

EOF
}

#------------------------------------------------------------------------
# Function: _ct_setup_colors         Colors (tput → ANSI → none)
# Purpose : make stuff pretty also safe for diff setups
#------------------------------------------------------------------------
_ct_setup_colors() {
    if command -v tput &>/dev/null && [[ $(tput colors 2>/dev/null) -ge 8 ]]; then
        __CT_RED=$(tput setaf 1)
        __CT_GREEN=$(tput setaf 2)
        __CT_CYAN=$(tput setaf 6)
        __CT_YELLOW=$(tput setaf 11)
        __CT_BPURP=$(tput setaf 13)
        __CT_GREY=$(tput setaf 245)
        __CT_RESET=$(tput sgr0)
        __CT_UNLINE=$(tput smul)
        __CT_STOPUNLINE=$(tput rmul)
    else
        __CT_RED='\033[31m'
        __CT_GREEN='\033[32m'
        __CT_CYAN='\033[36m'
        __CT_YELLOW='\033[93m'
        __CT_BPURP='\033[95m'
        __CT_GREY='\033[90m'
        __CT_RESET='\033[0m'
        __CT_UNLINE='\033[4m'
        __CT_STOPUNLINE='\033[24m'   
    fi

    # Ensure variables are always set
    : "${__CT_RED:=}"
    : "${__CT_GREEN:=}"
    : "${__CT_CYAN:=}"
    : "${__CT_YELLOW:=}"
    : "${__CT_BPURP:=}"
    : "${__CT_GREY:=}"
    : "${__CT_RESET:=}"
    : "${__CT_UNLINE:=}"
    : "${__CT_STOPUNLINE:=}"
}

#------------------------------------------------------------------------
# Function: _ct_add_admin_paths
# Purpose : Temporarily extend PATH for discovery
#
# Adds common user, admin, package-manager, and container paths.
#------------------------------------------------------------------------
_ct_add_admin_paths() {
    # Save PATH if not already saved 
    if [[ -z "$__CT_OLD_PATH" ]]; then
        __CT_OLD_PATH="$PATH"
    fi

    local admin_dirs=(
        "/flatpak/bin"
        "/snap/bin"
        "/usr/local/sbin" 
        "/usr/sbin"
        "/sbin"
        "/opt/sbin"
    )
        
    for dir in "${admin_dirs[@]}"; do
        [[ -d "$dir" && ":$PATH:" != *":$dir:"* ]] && PATH="$PATH:$dir"
    done
}

#------------------------------------------------------------------------
# Function: _ct_restore_path
# Purpose : Automatically restore PATH when needed
#------------------------------------------------------------------------
_ct_restore_path() {
    if [[ -n "$__CT_OLD_PATH" ]]; then
        PATH="$__CT_OLD_PATH"
    fi
}

#------------------------------------------------------------------------
# Function: _ct_completion
# Purpose : Enable tab completion for entering commands 
#------------------------------------------------------------------------
# Completion Notes:
#
# • Completion temporarily extends PATH to match ct resolution behavior.
# • Results include aliases, functions, builtins, and PATH commands.
#------------------------------------------------------------------------
_ct_completion() {
    local cur="${COMP_WORDS[COMP_CWORD]}"
    local commands
    
    local _old_path="$PATH"
    _ct_add_admin_paths
    
    # Collect possible completions: aliases, functions, builtins, and executables in PATH
    mapfile -t commands < <(compgen -A function -A alias -A builtin -A command -- "$cur")
    # Provide them to bash-completion
    COMPREPLY=("${commands[@]}")
    PATH="$_old_path"
}

#------------------------------------------------------------------------
# Function: _ct_enable_extglob / __CT_RESET_extglob
# Purpose : Safely manage extglob state
#
# extglob is needed for advanced pattern matching but must not
# permanently alter the user's shell environment.
#------------------------------------------------------------------------
_ct_enable_extglob() {
    if shopt -q extglob; then
        __CT_extglob_was=1
    else
        shopt -s extglob
    fi
}

#------------------------------------------------------------------------
# Function: __CT_RESET_extglob
# Purpose : Restore extglob to prior state, safley turn off if needed.
#------------------------------------------------------------------------
__CT_RESET_extglob() {
    (( !__CT_extglob_was )) && shopt -u extglob     # Restore extglob state if we enabled it
}

#------------------------------------------------------------------------
# Function: _ct_check_dependencies
# Purpose : Check core and optional dependencies, optionally exit on missing core deps
# Usage   : _ct_check_dependencies __Ct_needed_deps __Ct_optional_deps [EXIT_ON_MISSING] 0or1
#------------------------------------------------------------------------
_ct_check_dependencies() {
    local -n required="$1"
    local -n optional="$2"
    local exit_on_missing="${3:-1}"

    local missing_required=()
    local missing_optional=()

    for cmd in "${required[@]}"; do
        command -v "$cmd" >/dev/null 2>&1 || missing_required+=("$cmd")
    done

    for cmd in "${optional[@]}"; do
        command -v "$cmd" >/dev/null 2>&1 || missing_optional+=("$cmd")
    done

    if (( ${#missing_required[@]} )); then
        printf "%b\n" "${__CT_RED}Warning: Missing REQUIRED tools: ${missing_required[*]}${__CT_RESET}"
        if (( exit_on_missing )); then
            printf "%b\n" "${__CT_RED}Cannot continue analysis without core dependencies.${__CT_RESET}"
        fi
    fi
    return 0
}

#------------------------------------------------------------------------
# Function: _ct_bash_ver_check
# Purpose : Enforce minimum Bash version
#
# Bash 4.4+ is required for associative arrays and namerefs.
#------------------------------------------------------------------------
_ct_bash_ver_check() {
    local MIN_BASH_MAJOR=4
    local MIN_BASH_MINOR=4
    local CURRENT_BASH_MAJOR=${BASH_VERSINFO[0]}
    local CURRENT_BASH_MINOR=${BASH_VERSINFO[1]}

    if (( CURRENT_BASH_MAJOR < MIN_BASH_MAJOR )) || \
       (( CURRENT_BASH_MAJOR == MIN_BASH_MAJOR && CURRENT_BASH_MINOR < MIN_BASH_MINOR )); then
        printf "Error: Bash %d.%d or higher is required for 'ct'. You are using %d.%d.\n" \
            "$MIN_BASH_MAJOR" "$MIN_BASH_MINOR" "$CURRENT_BASH_MAJOR" "$CURRENT_BASH_MINOR" >&2
        printf "Please upgrade your Bash version to use this script.\n" >&2
        return 1
    fi
}

#------------------------------------------------------------------------
# Function: _ct_json_output
# Purpose : json output the name kind of says it all... 
#------------------------------------------------------------------------
# JSON OUTPUT NOTES:
#
# • JSON output is intended for scripting and inspection.
# • Fields may be added in future versions; consumers should not assume
#   strict schema stability across major versions.
#
# • Null values indicate "not applicable" or "not present".
# • Boolean fields are always true/false (never null).
#
# • bash_type possible values:
#     alias | function | keyword | builtin | path | notfound
#------------------------------------------------------------------------
_ct_json_output() {
    local cmd="$1"
    declare -A RES
    declare -a PATH_ENTRIES

    _ct_resolve "$cmd" RES PATH_ENTRIES || return 1

    if [[ ${RES[bash_type]} == "notfound" ]]; then
        printf '{ "error": true, "type": "invalid_input", "message": "Unknown or invalid command: %s" }\n' "$cmd"
        return 1
    fi

    # Precompute JSON values
    local bash_type_json="${RES[bash_type]:-null}"
    [[ -n "$bash_type_json" && "$bash_type_json" != "null" ]] && bash_type_json="\"$bash_type_json\""

    # auto_extended is true only if automatic PATH extension occurred
    if (( __CT_AUTO_EXTEND )) && [[ -z "${CT_MANUAL_PATH_EXTEND}" ]]; then
        local auto_extended_json=true
    else
        local auto_extended_json=false
    fi

    local alias_def_json="${RES[alias_def]:-null}"
    [[ -n "$alias_def_json" && "$alias_def_json" != "null" ]] && alias_def_json="\"$alias_def_json\""
    local function_file_json="${RES[function_file]:-null}"
    [[ -n "$function_file_json" && "$function_file_json" != "null" ]] && function_file_json="\"$function_file_json\""
    local function_line_json="${RES[function_line]:-null}"
    [[ -n "$function_line_json" && "$function_line_json" != "null" ]] && function_line_json="\"$function_line_json\""
    local builtin_state_json="${RES[builtin_state]:-null}"
    [[ -n "$builtin_state_json" && "$builtin_state_json" != "null" ]] && builtin_state_json="\"$builtin_state_json\""
    local kernel_path_json="${RES[kernel_path]:-null}"
    [[ -n "$kernel_path_json" && "$kernel_path_json" != "null" ]] && kernel_path_json="\"$kernel_path_json\""
    local kernel_canonical_json="${RES[kernel_canonical]:-null}"
    [[ -n "$kernel_canonical_json" && "$kernel_canonical_json" != "null" ]] && kernel_canonical_json="\"$kernel_canonical_json\""
    local elf_interp_json="${RES[elf_interp]:-null}"
    [[ -n "$elf_interp_json" && "$elf_interp_json" != "null" ]] && elf_interp_json="\"$elf_interp_json\""
    local shebang_json="${RES[shebang]:-null}"
    [[ -n "$shebang_json" && "$shebang_json" != "null" ]] && shebang_json="\"$shebang_json\""

    # Convert kernel_chain string to JSON array
    local chain_json=""
    if [[ -n "${RES[kernel_chain]}" ]]; then
        IFS=$'\n'
        for line in ${RES[kernel_chain]}; do
            [[ -n "$chain_json" ]] && chain_json+=", "
            chain_json+="\"$line\""
        done
        unset IFS
    fi

    # PATH JSON, one object per line
    path_json="["
    for entry in "${PATH_ENTRIES[@]}"; do
        IFS='|' read -r dir state target shadow <<<"$entry"
        [[ -z "$target" ]] && target_json=null || target_json="\"$target\""
        [[ $shadow == true ]] && shadow_json=true || shadow_json=false

        usr_merged=false

        case "$dir" in
            /bin|/sbin|/lib|/lib64)
                if [[ -L "$dir" ]]; then
                    # symlinked usr-merge
                    [[ $(readlink -f -- "$dir") == /usr/* ]] && usr_merged=true
                else
                    # inode-identical usr-merge (bind mount or hard layout)
                    local usr_dir="/usr$dir"
                    if [[ -d "$usr_dir" ]]; then
                        [[ $(stat -Lc '%d:%i' -- "$dir" 2>/dev/null) == \
                           $(stat -Lc '%d:%i' -- "$usr_dir" 2>/dev/null) ]] && usr_merged=true
                    fi
                fi
                ;;
        esac

        path_json+=$'\n    {'
        path_json+="\"dir\": \"$dir\","
        path_json+="\"state\": \"$state\","
        path_json+="\"target\": $target_json,"
        path_json+="\"shadowed\": $shadow_json,"
        path_json+="\"usr_merged\": $usr_merged"
        path_json+="},"
    done

    # Remove trailing comma and add closing bracket
    path_json="${path_json%,}"$'\n]'

    # Convert kernel_chain string to JSON array
    chain_json=""
    if [[ -n "${RES[kernel_chain]}" ]]; then
        local first=1
        while IFS= read -r line; do
            (( first )) || chain_json+=", "
            chain_json+="$(_ct_json_string_or_null "$line")"
            first=0
        done <<<"${RES[kernel_chain]}"
    fi

        # Output JSON uuoc
        cat <<EOF
{
  "command": "${RES[command]}",
  "posix_mode": "${RES[posix_mode]}",
  "auto_extended": $auto_extended_json,
  "bash_type": $bash_type_json,
  "shadowed_fs": ${RES[shadowed_fs]},
  "alias": {
    "found": ${RES[alias_found]},
    "definition": $alias_def_json
  },
  "keyword": {
    "found": ${RES[keyword_found]}
  },
  "function": {
    "found": ${RES[function_found]},
    "file": $function_file_json,
    "line": $function_line_json,
    "shadowed": ${RES[function_shadowed]}
  },
  "builtin": {
    "found": ${RES[builtin_found]},
    "state": $builtin_state_json,
    "shadowed": ${RES[builtin_shadowed]}
  },
  "path": $path_json,
  "kernel": {
    "path": $kernel_path_json,
    "canonical": $kernel_canonical_json,
    "elf_interpreter": $elf_interp_json,
    "shebang": $shebang_json,
    "chain": [$chain_json]
  }
}
EOF
}

#------------------------------------------------------------------------
# Function: _ct_json_string_or_null
# Purpose : get null right fool!
#------------------------------------------------------------------------
_ct_json_string_or_null() {
    if [[ -z $1 || $1 == "null" ]]; then
        printf 'null'
    else
        printf '"%s"' "$(printf '%s' "$1" | sed 's/\\/\\\\/g; s/"/\\"/g')"
    fi
}

#------------------------------------------------------------------------
# End of script
