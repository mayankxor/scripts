#!/usr/bin/env bash

# RULES:
# Syntax:
#   key: value
#   value must NOT contain ': '
#   key for base64 encoded data must be prefixed with base64_
#

copy(){
  if [[ -n "$WAYLAND_DISPLAY" ]]; then
    printf '%s' "$1" | sed 's/[[:space:]]*$//' | wl-copy
    ( sleep 45 && echo -n "" | wl-copy ) &
  else
    printf '%s' "$1" | sed 's/[[:space:]]*$//' | xclip -selection clipboard
    ( sleep 45 && echo -n "" | xclip -selection clipboard ) &
  fi
}

choice="$(ls ~/.password-store | sed 's/\.gpg$//'| dmenu)"
if [[ -z "$choice" ]]; then
  exit 1
fi
key="$(pass $choice |awk '
BEGIN { seen_pass=0 }
{
  if (index($0, ": ") > 0) {
    split($0, a, ": ")
    print a[1]
  }
  else if (index($0, ":") > 0) {
    print "totp"
  }
  else {
    if (!seen_pass) {
      print "pass"
      seen_pass=1
    } else {
      print "totp"
    }
  }
}
'| dmenu)"
case "$key" in
  pass)
    response="$(pass --clip "$choice")"
    notify-send "${response%%.*}" "${response#*. }"
    ;;
  totp)
    response="$(pass otp --clip "$choice")"
    notify-send "${response%%.*}" "${response#*. }"
    ;;
  base64_*)
    response="$(pass "$choice")"
    file="$(mktemp)"
    printf '%s' "$response" | command grep "$key" | sed "s/"$key": "// | sed 's/[[:space:]]*$//' | base64 -d > "$file"
    notify-send "Wrote secret to $file" "File will be deleted in 45 seconds"
    ( sleep 45 && rm "$file" ) &
    ;;
  *)
    copy "$(pass show "$choice" | awk -F': ' -v k="$key" '$1==k {print $2}')"
    notify-send "Copied $key for $choice" "Will be removed from clipboard after 45 seconds"
esac

