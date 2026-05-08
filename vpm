#!/bin/tcsh -f

# ============================================================
# VPM 3.2: Vim Package Manager
# ============================================================

set VIM_DIR   = "/disk/home/thomas/.vim"
set PACK_BASE = "$VIM_DIR/pack"
set LOG_FILE  = "$VIM_DIR/vpm.log"
set MANIFEST  = "$VIM_DIR/.vpm_manifest"

set nonomatch

# --- 1. Identify Valid Packages (Skip folders starting with .) ---
set plugins = ( "$PACK_BASE"/[^.]* )
if ( "$plugins" == "$PACK_BASE/[^.]*" ) then
    set p_total = 0
else
    set p_total = $#plugins
endif

# --- 2. Argument Parsing ---
if ( $#argv == 0 ) then
    set cmd = "load"
    set target = "all"
else
    set opt = $argv[1]
    switch ("$opt")
        case "-s":
        case "--status":
            set cmd = "status"; breaksw
        case "-a":
            set cmd = "load"; set target = $argv[2]; breaksw
        case "-d":
            set cmd = "del";  set target = $argv[2]; breaksw
        case "-c":
            set cmd = "clean"; breaksw
        case "--log":
            set cmd = "log"; breaksw
        default:
            echo "Unknown option: $opt"
            goto usage
            breaksw
    endsw
endif

# --- Dispatcher ---
if ( "$cmd" == "status" ) goto do_status
if ( "$cmd" == "log" )    goto do_log
if ( "$cmd" == "clean" )  goto do_clean
if ( "$cmd" == "del" )    goto do_delete
if ( "$cmd" == "load" )   goto do_load
exit 0

usage:
    echo "VPM 3.2 - Vim Package Manager"
    echo "Usage: vpm [options]"
    echo "  (no args)   Load all pending packages"
    echo "  -a <pkg>    Add a specific package from pack/"
    echo "  -d <pkg>    Delete a specific package"
    echo "  -s          Show current installation status"
    echo "  -c          Clean everything (Safe Mode)"
    echo "  --log       Show summarized status and file list"
    exit 1

# --- Logic: Status (Console Output) ---
do_status:
    set p_inst = 0
    if ( -f "$MANIFEST" ) then
        foreach p ( $plugins )
            set pn = `basename "$p"`
            grep -q "\[Plugin: $pn\]" "$MANIFEST" >& /dev/null
            if ( $status == 0 ) @ p_inst++
        end
    endif
    @ p_pend = $p_total - $p_inst
    echo "Packages -- $p_total total, $p_inst installed, $p_pend pending"
    foreach p ( $plugins )
        set pn = `basename "$p"`
        grep -q "\[Plugin: $pn\]" "$MANIFEST" >& /dev/null
        if ( $status == 0 ) then
            echo "  + $pn ( Installed )"
        else
            echo "  - $pn ( Pending )"
        endif
    end
    exit 0

# --- Logic: Log Viewer ---
do_log:
    echo "Log Path: $LOG_FILE"
    echo "----------------------------------------------------------"
    if ( -f "$LOG_FILE" ) cat "$LOG_FILE"
    exit 0

# --- Logic: Load ---
do_load:
    echo " >> Deploying packages..."
    foreach p ( $plugins )
        set pn = `basename "$p"`
        if ( "$target" != "all" && "$target" != "$pn" ) continue
        
        if ( -f "$MANIFEST" ) then
            grep -q "\[Plugin: $pn\]" "$MANIFEST" >& /dev/null
            if ( $status == 0 ) continue
        endif

        echo " >> Installing: $pn"
        echo "[Plugin: $pn]" >> "$MANIFEST"
        
        foreach sub ( `ls -F "$p" | grep '/' | sed 's|/||'` )
            if ( "$sub" == "README" || "$sub" == "LICENSE" ) continue
            foreach f ( `find "$p/$sub" -type f` )
                set rel = `echo "$f" | sed "s|^$p/||"`
                set tgt = "$VIM_DIR/$rel"
                mkdir -p `dirname "$tgt"`
                cp -f "$f" "$tgt"
                echo "FILE: $tgt" >> "$MANIFEST"
            end
        end
    end
    goto rebuild_log

# --- Logic: Delete ---
do_delete:
    if ( ! -f "$MANIFEST" ) exit 1
    echo " >> Removing package: $target"
    set files = `sed -n '/\[Plugin: '$target'\]/,/\[Plugin: /p' "$MANIFEST" | grep "^FILE: " | cut -d' ' -f2-`
    foreach f ( $files )
        if ( -f "$f" ) rm -f "$f"
    end
    sed -i '/\[Plugin: '$target'\]/,/\[Plugin: /{/\[Plugin: /!d;}' "$MANIFEST"
    sed -i '/\[Plugin: '$target'\]/d' "$MANIFEST"
    find "$VIM_DIR" -type d -empty -not -path "$PACK_BASE*" -delete >& /dev/null
    goto rebuild_log

# --- Logic: Clean ---
do_clean:
    echo " >> Cleaning all runtime files..."
    cd "$VIM_DIR"
    foreach item ( * .vpm_manifest )
        # Safety: Protect VPM script, its log, and the pack source
        if ( "$item" =~ "vpm*" || "$item" == "pack" ) continue
        if ( -e "$item" ) rm -rf "$item"
    end
    echo "Packages -- $p_total total, 0 installed, $p_total pending" > "$LOG_FILE"
    echo "Wiped at `date`" >> "$LOG_FILE"
    echo " >> All cleaned."
    exit 0

# --- Logic: Rebuild Log (Flat List Mode) ---
rebuild_log:
    set p_inst = 0
    if ( -f "$MANIFEST" ) then
        foreach p ( $plugins )
            set pn = `basename "$p"`
            grep -q "\[Plugin: $pn\]" "$MANIFEST" >& /dev/null
            if ( $status == 0 ) @ p_inst++
        end
    endif
    @ p_pend = $p_total - $p_inst

    # Log Header
    echo "Packages -- $p_total total, $p_inst installed, $p_pend pending" > "$LOG_FILE"
    foreach p ( $plugins )
        set pn = `basename "$p"`
        grep -q "\[Plugin: $pn\]" "$MANIFEST" >& /dev/null
        if ( $status == 0 ) then
            echo "  + $pn ( Installed )" >> "$LOG_FILE"
        else
            echo "  - $pn ( Pending )" >> "$LOG_FILE"
        endif
    end
    echo "" >> "$LOG_FILE"
    echo "--- File Details ---" >> "$LOG_FILE"

    # Log File List per Plugin
    foreach p ( $plugins )
        set pn = `basename "$p"`
        grep -q "\[Plugin: $pn\]" "$MANIFEST" >& /dev/null
        if ( $status != 0 ) continue

        echo "Installed package $pn ---" >> "$LOG_FILE"
        set pkg_files = `sed -n '/\[Plugin: '$pn'\]/,/\[Plugin: /p' "$MANIFEST" | grep "^FILE: " | cut -d' ' -f2-`
        foreach f ( $pkg_files )
            set display_path = `echo "$f" | sed "s|^$VIM_DIR/||" | sed 's|/| / |g'`
            echo "  + $display_path" >> "$LOG_FILE"
        end
        echo "" >> "$LOG_FILE"
    end
    
    if ( -d "$VIM_DIR/doc" ) gvim -u NONE -es "+helptags $VIM_DIR/doc" +q >>& /dev/null
    echo " >> Log updated."
    exit 0
