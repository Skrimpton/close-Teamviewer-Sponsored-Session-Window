# killSponsoredWindowTeamviewer
## Shell script that uses wmctrl to check for and close sponsored session window

```sh
# !/bin/sh

killMe=0
teamviewer_window_class="TeamViewer.TeamViewer"
hasWmctrl="$(which wmctrl)"
loopCheck=0
declare -a closeArr

#---------------------------------------------------------------------------------------------
# USAGE:
#  start ( teamviewer in subshell and disown )   &&   let window get drawn   &&   start deamon
#---------------------------------------------------------------------------------------------

# ( /opt/teamviewer/tv_bin/script/teamviewer & ) && sleep 2 && killSponsoredWindowTeamviewer

#---------------------------------------------------------------------------------------------
# DEPENDENCIES:
#  wmtrl
#  awk
#---------------------------------------------------------------------------------------------

#---------------------------------------------------------------------------------------------
# EXPLANATION:

help="""
———————————————————————————————————————————————————————————————————
HOMEMADE TeamViewer ADBLOCKER - Because they chose the dark side
———————————————————————————————————————————————————————————————————

  - Run in a loop every second (sleep 1) until '\$loopCheck' == 1

  - Check/update wmctrl list of all windows, with windowclass (the x-flag): '\$(wmctrl -lx)'

  - Check if open  window any has window class matching '\$teamviewer_window_class': '$teamviewer_window_class'
    (or whatever you have from wmtrcl -lx)
    and add all of them to a list called '\$closeArr'

  - Then check if teamviewer window has 'Sponsored session' in it's title

    ⋅ If it does then \$loopCheck = 1 and close all window id's in '\${closeArr[@]}' with '\$(wctrl -ic \$id)'

  - If there are no windows with windowclass = '$teamviewer_window_class' then \$loopCheck = 1, do nothing and exit script
"""
#---------------------------------------------------------------------------------------------

if [[ $@ = "-h" || $@ = "--help" ]]
then
  echo "$help"
else
  if [[ $hasWmctrl != "" ]]
  then
    while [[ $loopCheck == 0 ]]
    do

      IFS=$'\n' myarr=($(wmctrl -lx))
      for mywin in ${myarr[@]}
      do

        windowclass="$(echo "$mywin" | awk -F' ' '{print $3}' | xargs)"

        case "$windowclass" in

          $teamviewer_window_class)

            closeArr+=($(echo "$mywin" | awk -F' ' '{print $1}'  2>/dev/null | xargs))
            checkTitle="$(echo "$mywin" | awk -F\\${HOST} '{print $NF}'  2>/dev/null | xargs)"

            if [[ $checkTitle = "Sponsored session" ]]
            then
              killMe=1
            fi
        ;;
        esac
      done

      if [[ ${#closeArr[@]} = 0 ]]
      then

        loopCheck=1

      elif [[ $killMe == 1 ]]
      then

        loopCheck=1

      else
        sleep 1
        closeArr=()
      fi

    done

    for id in ${closeArr[@]}
    do
      wmctrl -ic $id
    done

  fi
fi
```
