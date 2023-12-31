# killSponsoredWindowTeamviewer
## Shell script that uses wmctrl to check for and close sponsored session window

```sh
#!/bin/sh

killMe=false
teamviewer_window_class="TeamViewer.TeamViewer"
hasWmctrl="$(which wmctrl)"
loopCheck=true
declare -a closeArr

#---------------------------------------------------------------------------------------------
# SUGGESTED USAGE:
#  start ( teamviewer in subshell and disown )   &&   let window get drawn   &&   start deamon
#---------------------------------------------------------------------------------------------

#  get the path from teamviewer's .desktop file
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

  - run in a loop every second (sleep 1) until '\$loopCheck' is false

  - output wmctrl list of all windows with windowclass '\$(wmctrl -lx)'

  - check if open  window any has window class matching '\$teamviewer_window_class': '$teamviewer_window_class'
    (or whatever you have from wmtrcl -lx)
    and add all of them to a list called '\$closeArr'

  - then check if teamviewer window has 'Sponsored session' in it's title
    ⋅ if it does then break the loop and close all windows \$id's in '\$closeArr[@]' with \$(wmtrcl -ic \$id)

  - if there are no windows with windowclass = '\$teamviewer_window_class' then break the loop, do nothing and exit script
"""
#---------------------------------------------------------------------------------------------

if [[ $@ = "-h" || $@ = "--help" ]]
then
  echo -e "$help"
else
  if [[ $hasWmctrl != "" ]]
  then

    while [[ $loopCheck == true ]]
    do

      IFS=$'\n' myarr=($(wmctrl -lx))

      for mywin in ${myarr[@]}
      do
        windowclass="$(echo "$mywin" | awk -F' ' '{print $3}' | xargs)"
        case "$windowclass" in

          $teamviewer_window_class)

            host_name="${HOSTNAME}"

            if [[ $host_name = "" ]]
            then
              host_name="${HOST}"
            fi


            if [[ $host_name = "" ]]
            then
              exit 1
            fi

            closeArr+=($(echo "$mywin" | awk -F' ' '{print $1}'  2>/dev/null | xargs))
            checkTitle="$(echo "$mywin" | awk -F\\${host_name} '{print $NF}'  2>/dev/null | xargs)"

            if [[ $checkTitle = "Sponsored session" ]]
            then
              killMe=true
            fi
        ;;
        esac
      done

      if [[ ${#closeArr[@]} = 0 ]]
      then

        loopCheck=false

      elif [[ $killMe == true ]]
      then

        loopCheck=false

      else
        sleep 1
        closeArr=()
      fi

    done

    for id in ${closeArr[@]}
    do
      wmctrl -ic $id &
    done

  fi
fi
```
