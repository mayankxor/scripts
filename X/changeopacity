#!/usr/bin/env bash

set -euo pipefail

ARG="${1:-}"
[ -z "$ARG" ] && { echo "Usage: $0 [+N|-N|N]"; exit 1; }

# Active window
WIN_ID=$(xprop -root _NET_ACTIVE_WINDOW | awk '{print $5}')
[ "$WIN_ID" = "0x0" ] && exit 1

# Read opacity (fallback = full)
RAW=$(xprop -id "$WIN_ID" _NET_WM_WINDOW_OPACITY 2>/dev/null || true)
CURRENT=$(echo "$RAW" | grep -oE '[0-9]+$' || echo 4294967295)

MAX=4294967295

# Convert percentage step → raw delta
percent_to_raw() {
    echo $(( $1 * MAX / 100 ))
}

if [[ "$ARG" =~ ^[+-][0-9]+$ ]]; then
    STEP=${ARG}
    DELTA=$(percent_to_raw "${STEP#[-+]}")
    if [[ "$STEP" == +* ]]; then
        NEW=$(( CURRENT + DELTA ))
    else
        NEW=$(( CURRENT - DELTA ))
    fi
elif [[ "$ARG" =~ ^[0-9]+$ ]]; then
    NEW=$(percent_to_raw "$ARG")
else
    echo "Invalid input"
    exit 1
fi

# Clamp (10%–100%)
MIN=$(percent_to_raw 10)
[ "$NEW" -lt "$MIN" ] && NEW=$MIN
[ "$NEW" -gt "$MAX" ] && NEW=$MAX

# Apply
xprop -id "$WIN_ID" -f _NET_WM_WINDOW_OPACITY 32c \
      -set _NET_WM_WINDOW_OPACITY "$NEW"

# Debug output
PERCENT=$(( NEW * 100 / MAX ))
echo "Opacity: ${PERCENT}%"
notify-send -h int:value:$PERCENT "Opacity" -r 9999 -t 500
