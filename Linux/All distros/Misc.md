# Miscellaneous

## Add git branch to Bash prompt

```bash
parse_git_branch() {
     git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}
```

## Add Docker compose status

```bash
parse_docker_compose() {
  if [[ "$( docker compose ps -q 2>/dev/null | wc -c )" -gt "0" ]]; then
    echo -e " RUNNING "
  fi
}
```

## Example Bash prompt
> todo: fix

```bash

Bparse_git_branch() {
  git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}

parse_docker_compose() {
  if [[ "$( docker compose ps -q 2>/dev/null | wc -c )" -gt "0" ]]; then
    echo -e "\e[44m\e[30m RUNNING \e[39m\e[49m$(parse_docker_onion)";
  fi
}

parse_docker_onion() {
  if [ -f .env ]; then
    export $(echo $(cat .env | sed 's/#.*//g'| xargs) | envsubst)
    if [ -v onion_name ]; then echo -e " \e[48;5;56m\e[30m V3 \e[39m\e[49m"; fi
  fi
}

check_free_space() {
  free="$( df -Ph . | awk 'NR == 2{print $5+0}' )"
  if [[ "$free" -gt "90" ]]; then
    echo -e "\e[41m \e[30m${free}%\e[39m \e[49m"
  elif [[ "$free" -gt "70" ]]; then
    echo -e "\e[43m \e[30m${free}%\e[39m \e[49m"
  else
    echo -e "\e[42m \e[30m${free}%\e[39m \e[49m"
  fi
}

export PS1="${BO} ${USER} ${DARKGRAY}@ ${GREEN}\\h ${NORMAL}${BC}${BO} ${YELLOW}\\w${NORMAL}${MAGENTA}\$(parse_git_branch)${NORMAL} ${BC} ${_ENV} ${BG_BLUE}${BLACK}\$(parse_docker_compose)${NORMAL}${BG_NORMAL}\\n${DARKGRAY}\\\$${NORMAL} "
```