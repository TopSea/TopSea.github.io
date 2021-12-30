---
layout: post
title: Android adb 命令
date: 2021-12-19
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: adb.jpg # Add image post (optional)
tags: [Android, 游戏, Java] # add tag
---
Android adb 命令
---
以下源码可在 `android/platform/system/core/adb/adb.bash` 中找到    

~~~shell
_adb() {
    if ! check_type "$1" >/dev/null; then
        return
    fi
    if check_type _init_completion >/dev/null; then
        _init_completion || return
    fi
    local where i cur serial
    COMPREPLY=()
    serial="${ANDROID_SERIAL:-none}"
    where=OPTIONS
    for ((i=1; i <= COMP_CWORD; i++)); do
        cur="${COMP_WORDS[i]}"
        case "${cur}" in
            -s)
                where=OPT_SERIAL
                ;;
            -p)
                where=OPT_PATH
                ;;
            -*)
                where=OPTIONS
                ;;
            *)
                if [[ $where == OPT_SERIAL ]]; then
                    where=OPT_SERIAL_ARG
                    serial=${cur}
                else
                    where=COMMAND
                    break
                fi
                ;;
        esac
    done
    if [[ $where == COMMAND && $i -ge $COMP_CWORD ]]; then
        where=OPTIONS
    fi
    OPTIONS="-d -e -s -p"
    COMMAND="devices connect disconnect push pull sync shell emu logcat lolcat forward jdwp install uninstall bugreport help version start-server kill-server get-state get-serialno status-window remount reboot reboot-bootloader root usb tcpip disable-verity"
    case $where in
        OPTIONS|OPT_SERIAL|OPT_PATH)
            COMPREPLY=( $(compgen -W "$OPTIONS $COMMAND" -- "$cur") )
            ;;
        OPT_SERIAL_ARG)
            local devices=$(command adb devices 2> /dev/null | grep -v "List of devices" | awk '{ print $1 }')
            COMPREPLY=( $(compgen -W "${devices}" -- ${cur}) )
            ;;
        COMMAND)
            if [[ $i -eq $COMP_CWORD ]]; then
                COMPREPLY=( $(compgen -W "$COMMAND" -- "$cur") )
            else
                i=$((i+1))
                case "${cur}" in
                    install)
                        _adb_cmd_install "$serial" $i
                        ;;
                    sideload)
                        _adb_cmd_sideload "$serial" $i
                        ;;
                    pull)
                        _adb_cmd_pull "$serial" $i
                        ;;
                    push)
                        _adb_cmd_push "$serial" $i
                        ;;
                    reboot)
                        if [[ $COMP_CWORD == $i ]]; then
                            args="bootloader recovery"
                            COMPREPLY=( $(compgen -W "${args}" -- "${COMP_WORDS[i]}") )
                        fi
                        ;;
                    shell)
                        _adb_cmd_shell "$serial" $i
                        ;;
                    uninstall)
                        _adb_cmd_uninstall "$serial" $i
                        ;;
                esac
            fi
            ;;
    esac
    return 0
}
~~~   
`COMMAND` 中包含的即是可接受的命令，也可以在终端或命令行直接 `adb` 或 `adb --help` 查看，这样可以看到
命令的描述和使用方法。    

`adb tcpip [port]: 设置监听端口`   
`adb devices: 列出连接的设备`   

adb shell
---
源码如下：
~~~shell
_adb_cmd_shell() {
    local serial IFS=$'\n' i cur
    local -a args
    serial=$1
    i=$2
    cur="${COMP_WORDS[i]}"
    if [ "$serial" != "none" ]; then
        args=(-s $serial)
    fi
    if [[ $i -eq $COMP_CWORD && ${cur:0:1} != "/" ]]; then
        paths=$(command adb ${args[@]} shell echo '$'PATH 2> /dev/null | tr -d '\r' | tr : '\n')
        COMMAND=$(command adb ${args[@]} shell ls $paths '2>' /dev/null | tr -d '\r' | {
            while read -r tmp; do
                command=${tmp##*/}
                printf '%s\n' "$command"
            done
        })
        COMPREPLY=( $(compgen -W "$COMMAND" -- "$cur") )
        return 0
    fi
    i=$((i+1))
    case "$cur" in
        ls)
            _adb_shell_file_command $serial $i "--color -A -C -F -H -L -R -S -Z -a -c -d -f -h -i -k -l -m -n -p -q -r -s -t -u -x -1"
            ;;
        cat)
            _adb_shell_file_command $serial $i "-h -e -t -u -v"
            ;;
        dumpsys)
            _adb_cmd_shell_dumpsys "$serial" $i
            ;;
        am)
            _adb_cmd_shell_am "$serial" $i
            ;;
        pm)
            _adb_cmd_shell_pm "$serial" $i
            ;;
        /*)
            _adb_util_list_files $serial "$cur"
            ;;
        *)
            COMPREPLY=( )
            ;;
    esac
    return 0
}
~~~   
`case` 中包含的就是可接受的命令。


`adb shell am` 的命令有很多，可以查看源码找到所有的命令。源码如下：   
~~~shell
_adb_cmd_shell_am() {
    local serial i cur
    local candidates
    unset IFS
    serial=$1
    i=$2
    if (( $i == $COMP_CWORD )) ; then
        cur="${COMP_WORDS[COMP_CWORD]}"
        candidates="broadcast clear-debug-app clear-watch-heap dumpheap force-stop get-config get-inactive hang idle-maintenance instrument kill kill-all monitor package-importance profile restart screen-compat send-trim-memory set-debug-app set-inactive set-watch-heap stack start startservice start-user stopservice stop-user suppress-resize-config-changes switch-user task to-app-uri to-intent-uri to-uri"
        COMPREPLY=( $(compgen -W "$candidates" -- "$cur") )
        return 0
    fi
    COMPREPLY=( )
    return 0
}
~~~    
`candidates` 即是可接受的命令。   


`adb shell pm` 的命令也不少，可以查看源码找到所有的命令。源码如下：   
~~~shell
_adb_cmd_shell_pm() {
    local serial i cur
    local candidates
    unset IFS
    serial=$1
    i=$2
    if (( $i == $COMP_CWORD )) ; then
        cur="${COMP_WORDS[COMP_CWORD]}"
        candidates="-l -lf -p clear create-user default-state disable"
        candidates+=" disable-until-used disable-user dump enable"
        candidates+=" get-app-link get-install-location get-max-users"
        candidates+=" get-max-running-users grant hide install"
        candidates+=" install-abandon install-commit install-create"
        candidates+=" install-write list move-package"
        candidates+=" move-primary-storage path remove-user"
        candidates+=" reset-permissions revoke set-app-link"
        candidates+=" set-installer set-install-location"
        candidates+=" set-permission-enforced trim-caches unhide"
        candidates+=" uninstall"
        COMPREPLY=( $(compgen -W "$candidates" -- "$cur") )
        return 0
    fi
    if (( $i + 1 == $COMP_CWORD )) && [[ "${COMP_WORDS[COMP_CWORD -1]}" == "list" ]]  ; then
        cur="${COMP_WORDS[COMP_CWORD]}"
        candidates="packages permission-groups permissions instrumentation features libraries users"
        COMPREPLY=( $(compgen -W "$candidates" -- "$cur") )
        return 0
    fi
    COMPREPLY=( )
    return 0
}
~~~    
`candidates` 即是可接受的命令。


