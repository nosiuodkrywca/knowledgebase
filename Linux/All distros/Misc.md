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

BLACK="\\[$(tput setaf 0)\\]"
RED="\\[$(tput setaf 1)\\]"
GREEN="\\[$(tput setaf 2)\\]"
LIME_YELLOW=$(tput setaf 190)
YELLOW="\\[$(tput setaf 3)\\]"
POWDER_BLUE=$(tput setaf 153)
BLUE="\\[$(tput setaf 4)\\]"
MAGENTA="\\[$(tput setaf 5)\\]"
CYAN=$(tput setaf 6)
WHITE=$(tput setaf 7)
BRIGHT=$(tput bold)
NORMAL="\\[$(tput sgr0)\\]"
BLINK=$(tput blink)
REVERSE=$(tput smso)
UNDERLINE=$(tput smul)
LIGHTGRAY="\[\e[37m\]"
DARKGRAY="\[\e[90m\]"
CYAN="\[\e[36m\]"
LIGHTRED="\[\e[91m\]"

BG_RED="\[\e[41m\]"
BG_YELLOW="\[\e[43m\]"
BG_GREEN="\[\e[42m\]"
BG_NORMAL="\[\e[49m\]"
BG_BLUE="\[\e[44m\]"

BO="${LIGHTGRAY}[${NORMAL}"
BC="${LIGHTGRAY}]${NORMAL}"

parse_git_branch() {
  git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}

parse_docker_compose() {
  if [[ "$( docker compose ps -q 2>/dev/null | wc -c )" -gt "0" ]]; then
    echo -e " RUNNING "
  fi
}

if [ "$EUID" -ne 0 ]; then
  USER="${CYAN}\\u${NORMAL}"
else
  USER="${LIGHTRED}\\u${NORMAL}"
fi

if [[ $ENV == *"test"* ]]; then
  _ENV="${BG_YELLOW}${BLACK} TEST ${NORMAL}${BG_NORMAL}"
elif [[ $ENV == *"prod"* ]]; then
  _ENV="${BG_GREEN}${BLACK} PROD ${NORMAL}${BG_NORMAL}"
elif [[ $ENV == *"dev"* ]]; then
  _ENV="${BG_BLUE}${BLACK} DEV ${NORMAL}${BG_NORMAL}"
else
  _ENV="${BG_RED}${BLACK} UNKNOWN ${NORMAL}${BG_NORMAL}"
fi

export PS1="${BO} ${USER} ${DARKGRAY}@ ${GREEN}\\h ${NORMAL}${BC}${BO} ${YELLOW}\\w${NORMAL}${MAGENTA}\$(parse_git_branch)${NORMAL} ${BC} ${_ENV} ${BG_BLUE}${BLACK}\$(parse_docker_compose)${NORMAL}${BG_NORMAL}\\n${DARKGRAY}\\\$${NORMAL} "
```