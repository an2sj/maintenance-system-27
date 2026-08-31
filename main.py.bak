# -*- coding: utf-8 -*-
"""
Ù†Ø¸Ø§Ù… Ø¥Ø¯Ø§Ø±Ø© Ø§Ù„ØµÙŠØ§Ù†Ø© ÙˆØ£ÙˆØ§Ù…Ø± Ø§Ù„Ø¹Ù…Ù„
Ù…Ø³ØªØ´ÙÙ‰ Ø§Ù„Ø£Ù…ÙŠØ± Ù…Ø­Ù…Ø¯ Ø¨Ù† Ù†Ø§ØµØ± | ØªØ¬Ù…Ø¹ Ø¬Ø§Ø²Ø§Ù† Ø§Ù„ØµØ­ÙŠ

ØªØ·Ø¨ÙŠÙ‚ FastAPI + SQLite Ø¬Ø§Ù‡Ø² Ù„Ù„Ù†Ø´Ø± Ø§Ù„Ø³Ø­Ø§Ø¨ÙŠ (Render / Replit):
  - ÙŠØ³ØªÙ…Ø¹ ØªÙ„Ù‚Ø§Ø¦ÙŠØ§Ù‹ Ø¹Ù„Ù‰ 0.0.0.0:$PORT (Ø§Ù„Ù…Ù†ÙØ° Ø§Ù„Ø¯ÙŠÙ†Ø§Ù…ÙŠÙƒÙŠ Ø§Ù„Ø°ÙŠ ØªØ­Ø¯Ø¯Ù‡ Ø§Ù„Ù…Ù†ØµØ©)
  - Ù‚Ø§Ø¹Ø¯Ø© Ø¨ÙŠØ§Ù†Ø§Øª SQLite Ù…Ø­Ù„ÙŠØ© ØªÙØ­ÙØ¸ Ø¨ÙŠÙ† Ø¹Ù…Ù„ÙŠØ§Øª Ø¥Ø¹Ø§Ø¯Ø© Ø§Ù„Ù†Ø´Ø± Ø¹Ø¨Ø± Ù‚Ø±Øµ Ø¯Ø§Ø¦Ù…
  - Ø­Ù…Ø§ÙŠØ© Ù„ÙˆØ­Ø© Ø§Ù„ØªØ­ÙƒÙ… Ø¨ÙƒÙ„Ù…Ø© Ù…Ø±ÙˆØ± (Ø§Ù„ØµÙØ­Ø§Øª Ø§Ù„Ø¹Ø§Ù…Ø© ØªØ¨Ù‚Ù‰ Ù…ÙØªÙˆØ­Ø©)
  - ÙˆØ§Ø¬Ù‡Ø© Ø«Ù†Ø§Ø¦ÙŠØ© Ø§Ù„Ù„ØºØ© (Ø¹Ø±Ø¨ÙŠ/Ø¥Ù†Ø¬Ù„ÙŠØ²ÙŠ)
  - Ù…Ø±ÙÙ‚Ø§Øª Ù…ÙŠØ¯ÙŠØ§ (ØµÙˆØ±/ÙÙŠØ¯ÙŠÙˆ) + Ø±Ù‚Ù… Ø·Ù„Ø¨ ÙØ±ÙŠØ¯ ØªÙ„Ù‚Ø§Ø¦ÙŠ
  - Ù…ØªØ¬Ø§ÙˆØ¨ Ø¨Ø§Ù„ÙƒØ§Ù…Ù„ Ù…Ø¹ Ø§Ù„Ø¢ÙŠØ¨Ø§Ø¯ ÙˆØ§Ù„Ø¬ÙˆØ§Ù„
"""

import hashlib
import io
import os
import re
import secrets
import shutil
from contextlib import contextmanager
from datetime import datetime, timedelta
from pathlib import Path
from urllib.parse import quote, unquote
from zoneinfo import ZoneInfo, ZoneInfoNotFoundError

import qrcode
import uvicorn
from fastapi import Depends, FastAPI, File, Form, HTTPException, Request, Response, UploadFile
from fastapi.responses import HTMLResponse, RedirectResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

# --------------------------------------------------------------------------- #
# Ø§Ù„Ø¥Ø¹Ø¯Ø§Ø¯Ø§Øª ÙˆØ§Ù„Ø«ÙˆØ§Ø¨Øª
# --------------------------------------------------------------------------- #
BASE_DIR = Path(__file__).resolve().parent
STATIC_DIR = BASE_DIR / "static"
TEMPLATES_DIR = BASE_DIR / "templates"

# Ù…Ø³Ø§Ø± Ù‚Ø§Ø¹Ø¯Ø© Ø§Ù„Ø¨ÙŠØ§Ù†Ø§Øª: ÙŠØ³Ù…Ø­ Ù„Ù„Ø¨ÙŠØ¦Ø© Ø§Ù„Ø³Ø­Ø§Ø¨ÙŠØ© Ø¨ØªÙˆØ¬ÙŠÙ‡Ù‡ Ø¥Ù„Ù‰ Ù‚Ø±Øµ Ø¯Ø§Ø¦Ù…
# Ø¹Ù„Ù‰ Render: Ù†Ø³ØªØ®Ø¯Ù… Ø§Ù„Ù‚Ø±Øµ Ø§Ù„Ù…Ø«Ø¨Øª Ø¹Ù„Ù‰ /data ØªÙ„Ù‚Ø§Ø¦ÙŠØ§Ù‹ Ø¥Ù† ØªÙˆÙØ±ØŒ ÙˆØ¥Ù„Ø§ Ù†Ø³ØªØ®Ø¯Ù… Ù…Ø¬Ù„Ø¯ Ø§Ù„Ù…Ø´Ø±ÙˆØ¹ Ù…Ø­Ù„ÙŠØ§Ù‹
_default_db = Path("/data/maintenance.db") if Path("/data").is_dir() else BASE_DIR / "data" / "maintenance.db"
DB_PATH = Path(os.environ.get("DB_PATH", str(_default_db)))

# Ø§Ù„Ù…Ù†Ø·Ù‚Ø© Ø§Ù„Ø²Ù…Ù†ÙŠØ© (Ø§ÙØªØ±Ø§Ø¶ÙŠØ©: Ø§Ù„Ø±ÙŠØ§Ø¶)
try:
    TZ = ZoneInfo(os.environ.get("APP_TIMEZONE", "Asia/Riyadh"))
except ZoneInfoNotFoundError:
    TZ = ZoneInfo("UTC")

# Ù‡ÙˆÙŠØ© Ø§Ù„Ù…Ø¤Ø³Ø³Ø© (Ø¨Ø¯ÙˆÙ† Ø´Ø±ÙƒØ© Ù…Ù‚Ø§ÙˆÙ„Ø§Øª)
HOSPITAL_NAME = os.environ.get("HOSPITAL_NAME", "Ù…Ø³ØªØ´ÙÙ‰ Ø§Ù„Ø£Ù…ÙŠØ± Ù…Ø­Ù…Ø¯ Ø¨Ù† Ù†Ø§ØµØ±")
ORG_RIGHT = os.environ.get("ORG_RIGHT", "ØªØ¬Ù…Ø¹ Ø¬Ø§Ø²Ø§Ù† Ø§Ù„ØµØ­ÙŠ")
ORG_LEFT = os.environ.get("ORG_LEFT", "")  # Ø£ÙØ²ÙŠÙ„Øª Ø´Ø±ÙƒØ© Ø§Ù„Ù…Ù‚Ø§ÙˆÙ„Ø§Øª Ù†Ù‡Ø§Ø¦ÙŠØ§Ù‹
DEPARTMENT_NAME = os.environ.get("DEPARTMENT_NAME", "Ø¥Ø¯Ø§Ø±Ø© Ø§Ù„ØµÙŠØ§Ù†Ø© ÙˆØ§Ù„Ø¹Ù…Ù„ÙŠØ§Øª")

# Ø´Ø¹Ø§Ø± ØªØ¬Ù…Ø¹ Ø¬Ø§Ø²Ø§Ù† Ø§Ù„ØµØ­ÙŠ (ÙŠÙØ³ØªØ®Ø¯Ù… ÙÙŠ Ø§Ù„Ù‡ÙŠØ¯Ø± ÙˆÙÙŠ Ù…Ø±ÙƒØ² Ø±Ù…Ø² QR)
LOGO_PATH = STATIC_DIR / "logo.jpg"
if not LOGO_PATH.exists():
    LOGO_PATH = STATIC_DIR / "logo.png"
    if not LOGO_PATH.exists():
        LOGO_PATH = None

# --------------------------------------------------------------------------- #
# Ù…Ù†Ø§Ø·Ù‚ Ø§Ù„ØªØ®Ø²ÙŠÙ† Ø§Ù„Ø¯Ø§Ø¦Ù… (Ø§Ù„Ù…ÙŠØ¯ÙŠØ§)
# --------------------------------------------------------------------------- #
# ØªÙØ®Ø²ÙŽÙ‘Ù† Ø§Ù„Ù…Ø±ÙÙ‚Ø§Øª Ø¹Ù„Ù‰ Ø§Ù„Ù‚Ø±Øµ Ø§Ù„Ø¯Ø§Ø¦Ù… (Ù…Ø¬Ù„Ø¯ Ø§Ù„Ø¨ÙŠØ§Ù†Ø§Øª) Ø­ØªÙ‰ Ù„Ø§ ØªÙÙÙ‚Ø¯ Ø¹Ù†Ø¯ Ø¥Ø¹Ø§Ø¯Ø© Ø§Ù„Ù†Ø´Ø±
UPLOAD_DIR = Path(os.environ.get("UPLOAD_DIR", DB_PATH.parent / "uploads"))
UPLOAD_DIR.mkdir(parents=True, exist_ok=True)
MEDIA_URL_PREFIX = "/media"
ALLOWED_MEDIA = {
    ".jpg": "image/jpeg", ".jpeg": "image/jpeg", ".png": "image/png",
    ".gif": "image/gif", ".webp": "image/webp",
    ".mp4": "video/mp4", ".mov": "video/quicktime", ".webm": "video/webm",
}
MAX_MEDIA = 5  # Ø§Ù„Ø­Ø¯ Ø§Ù„Ø£Ù‚ØµÙ‰ Ù„Ø¹Ø¯Ø¯ Ø§Ù„Ù…Ø±ÙÙ‚Ø§Øª ÙÙŠ Ø¨Ù„Ø§Øº ÙˆØ§Ø­Ø¯

# --------------------------------------------------------------------------- #
# Ø£Ù†ÙˆØ§Ø¹ Ø§Ù„ØµÙŠØ§Ù†Ø© ÙˆÙ†ÙˆØ¹ Ø§Ù„Ù…Ø´ÙƒÙ„Ø© Ø§Ù„Ø¯ÙŠÙ†Ø§Ù…ÙŠÙƒÙŠ
# --------------------------------------------------------------------------- #
MAINTENANCE_TYPES = ["ÙƒÙ‡Ø±Ø¨Ø§Ø¡", "ØªÙƒÙŠÙŠÙ", "Ø³Ø¨Ø§ÙƒØ©", "Ù…Ø¯Ù†ÙŠ", "Ù†Ø¬Ø§Ø±Ø©", "Ø£Ù…Ù† ÙˆØ³Ù„Ø§Ù…Ø©", "Ø£Ø®Ø±Ù‰"]

PROBLEM_TYPES = {
    "ÙƒÙ‡Ø±Ø¨Ø§Ø¡": ["Ø§Ù†Ù‚Ø·Ø§Ø¹ Ø§Ù„ØªÙŠØ§Ø±", "ØªÙ„Ù Ù…Ù†ÙØ°/Ù…Ù‚Ø¨Ø³", "Ù…Ø´ÙƒÙ„Ø© Ø¥Ø¶Ø§Ø¡Ø©", "Ù‚Ø§Ø·Ø¹ ÙƒÙ‡Ø±Ø¨Ø§Ø¦ÙŠ", "Ù…Ø±ÙˆØ­Ø©/Ø´ÙØ§Ø·", "Ø£Ø®Ø±Ù‰"],
    "ØªÙƒÙŠÙŠÙ": ["Ù„Ø§ ÙŠØ¨Ø±Ø¯", "ØªØ³Ø±ÙŠØ¨ Ù…Ø§Ø¡", "Ø¶Ø¬ÙŠØ¬", "Ù„Ø§ ÙŠØ¹Ù…Ù„", "Ø±ÙŠÙ…ÙˆØª ÙƒÙˆÙ†ØªØ±ÙˆÙ„", "Ø£Ø®Ø±Ù‰"],
    "Ø³Ø¨Ø§ÙƒØ©": ["ØªØ³Ø±ÙŠØ¨ Ù…Ø§Ø¡", "Ø§Ù†Ø³Ø¯Ø§Ø¯", "Ø­Ø±Ø§Ø±Ø©/Ù…Ø§Ø¡ Ø³Ø§Ø®Ù†", "ØµØ±Ù ØµØ­ÙŠ", "Ø®Ù„Ø§Ø·/ØµÙ†Ø§Ø¨ÙŠØ±", "Ø£Ø®Ø±Ù‰"],
    "Ù…Ø¯Ù†ÙŠ": ["ØªØ´Ù‚Ù‚/ØªØµØ¯Ø¹", "ØªØ³Ø±Ø¨ Ø³Ù‚Ù", "Ø¨Ù„Ø§Ø·/Ø³ÙŠØ±Ø§Ù…ÙŠÙƒ", "Ø¯Ù‡Ø§Ù†Ø§Øª", "Ø£Ø¨ÙˆØ§Ø¨/Ù†ÙˆØ§ÙØ°", "Ø£Ø®Ø±Ù‰"],
    "Ù†Ø¬Ø§Ø±Ø©": ["Ø®Ø²Ø§Ù†Ø©/Ø¯ÙˆÙ„Ø§Ø¨", "ÙƒØ³Ø± Ø£Ø«Ø§Ø«", "Ø·Ø§ÙˆÙ„Ø©/Ù…Ù†Ø¶Ø¯Ø©", "ØªØ´Ù„ÙŠØ­ Ø®Ø´Ø¨", "ÙƒØ±Ø³ÙŠ/Ù…Ù‚Ø¹Ø¯", "Ø£Ø®Ø±Ù‰"],
    "Ø£Ù…Ù† ÙˆØ³Ù„Ø§Ù…Ø©": ["Ø·ÙØ§ÙŠØ© Ø­Ø±ÙŠÙ‚", "Ù†Ø¸Ø§Ù… Ø¥Ù†Ø°Ø§Ø±", "Ù…Ø®Ø±Ø¬ Ø·ÙˆØ§Ø±Ø¦", "Ø¥Ø¶Ø§Ø¡Ø© Ø·ÙˆØ§Ø±Ø¦", "Ø¨ÙˆØ§Ø¨Ø©/Ø­Ø§Ø¬Ø²", "Ø£Ø®Ø±Ù‰"],
    "Ø£Ø®Ø±Ù‰": ["Ø£Ø®Ø±Ù‰"],
}

PRIORITIES = ["Ø¹Ø§Ø¯ÙŠØ©", "Ù…Ø³ØªØ¹Ø¬Ù„Ø©", "Ø·Ø§Ø±Ø¦Ø©"]

STATUS_META = {
    "pending": ("Ø¨Ø§Ù†ØªØ¸Ø§Ø± Ø§Ù„Ø§Ø¹ØªÙ…Ø§Ø¯", "warn"),
    "approved": ("Ù…Ø¹ØªÙ…Ø¯ - Ù‚ÙŠØ¯ Ø§Ù„ØªÙ†ÙÙŠØ°", "info"),
    "done": ("Ù…Ù†Ø¬Ø² ÙˆÙ…ØºÙ„Ù‚", "ok"),
    "rejected": ("Ù…Ø±ÙÙˆØ¶", "bad"),
}

MONTHS_AR = [
    "ÙŠÙ†Ø§ÙŠØ±", "ÙØ¨Ø±Ø§ÙŠØ±", "Ù…Ø§Ø±Ø³", "Ø£Ø¨Ø±ÙŠÙ„", "Ù…Ø§ÙŠÙˆ", "ÙŠÙˆÙ†ÙŠÙˆ",
    "ÙŠÙˆÙ„ÙŠÙˆ", "Ø£ØºØ³Ø·Ø³", "Ø³Ø¨ØªÙ…Ø¨Ø±", "Ø£ÙƒØªÙˆØ¨Ø±", "Ù†ÙˆÙÙ…Ø¨Ø±", "Ø¯ÙŠØ³Ù…Ø¨Ø±",
]
MONTHS_EN = [
    "January", "February", "March", "April", "May", "June",
    "July", "August", "September", "October", "November", "December",
]
WEEKDAYS_AR = ["Ø§Ù„Ø§Ø«Ù†ÙŠÙ†", "Ø§Ù„Ø«Ù„Ø§Ø«Ø§Ø¡", "Ø§Ù„Ø£Ø±Ø¨Ø¹Ø§Ø¡", "Ø§Ù„Ø®Ù…ÙŠØ³", "Ø§Ù„Ø¬Ù…Ø¹Ø©", "Ø§Ù„Ø³Ø¨Øª", "Ø§Ù„Ø£Ø­Ø¯"]

# --------------------------------------------------------------------------- #
# Ø§Ù„Ù…ÙˆØ§Ù‚Ø¹ / Ø§Ù„Ù…Ø¨Ø§Ù†ÙŠ
# --------------------------------------------------------------------------- #
PUBLIC_FACILITIES = [
    "Ø§Ù„Ù…ÙƒØªØ¨", "Ø§Ù„Ø§Ø³ØªÙ‚Ø¨Ø§Ù„", "Ø§Ù„Ø­Ø¯ÙŠÙ‚Ø©", "Ù…ÙˆØ§Ù‚Ù Ø§Ù„Ø³ÙŠØ§Ø±Ø§Øª",
    "ØºØ±ÙØ© Ø§Ù„Ø£Ù…Ù†", "Ø§Ù„Ù…Ø¶Ø®Ø§Øª", "ØºØ±ÙØ© Ø§Ù„ÙƒÙ‡Ø±Ø¨Ø§Ø¡",
]

# Ù…Ø¨Ù†Ù‰ T5 Ùˆ T6: 6 Ø£Ø¯ÙˆØ§Ø± Ã— 4 Ø´Ù‚Ù‚ = 24 Ø´Ù‚Ø© (Ø§Ù„Ø¯ÙˆØ± 1..6)
T56_UNITS = []
for floor in range(1, 7):
    start = (floor - 1) * 4 + 1
    for i in range(4):
        T56_UNITS.append(f"Ø§Ù„Ø¯ÙˆØ± {floor} - Ø´Ù‚Ø© {start + i}")

BUILDINGS = ["Ø§Ù„Ù…Ø±Ø§ÙÙ‚ Ø§Ù„Ø¹Ø§Ù…Ø©", "Ù…Ø¨Ù†Ù‰ T5", "Ù…Ø¨Ù†Ù‰ T6", "Ù…Ø¨Ù†Ù‰ T1"]

# Ø§Ù„Ù…ØµØ§Ø¯Ù‚Ø© Ø§Ù„Ø§Ø®ØªÙŠØ§Ø±ÙŠØ©
DASHBOARD_PASSWORD = os.environ.get("DASHBOARD_PASSWORD", "")
AUTH_COOKIE = "mn_maintenance_auth"
PROTECTED = bool(DASHBOARD_PASSWORD)


def auth_token() -> str:
    return hashlib.sha256(("mn-maintenance:" + DASHBOARD_PASSWORD).encode()).hexdigest()


def is_public_path(path: str) -> bool:
    return (
        path in ("/login", "/logout", "/healthz")
        or path.startswith(("/report", "/track/", "/qr", "/poster-qr", "/media", "/static"))
    )


# --------------------------------------------------------------------------- #
# Ø§Ù„Ù„ØºØ© (Ø«Ù†Ø§Ø¦ÙŠØ© Ø§Ù„Ù„ØºØ©: Ø¹Ø±Ø¨ÙŠ / Ø¥Ù†Ø¬Ù„ÙŠØ²ÙŠ)
# --------------------------------------------------------------------------- #
LANG_COOKIE = "mn_lang"
SUPPORTED_LANGS = ("ar", "en")


def pick_lang(lang: str | None, cookie: str | None = None) -> str:
    """Ø§Ø®ØªÙŠØ§Ø± Ø§Ù„Ù„ØºØ©: Ø£ÙˆÙ„ÙˆÙŠØ© Ø§Ù„Ù…Ø¹Ø§Ù…Ù„ Ø«Ù… Ø§Ù„ÙƒÙˆÙƒÙŠØ² Ø«Ù… Ø§Ù„Ø¹Ø±Ø¨ÙŠØ©."""
    for candidate in (lang, cookie):
        if candidate in SUPPORTED_LANGS:
            return candidate
    return "ar"


# Ù‚ÙˆØ§Ù…ÙŠØ³ Ø§Ù„ØªØ±Ø¬Ù…Ø© Ø§Ù„Ø´Ø§Ù…Ù„Ø© (Ø¹Ø±Ø¨ÙŠ -> Ø¥Ù†Ø¬Ù„ÙŠØ²ÙŠ Ø­Ø³Ø¨ Ø§Ù„Ù…ÙØªØ§Ø­)
AR = {}
EN = {}


def I(key, ar, en):
    """ØªØ³Ø¬ÙŠÙ„ Ù†Øµ Ù…ØªØ±Ø¬Ù…."""
    AR[key] = ar
    EN[key] = en


def tr(lang: str, key: str) -> str:
    table = EN if lang == "en" else AR
    return table.get(key, key)


def tr_value(lang: str, value: str, map_key: str = None) -> str:
    """ØªØ±Ø¬Ù…Ø© Ù‚ÙŠÙ…Ø© Ø¯ÙŠÙ†Ø§Ù…ÙŠÙƒÙŠØ© (Ù…Ø¨Ù†Ù‰/Ù‚Ø³Ù…/Ø­Ø§Ù„Ø©/Ø£ÙˆÙ„ÙˆÙŠØ©)."""
    if lang != "en" or not value:
        return value
    table = VALUE_EN[map_key] if map_key in VALUE_EN else {}
    return table.get(value, value)


VALUE_EN = {
    "status": {},
    "priority": {},
    "maintenance": {},
    "building": {},
}


def setup_values():
    for k, (ar, _c) in STATUS_META.items():
        VALUE_EN["status"][ar] = STATUS_META_EN.get(k, ar)
    for ar, en in PRIORITY_EN.items():
        VALUE_EN["priority"][ar] = en
    for ar, en in MAINTENANCE_EN.items():
        VALUE_EN["maintenance"][ar] = en
    for ar, en in BUILDING_EN.items():
        VALUE_EN["building"][ar] = en


STATUS_META_EN = {
    "pending": "Pending Approval",
    "approved": "Approved - In Progress",
    "done": "Completed & Closed",
    "rejected": "Rejected",
}
PRIORITY_EN = {"Ø¹Ø§Ø¯ÙŠØ©": "Normal", "Ù…Ø³ØªØ¹Ø¬Ù„Ø©": "Urgent", "Ø·Ø§Ø±Ø¦Ø©": "Critical"}
BUILDING_EN = {
    "Ø§Ù„Ù…Ø±Ø§ÙÙ‚ Ø§Ù„Ø¹Ø§Ù…Ø©": "Public Facilities",
    "Ù…Ø¨Ù†Ù‰ T5": "Building T5",
    "Ù…Ø¨Ù†Ù‰ T6": "Building T6",
    "Ù…Ø¨Ù†Ù‰ T1": "Building T1",
}
MAINTENANCE_EN = {
    "ÙƒÙ‡Ø±Ø¨Ø§Ø¡": "Electrical", "ØªÙƒÙŠÙŠÙ": "AC / Cooling", "Ø³Ø¨Ø§ÙƒØ©": "Plumbing",
    "Ù…Ø¯Ù†ÙŠ": "Civil", "Ù†Ø¬Ø§Ø±Ø©": "Carpentry", "Ø£Ù…Ù† ÙˆØ³Ù„Ø§Ù…Ø©": "Safety",
    "Ø£Ø®Ø±Ù‰": "Other",
}


# ======================================================================= #
# Ø§Ù„Ù†ØµÙˆØµ Ø§Ù„Ø¹Ø§Ù…Ø© (ÙˆØ§Ø¬Ù‡Ø© Ø«Ù†Ø§Ø¦ÙŠØ© Ø§Ù„Ù„ØºØ© ÙƒØ§Ù…Ù„Ø©)
# ======================================================================= #
I("nav_dash", "Ù„ÙˆØ­Ø© Ø§Ù„Ù…ØªØ§Ø¨Ø¹Ø©", "Dashboard")
I("nav_new", "Ø£Ù…Ø± Ø¹Ù…Ù„ Ø¬Ø¯ÙŠØ¯", "New Work Order")
I("nav_incoming", "Ø§Ù„Ø·Ù„Ø¨Ø§Øª Ø§Ù„ÙˆØ§Ø±Ø¯Ø©", "Incoming Requests")
I("nav_monthly", "Ø§Ù„ØªÙ‚Ø±ÙŠØ± Ø§Ù„Ø´Ù‡Ø±ÙŠ", "Monthly Report")
I("nav_poster", "Ù…Ù„ØµÙ‚ QR", "QR Poster")
I("nav_logout", "Ø®Ø±ÙˆØ¬", "Logout")
I("lang_switch", "EN", "Ø¹Ø±Ø¨ÙŠ")

I("dash_title", "Ù„ÙˆØ­Ø© Ù…ØªØ§Ø¨Ø¹Ø© Ø£ÙˆØ§Ù…Ø± Ø§Ù„Ø¹Ù…Ù„", "Work Orders Dashboard")
I("dash_totals", "Ø¥Ø¬Ù…Ø§Ù„ÙŠ Ø£ÙˆØ§Ù…Ø± Ø§Ù„Ø¹Ù…Ù„", "Total Work Orders")
I("dash_pending_review", "Ø¨Ø§Ù†ØªØ¸Ø§Ø± Ø§Ù„ØªØ¯Ù‚ÙŠÙ‚", "Pending Review")
I("dash_in_progress", "Ù‚ÙŠØ¯ Ø§Ù„ØªÙ†ÙÙŠØ°", "In Progress")
I("dash_done", "Ù…Ù†Ø¬Ø²Ø© ÙˆÙ…ØºÙ„Ù‚Ø©", "Completed & Closed")
I("dash_urgent", "Ø£ÙˆØ§Ù…Ø± Ø·Ø§Ø±Ø¦Ø©", "Critical Orders")
I("search_ph", "Ø¨Ø­Ø«: Ø§Ù„Ø§Ø³Ù… / Ø±Ù‚Ù… Ø§Ù„Ø£Ù…Ø± / Ø§Ù„Ù…ÙˆÙ‚Ø¹ / Ø§Ù„ÙÙ†ÙŠ / Ø§Ù„Ø¬ÙˆØ§Ù„...", "Search: name / order no / location / technician / phone...")
I("all_status", "ÙƒÙ„ Ø§Ù„Ø­Ø§Ù„Ø§Øª", "All Statuses")
I("filter", "ØªØµÙÙŠØ©", "Filter")
I("clear", "Ù…Ø³Ø­", "Clear")
I("col_order", "Ø±Ù‚Ù… Ø§Ù„Ø£Ù…Ø±", "Order No")
I("col_year", "Ø§Ù„Ø³Ù†Ø©", "Year")
I("col_title", "Ø§Ù„ØªØ§Ø±ÙŠØ®", "Date")
I("col_reporter", "Ø§Ø³Ù… Ø§Ù„Ù…Ø¨Ù„Ù‘Øº", "Reporter")
I("col_contact", "Ø±Ù‚Ù… Ø§Ù„ØªÙˆØ§ØµÙ„", "Contact")
I("col_location", "Ø§Ù„Ù…ÙˆÙ‚Ø¹", "Location")
I("col_section", "Ù†ÙˆØ¹ Ø§Ù„ØµÙŠØ§Ù†Ø©", "Maintenance")
I("col_problem", "Ù†ÙˆØ¹ Ø§Ù„Ù…Ø´ÙƒÙ„Ø©", "Problem Type")
I("col_priority", "Ø§Ù„Ø£ÙˆÙ„ÙˆÙŠØ©", "Priority")
I("col_technician", "Ø§Ù„ÙÙ†ÙŠ", "Technician")
I("col_status", "Ø§Ù„Ø­Ø§Ù„Ø©", "Status")
I("col_source", "Ø§Ù„Ù…ØµØ¯Ø±", "Source")
I("col_actions", "Ø¥Ø¬Ø±Ø§Ø¡Ø§Øª", "Actions")
I("action_view", "Ø¹Ø±Ø¶", "View")
I("action_edit", "ØªØ¹Ø¯ÙŠÙ„", "Edit")

I("new_title", "Ø¥Ù†Ø´Ø§Ø¡ Ø£Ù…Ø± Ø¹Ù…Ù„ Ø¬Ø¯ÙŠØ¯", "Create New Work Order")
I("new_sub", "ÙŠÙØ­ÙØ¸ Ø§Ù„Ø£Ù…Ø± Ù…Ø¨Ø§Ø´Ø±Ø© ÙˆÙŠØ¸Ù‡Ø± ÙÙˆØ±Ø§Ù‹ ÙÙŠ Ù„ÙˆØ­Ø© Ø§Ù„Ù…ØªØ§Ø¨Ø¹Ø©", "Saved instantly and shown in the dashboard")
I("f_reporter", "Ø§Ø³Ù… Ø§Ù„Ù…Ø¨Ù„Ù‘Øº", "Reporter Name")
I("f_contact", "Ø±Ù‚Ù… Ø§Ù„ØªÙˆØ§ØµÙ„", "Contact Number")
I("f_maintenance_type", "Ù†ÙˆØ¹ Ø§Ù„ØµÙŠØ§Ù†Ø©", "Maintenance Type")
I("f_problem_type", "Ù†ÙˆØ¹ Ø§Ù„Ù…Ø´ÙƒÙ„Ø©", "Problem Type")
I("f_priority", "Ø§Ù„Ø£ÙˆÙ„ÙˆÙŠØ©", "Priority")
I("f_building", "Ø§Ù„Ù…Ø¨Ù†Ù‰ / Ø§Ù„Ù…ÙˆÙ‚Ø¹", "Building / Location")
I("f_unit", "Ø§Ù„ÙˆØ­Ø¯Ø© / Ø§Ù„Ø´Ù‚Ø© / Ø§Ù„ØªÙØµÙŠÙ„", "Unit / Apartment / Detail")
I("f_technician", "Ø§Ù„ÙÙ†ÙŠ Ø§Ù„Ù…ÙƒÙ„Ù", "Assigned Technician")
I("f_description", "ÙˆØµÙ Ø§Ù„Ù…Ø´ÙƒÙ„Ø©", "Problem Description")
I("f_media", "Ù…Ø±ÙÙ‚Ø§Øª (ØµÙˆØ± / ÙÙŠØ¯ÙŠÙˆ)", "Attachments (Photo / Video)")
I("f_media_hint", "ÙŠÙ…ÙƒÙ†Ùƒ Ø¥Ø±ÙØ§Ù‚ Ø­ØªÙ‰ 5 ØµÙˆØ± Ø£Ùˆ Ù…Ù‚Ø·Ø¹ ÙÙŠØ¯ÙŠÙˆ Ù„Ù…Ø³Ø§Ø¹Ø¯Ø© Ø§Ù„ÙÙ†ÙŠ", "You can attach up to 5 photos or a video to help the technician")
I("f_location", "Ø§Ù„Ù…ÙˆÙ‚Ø¹ Ø§Ù„Ù…ÙØµÙ„", "Detailed Location")
I("btn_save", "ðŸ’¾ Ø­ÙØ¸ ÙˆØ¥ØµØ¯Ø§Ø± Ø§Ù„Ø£Ù…Ø±", "ðŸ’¾ Save & Issue Order")
I("btn_cancel", "Ø¥Ù„ØºØ§Ø¡", "Cancel")
I("btn_submit", "ðŸ“¨ Ø¥Ø±Ø³Ø§Ù„ Ø§Ù„Ø¨Ù„Ø§Øº", "ðŸ“¨ Submit Report")

I("title_report", "Ø¨Ù„Ø§Øº ØµÙŠØ§Ù†Ø©", "Maintenance Report")
I("sub_report", "Ù†Ù…ÙˆØ°Ø¬ Ø±ÙØ¹ Ø¨Ù„Ø§Øº / Ø·Ù„Ø¨ ØµÙŠØ§Ù†Ø© â€” ÙŠØµÙ„ ÙÙˆØ±Ø§Ù‹ Ø¥Ù„Ù‰ Ù‚Ø³Ù… Ø§Ù„ØµÙŠØ§Ù†Ø© Ù„Ù„ØªØ¯Ù‚ÙŠÙ‚ ÙˆØ§Ù„Ø§Ø¹ØªÙ…Ø§Ø¯", "Report / maintenance request form â€” reaches maintenance instantly for review & approval")
I("report_choose_issue", "Ø§Ø¶ØºØ· Ø¹Ù„Ù‰ Ù†ÙˆØ¹ Ø§Ù„ØµÙŠØ§Ù†Ø© Ø«Ù… Ø§Ø®ØªØ± ØªÙØ§ØµÙŠÙ„ Ø§Ù„Ù…Ø´ÙƒÙ„Ø©", "Select maintenance type then choose the issue details")

I("thanks_title", "ØªÙ… Ø§Ø³ØªÙ„Ø§Ù… Ø¨Ù„Ø§ØºÙƒ Ø¨Ù†Ø¬Ø§Ø­", "Your report was received successfully")
I("thanks_ref", "Ø±Ù‚Ù… Ù…Ø±Ø¬Ø¹ÙŠ Ù„ØªØªØ¨Ø¹ Ø¨Ù„Ø§ØºÙƒ", "Your reference number")
I("thanks_note", "Ø³ÙŠØ¯Ø±Ù‘Ø³ Ù‚Ø³Ù… Ø§Ù„ØµÙŠØ§Ù†Ø© Ø§Ù„Ø¨Ù„Ø§Øº ÙˆÙŠØ­ÙˆÙ‘Ù„Ù‡ Ø¥Ù„Ù‰ Ø£Ù…Ø± Ø¹Ù…Ù„ Ø±Ø³Ù…ÙŠ Ø®Ù„Ø§Ù„ Ø£Ù‚Ø±Ø¨ ÙˆÙ‚Øª.", "Maintenance will review and convert to an official work order soon.")
I("thanks_again", "ØªÙ‚Ø¯ÙŠÙ… Ø¨Ù„Ø§Øº Ø¢Ø®Ø±", "Submit Another Report")

I("track_status", "Ø­Ø§Ù„Ø© Ø£Ù…Ø± Ø§Ù„Ø¹Ù…Ù„", "Work order status")
I("track_notfound", "Ø±Ø§Ø¨Ø· ØºÙŠØ± ØµØ§Ù„Ø­", "Invalid link")
I("track_notfound_note", "Ù‡Ø°Ø§ Ø§Ù„Ø±Ù…Ø² ØºÙŠØ± Ù…Ø±ØªØ¨Ø· Ø¨Ø£Ù…Ø± Ø¹Ù…Ù„ Ù…ÙˆØ¬ÙˆØ¯.", "This code is not linked to an existing work order.")
I("track_inquiry", "Ù„Ù„Ø§Ø³ØªÙØ³Ø§Ø± ÙŠØ±Ø¬Ù‰ Ø§Ù„ØªÙˆØ§ØµÙ„ Ù…Ø¹ Ù‚Ø³Ù… Ø§Ù„ØµÙŠØ§Ù†Ø©", "For inquiries, contact the maintenance department")
I("track_assigned_soon", "Ø³ÙŠØªÙ… Ø§Ù„Ø¥Ø³Ù†Ø§Ø¯ Ù‚Ø±ÙŠØ¨Ø§Ù‹", "To be assigned soon")
I("f_completion", "ØªØ§Ø±ÙŠØ® Ø§Ù„Ø¥Ù†Ø¬Ø§Ø²", "Completion Date")
I("f_received", "ØªØ§Ø±ÙŠØ® Ø§Ù„Ø§Ø³ØªÙ„Ø§Ù…", "Received Date")

I("edit_title", "ØªØ¹Ø¯ÙŠÙ„ Ø£Ù…Ø± Ø§Ù„Ø¹Ù…Ù„", "Edit Work Order")
I("btn_save_changes", "ðŸ’¾ Ø­ÙØ¸ Ø§Ù„ØªØ¹Ø¯ÙŠÙ„Ø§Øª", "ðŸ’¾ Save Changes")
I("btn_update", "ðŸ’¾ Ø­ÙØ¸ Ø§Ù„ØªØ­Ø¯ÙŠØ«Ø§Øª", "ðŸ’¾ Save Updates")
I("col_notes", "Ù…Ù„Ø§Ø­Ø¸Ø§Øª / Ø§Ù„Ø¥Ø¬Ø±Ø§Ø¡", "Notes / Action Taken")
I("btn_delete", "Ø­Ø°Ù Ø§Ù„Ø£Ù…Ø±", "Delete Order")
I("btn_print_header", "ðŸ–¨ Ø·Ø¨Ø§Ø¹Ø© Ø¨Ø§Ù„Ù‡ÙŠØ¯Ø± Ø§Ù„Ø±Ø³Ù…ÙŠ", "ðŸ–¨ Print with Official Header")
I("btn_track", "ØµÙØ­Ø© Ø§Ù„ØªØªØ¨Ø¹", "Tracking Page")
I("btn_print", "ðŸ–¨ Ø·Ø¨Ø§Ø¹Ø©", "ðŸ–¨ Print")

I("inc_title", "Ø§Ù„Ø·Ù„Ø¨Ø§Øª Ø§Ù„ÙˆØ§Ø±Ø¯Ø© â€” Ø¨Ø§Ù†ØªØ¸Ø§Ø± Ø§Ù„ØªØ¯Ù‚ÙŠÙ‚", "Incoming Requests â€” Pending Review")
I("inc_sub", "Ø§Ù„Ø¨Ù„Ø§ØºØ§Øª Ø§Ù„Ù…Ø±ÙÙˆØ¹Ø© Ø¹Ø¨Ø± Ù…Ø³Ø­ Ø±Ù…Ø² QR ØªØ¸Ù‡Ø± Ù‡Ù†Ø§ Ø£ÙˆÙ„Ø§Ù‹ ÙˆÙ„Ø§ ØªØ¯Ø®Ù„ Ø§Ù„Ø³Ø¬Ù„ Ø§Ù„Ø±Ø³Ù…ÙŠ Ø¥Ù„Ø§ Ø¨Ø¹Ø¯ Ø§Ø¹ØªÙ…Ø§Ø¯Ù‡Ø§", "QR submissions appear here first and enter the official log only after approval")
I("btn_approve", "Ø§Ø¹ØªÙ…Ø§Ø¯ âœ“", "Approve âœ“")
I("btn_reject", "Ø±ÙØ¶ âœ•", "Reject âœ•")
I("btn_review", "ØªØ¯Ù‚ÙŠÙ‚", "Review")
I("inc_empty_title", "Ù„Ø§ ØªÙˆØ¬Ø¯ Ø·Ù„Ø¨Ø§Øª ÙˆØ§Ø±Ø¯Ø© Ø¬Ø¯ÙŠØ¯Ø©", "No new incoming requests")
I("inc_empty_note", "Ø¹Ù†Ø¯ Ù…Ø³Ø­ Ø±Ù…Ø² QR Ø§Ù„Ù…ÙˆØ¬ÙˆØ¯ Ø¹Ù„Ù‰ Ù…Ù„ØµÙ‚ Ø§Ù„Ø¨Ù„Ø§ØºØ§Øª Ø³ÙŠØ¸Ù‡Ø± Ø§Ù„Ø¨Ù„Ø§Øº Ù‡Ù†Ø§ Ù„Ù„ØªØ¯Ù‚ÙŠÙ‚ ÙˆØ§Ù„Ø§Ø¹ØªÙ…Ø§Ø¯.", "Scanning the QR poster will bring reports here for review and approval.")

I("mth_title", "Ø§Ù„ØªÙ‚Ø±ÙŠØ± Ø§Ù„Ø´Ù‡Ø±ÙŠ Ù„Ø£ÙˆØ§Ù…Ø± Ø§Ù„Ø¹Ù…Ù„", "Monthly Work Orders Report")
I("mth_total", "Ø¥Ø¬Ù…Ø§Ù„ÙŠ Ø§Ù„Ø£ÙˆØ§Ù…Ø±", "Total Orders")
I("mth_done", "Ù…Ù†Ø¬Ø²Ø© ÙˆÙ…ØºÙ„Ù‚Ø©", "Completed & Closed")
I("mth_in_progress", "Ù‚ÙŠØ¯ Ø§Ù„ØªÙ†ÙÙŠØ°", "In Progress")
I("mth_pending", "Ø¨Ø§Ù†ØªØ¸Ø§Ø± Ø§Ù„Ø§Ø¹ØªÙ…Ø§Ø¯", "Pending Approval")
I("mth_rejected", "Ù…Ø±ÙÙˆØ¶Ø©", "Rejected")
I("mth_urgent", "Ø£ÙˆØ§Ù…Ø± Ø·Ø§Ø±Ø¦Ø©", "Critical Orders")
I("mth_rate", "Ù†Ø³Ø¨Ø© Ø§Ù„Ø¥Ù†Ø¬Ø§Ø²", "Completion Rate")
I("mth_by_section", "Ù‚Ø³Ù… Ø§Ù„ØµÙŠØ§Ù†Ø©", "Section")
I("mth_by_priority", "Ø§Ù„Ø£ÙˆÙ„ÙˆÙŠØ©", "Priority")
I("mth_by_status", "Ø§Ù„Ø­Ø§Ù„Ø©", "Status")
I("mth_by_source", "Ø§Ù„Ù…ØµØ¯Ø±", "Source")
I("mth_details", "ØªÙØ§ØµÙŠÙ„ Ø£ÙˆØ§Ù…Ø± Ø§Ù„Ø¹Ù…Ù„", "Work order details")
I("mth_show", "Ø¹Ø±Ø¶", "Show")
I("mth_print", "ðŸ–¨ Ø·Ø¨Ø§Ø¹Ø© Ø§Ù„ØªÙ‚Ø±ÙŠØ±", "ðŸ–¨ Print Report")
I("mth_count", "Ø§Ù„Ø¹Ø¯Ø¯", "Count")

I("login_title", "Ù„ÙˆØ­Ø© Ø§Ù„ØªØ­ÙƒÙ…", "Dashboard")
I("login_pass", "ÙƒÙ„Ù…Ø© Ø§Ù„Ù…Ø±ÙˆØ±", "Password")
I("login_btn", "ØªØ³Ø¬ÙŠÙ„ Ø§Ù„Ø¯Ø®ÙˆÙ„", "Sign In")
I("login_err", "ÙƒÙ„Ù…Ø© Ø§Ù„Ù…Ø±ÙˆØ± ØºÙŠØ± ØµØ­ÙŠØ­Ø©. Ø­Ø§ÙˆÙ„ Ù…Ø±Ø© Ø£Ø®Ø±Ù‰.", "Incorrect password. Try again.")
I("login_ph", "â€¢â€¢â€¢â€¢â€¢â€¢â€¢â€¢", "â€¢â€¢â€¢â€¢â€¢â€¢â€¢â€¢")

I("f_unit_placeholder", "Ù…Ø«Ø§Ù„: Ø§Ù„Ø¯ÙˆØ± 3 - Ø´Ù‚Ø© 12", "e.g. Floor 3 - Apartment 12")
I("select_placeholder", "Ø§Ø®ØªØ±...", "Select...")

setup_values()


# --------------------------------------------------------------------------- #
# Ù‚Ø§Ø¹Ø¯Ø© Ø§Ù„Ø¨ÙŠØ§Ù†Ø§Øª
# --------------------------------------------------------------------------- #
DB_PATH.parent.mkdir(parents=True, exist_ok=True)


@contextmanager
def db():
    conn = __import__("sqlite3").connect(DB_PATH)
    conn.row_factory = __import__("sqlite3").Row
    conn.execute("PRAGMA foreign_keys = ON")
    try:
        yield conn
        conn.commit()
    finally:
        conn.close()


def init_db() -> None:
    with db() as c:
        c.executescript(
            """
            CREATE TABLE IF NOT EXISTS work_orders (
                id            INTEGER PRIMARY KEY AUTOINCREMENT,
                order_no      TEXT UNIQUE,
                reference_id  TEXT UNIQUE,
                reporter_name TEXT NOT NULL,
                contact       TEXT NOT NULL DEFAULT '',
                location      TEXT NOT NULL DEFAULT '',
                building      TEXT NOT NULL DEFAULT '',
                unit          TEXT NOT NULL DEFAULT '',
                section       TEXT NOT NULL DEFAULT '',
                problem_type  TEXT NOT NULL DEFAULT '',
                priority      TEXT NOT NULL DEFAULT 'Ø¹Ø§Ø¯ÙŠØ©',
                technician    TEXT NOT NULL DEFAULT '',
                description   TEXT NOT NULL DEFAULT '',
                notes         TEXT NOT NULL DEFAULT '',
                media         TEXT NOT NULL DEFAULT '',
                status        TEXT NOT NULL DEFAULT 'pending',
                source        TEXT NOT NULL DEFAULT 'manual',
                token         TEXT UNIQUE,
                created_at    TEXT NOT NULL,
                completed_at  TEXT
            );
            CREATE INDEX IF NOT EXISTS idx_wo_status ON work_orders(status);
            CREATE INDEX IF NOT EXISTS idx_wo_created ON work_orders(created_at);
            """
        )
        # ØªØ±Ø­ÙŠÙ„ Ù„Ù„Ø£Ø±Ø´ÙŠÙ: Ø¥Ø¶Ø§ÙØ© Ø§Ù„Ø£Ø¹Ù…Ø¯Ø© Ø§Ù„Ø¬Ø¯ÙŠØ¯Ø© Ø¥Ù† Ù„Ù… ØªÙƒÙ† Ù…ÙˆØ¬ÙˆØ¯Ø© (ØªÙˆØ§ÙÙ‚ Ø¨ÙŠØ§Ù†Ø§Øª Ù‚Ø¯ÙŠÙ…Ø©)
        existing = {r[1] for r in c.execute("PRAGMA table_info(work_orders)").fetchall()}
        mig = {
            "reference_id": "TEXT",
            "building": "TEXT NOT NULL DEFAULT ''",
            "unit": "TEXT NOT NULL DEFAULT ''",
            "problem_type": "TEXT NOT NULL DEFAULT ''",
            "media": "TEXT NOT NULL DEFAULT ''",
        }
        for col, ddl in mig.items():
            if col not in existing:
                c.execute(f"ALTER TABLE work_orders ADD COLUMN {col} {ddl}")


init_db()


def now_local() -> datetime:
    return datetime.now(TZ).replace(tzinfo=None)


def fmt(dt: datetime, with_time: bool = True) -> str:
    s = dt.strftime("%Y-%m-%d")
    if with_time:
        s += " " + dt.strftime("%H:%M")
    return s


# --------------------------------------------------------------------------- #
# ØªØ·Ø¨ÙŠÙ‚ FastAPI
# --------------------------------------------------------------------------- #
app = FastAPI(
    title=f"Ù†Ø¸Ø§Ù… Ø£ÙˆØ§Ù…Ø± Ø§Ù„Ø¹Ù…Ù„ - {HOSPITAL_NAME}",
    docs_url="/api/docs",
    redoc_url="/api/redoc",
    openapi_url="/api/openapi.json",
)
templates = Jinja2Templates(directory=str(TEMPLATES_DIR))


def _load_meta(lang: str):
    return {
        k: (v[0], v[1]) for k, v in STATUS_META.items()
    }


templates.env.globals.update(
    HOSPITAL_NAME=HOSPITAL_NAME,
    ORG_RIGHT=ORG_RIGHT,
    ORG_LEFT=ORG_LEFT,
    DEPARTMENT_NAME=DEPARTMENT_NAME,
    LOGO=("/static/" + LOGO_PATH.name) if LOGO_PATH else None,
    MAINTENANCE_TYPES=MAINTENANCE_TYPES,
    PROBLEM_TYPES=PROBLEM_TYPES,
    PUBLIC_FACILITIES=PUBLIC_FACILITIES,
    T56_UNITS=T56_UNITS,
    BUILDINGS=BUILDINGS,
    PRIORITIES=PRIORITIES,
    STATUS_META=STATUS_META,
    MONTHS_AR=MONTHS_AR,
    MONTHS_EN=MONTHS_EN,
)
app.mount("/static", StaticFiles(directory=str(STATIC_DIR)), name="static")


# ÙˆØ³Ù… ØªØ­ÙˆÙŠÙ„ Ø§Ù„ØªØ§Ø±ÙŠØ® Ù„Ù„Ù†Ù…Ø§Ø°Ø¬
def dt_filter(value, with_time: bool = True):
    if not value:
        return "â€”"
    try:
        return fmt(datetime.strptime(str(value), "%Y-%m-%d %H:%M"), with_time)
    except Exception:
        try:
            return fmt(datetime.strptime(str(value), "%Y-%m-%d"), with_time)
        except Exception:
            return str(value)


templates.env.filters["dt"] = dt_filter


# --------------------------------------------------------------------------- #
# Ø§Ù„Ù…ØµØ§Ø¯Ù‚Ø©
# --------------------------------------------------------------------------- #
@app.middleware("http")
async def auth_gate(request: Request, call_next):
    if PROTECTED and not is_public_path(request.url.path):
        if request.cookies.get(AUTH_COOKIE) != auth_token():
            nxt = request.url.path
            if request.url.query:
                nxt += "?" + request.url.query
            return RedirectResponse(f"/login?next={nxt}", status_code=303)
    return await call_next(request)


@app.middleware("http")
async def lang_gate(request: Request, call_next):
    """ØªØ­Ø¯ÙŠØ¯ Ø§Ù„Ù„ØºØ© Ø§Ù„Ø­Ø§Ù„ÙŠØ© Ù…Ù† Ø§Ù„Ù…Ø¹Ø§Ù…Ù„ ?lang= Ø£Ùˆ Ù…Ù† Ø§Ù„ÙƒÙˆÙƒÙŠØ²ØŒ ÙˆØªØ®Ø²ÙŠÙ†Ù‡Ø§ ÙÙŠ Ø­Ø§Ù„Ø© Ø§Ù„Ø·Ù„Ø¨."""
    request.state.lang = pick_lang(request.query_params.get("lang"), request.cookies.get(LANG_COOKIE))
    response = await call_next(request)
    # Ø­ÙØ¸ ØªÙØ¶ÙŠÙ„ Ø§Ù„Ù„ØºØ© ÙÙŠ Ø§Ù„ÙƒÙˆÙƒÙŠØ² Ù„ÙŠØ³ØªÙ…Ø± Ø¹Ø¨Ø± Ø§Ù„ØµÙØ­Ø§Øª
    if request.query_params.get("lang"):
        response.set_cookie(
            LANG_COOKIE, request.state.lang,
            max_age=60 * 60 * 24 * 365, samesite="lax",
        )
    return response


@app.get("/login", response_class=HTMLResponse)
def login_page(request: Request, next: str = "/", err: str = ""):
    return templates.TemplateResponse(
        request, "login.html",
        context(request, next=next or "/", err=err),
    )


@app.post("/login")
def login_submit(password: str = Form(""), next: str = Form("/")):
    if password != DASHBOARD_PASSWORD:
        return RedirectResponse(f"/login?err=1&next={next}", status_code=303)
    safe_next = next if next.startswith("/") and not next.startswith("//") else "/"
    resp = RedirectResponse(safe_next, status_code=303)
    resp.set_cookie(
        AUTH_COOKIE,
        auth_token(),
        max_age=60 * 60 * 24 * 30,
        httponly=True,
        samesite="lax",
        secure=os.environ.get("COOKIE_SECURE", "1") == "1",
    )
    return resp


@app.get("/logout")
def logout():
    resp = RedirectResponse("/login", status_code=303)
    resp.delete_cookie(AUTH_COOKIE)
    return resp


# --------------------------------------------------------------------------- #
# Ø¯ÙˆØ§Ù„ Ù…Ø³Ø§Ø¹Ø¯Ø©
# --------------------------------------------------------------------------- #
FLASH_MESSAGES = {
    "created": ("ØªÙ… Ø¥Ù†Ø´Ø§Ø¡ Ø£Ù…Ø± Ø§Ù„Ø¹Ù…Ù„ Ø¨Ù†Ø¬Ø§Ø­.", "ØªÙ… Ø¥Ù†Ø´Ø§Ø¡ Ø£Ù…Ø± Ø§Ù„Ø¹Ù…Ù„ Ø¨Ù†Ø¬Ø§Ø­.", "ok"),
    "updated": ("ØªÙ… ØªØ­Ø¯ÙŠØ« Ø¨ÙŠØ§Ù†Ø§Øª Ø£Ù…Ø± Ø§Ù„Ø¹Ù…Ù„.", "ØªÙ… ØªØ­Ø¯ÙŠØ« Ø¨ÙŠØ§Ù†Ø§Øª Ø£Ù…Ø± Ø§Ù„Ø¹Ù…Ù„.", "ok"),
    "deleted": ("ØªÙ… Ø­Ø°Ù Ø£Ù…Ø± Ø§Ù„Ø¹Ù…Ù„.", "ØªÙ… Ø­Ø°Ù Ø£Ù…Ø± Ø§Ù„Ø¹Ù…Ù„.", "ok"),
    "approved": ("ØªÙ… Ø§Ø¹ØªÙ…Ø§Ø¯ Ø§Ù„Ø·Ù„Ø¨ ÙˆØªØ­ÙˆÙŠÙ„Ù‡ Ø¥Ù„Ù‰ Ø£Ù…Ø± Ø¹Ù…Ù„.", "ØªÙ… Ø§Ø¹ØªÙ…Ø§Ø¯ Ø§Ù„Ø·Ù„Ø¨ ÙˆØªØ­ÙˆÙŠÙ„Ù‡ Ø¥Ù„Ù‰ Ø£Ù…Ø± Ø¹Ù…Ù„.", "ok"),
    "rejected": ("ØªÙ… Ø±ÙØ¶ Ø§Ù„Ø·Ù„Ø¨.", "ØªÙ… Ø±ÙØ¶ Ø§Ù„Ø·Ù„Ø¨.", "warn"),
    "err": ("ÙŠØ±Ø¬Ù‰ ØªØ¹Ø¨Ø¦Ø© Ø¬Ù…ÙŠØ¹ Ø§Ù„Ø­Ù‚ÙˆÙ„ Ø§Ù„Ø¥Ù„Ø²Ø§Ù…ÙŠØ©.", "ÙŠØ±Ø¬Ù‰ ØªØ¹Ø¨Ø¦Ø© Ø¬Ù…ÙŠØ¹ Ø§Ù„Ø­Ù‚ÙˆÙ„ Ø§Ù„Ø¥Ù„Ø²Ø§Ù…ÙŠØ©.", "bad"),
}

def context(request: Request, **extra) -> dict:
    with db() as c:
        pending_count = c.execute(
            "SELECT COUNT(*) FROM work_orders WHERE status='pending'"
        ).fetchone()[0]
    lang = getattr(request.state, "lang", "ar")
    # Ø¬Ù…Ø¹ Ø¥Ø´Ø¹Ø§Ø±Ø§Øª Ø§Ù„ÙˆÙ…ÙŠØ¶ Ù…Ù† Ù…Ø¹Ø§Ù…Ù„Ø§Øª Ø§Ù„Ø§Ø³ØªØ¹Ù„Ø§Ù… (created=1, deleted=1, err=1 ...)
    flashes = []
    for key, qv in request.query_params.items():
        if key in FLASH_MESSAGES and qv in ("1",):
            ar, en, level = FLASH_MESSAGES[key]
            flashes.append((level, en if lang == "en" else ar))
    data = {
        "request": request,
        "path": request.url.path,
        "pending_count": pending_count,
        "protected": PROTECTED,
        "flashes": flashes,
        "now": now_local(),
        "lang": lang,
        "LANG": lang,
        "tr": lambda key: tr(lang, key),
        "trv": lambda value, mk=None: tr_value(lang, value, mk),
        "T_STATUS": lambda v: tr_value(lang, v, "status"),
        "T_PRIO": lambda v: tr_value(lang, v, "priority"),
        "T_MAINT": lambda v: tr_value(lang, v, "maintenance"),
        "T_BUILD": lambda v: tr_value(lang, v, "building"),
    }
    data.update(extra)
    return data


def base_url(request: Request) -> str:
    env = os.environ.get("PUBLIC_BASE_URL", "").strip().rstrip("/")
    if env:
        return env
    return str(request.base_url).rstrip("/")


def make_qr_png(url: str, with_logo: bool = True) -> Response:
    qr = qrcode.QRCode(border=2, box_size=10, error_correction=qrcode.constants.ERROR_CORRECT_H)
    qr.add_data(url)
    qr.make(fit=True)
    img = qr.make_image(fill_color="#0f172a", back_color="white").convert("RGB")
    # Ø¯Ù…Ø¬ Ø´Ø¹Ø§Ø± ØªØ¬Ù…Ø¹ Ø¬Ø§Ø²Ø§Ù† ÙÙŠ Ù…Ø±ÙƒØ² Ø±Ù…Ø² QR
    if with_logo and LOGO_PATH:
        try:
            from PIL import Image
            logo = Image.open(LOGO_PATH).convert("RGBA")
            logo = logo.resize((70, 70), Image.LANCZOS)
            # Ø®Ù„ÙÙŠØ© Ø¨ÙŠØ¶Ø§Ø¡ Ø®Ù„Ù Ø§Ù„Ø´Ø¹Ø§Ø± Ù„Ø¥Ø¨Ø±Ø§Ø² Ø§Ù„ÙˆØ¶ÙˆØ­
            bg = Image.new("RGBA", logo.size, (255, 255, 255, 255))
            base = img
            # Ù…Ø±ÙƒØ² Ø§Ù„ØµÙˆØ±Ø©
            w, h = img.size
            cx, cy = (w - logo.width) // 2, (h - logo.height) // 2
            bg.paste(logo, (0, 0), logo)
            base.paste(bg, (cx, cy), bg)
            bio = io.BytesIO()
            base.save(bio, format="PNG")
            return Response(content=bio.getvalue(), media_type="image/png")
        except Exception:
            pass
    bio = io.BytesIO()
    img.save(bio, format="PNG")
    return Response(content=bio.getvalue(), media_type="image/png")


def save_media(files) -> list[str]:
    """Ø­ÙØ¸ Ø§Ù„Ù…Ù„ÙØ§Øª Ø§Ù„Ù…Ø±ÙÙˆØ¹Ø© Ø¹Ù„Ù‰ Ø§Ù„Ù‚Ø±Øµ Ø§Ù„Ø¯Ø§Ø¦Ù…ØŒ ÙˆØ¥Ø±Ø¬Ø§Ø¹ Ø£Ø³Ù…Ø§Ø¡ Ø§Ù„Ù…Ù„ÙØ§Øª Ø§Ù„Ù†Ø³Ø¨ÙŠØ©."""
    saved = []
    for f in (files or []):
        if not f or not f.filename:
            continue
        # Ø§Ù„ØªØ­Ù‚Ù‚ Ù…Ù† Ø§Ù„Ù†ÙˆØ¹ ÙˆØ§Ù„Ø³Ù„Ø§Ù…Ø©
        ext = Path(f.filename).suffix.lower()
        if ext not in ALLOWED_MEDIA:
            continue
        if len(saved) >= MAX_MEDIA:
            break
        # Ø§Ø³Ù… Ù…Ù„Ù Ø¢Ù…Ù† ÙˆÙØ±ÙŠØ¯
        safe_name = secrets.token_hex(6) + ext
        dest = UPLOAD_DIR / safe_name
        try:
            with dest.open("wb") as out:
                shutil.copyfileobj(f.file, out)
            saved.append(f"/{MEDIA_URL_PREFIX.lstrip('/')}/{safe_name}")
        except Exception:
            continue
    return saved


# Ø¯Ø§Ù„Ø© Ø¥Ø±Ø¬Ø§Ø¹ Ù‚ÙˆØ§Ø¦Ù… Ø§Ù„Ù…Ø±ÙÙ‚Ø§Øª Ø¨ØµÙŠØºØ© Ø¢Ù…Ù†Ø©
def media_list(media_str: str) -> list[str]:
    if not media_str:
        return []
    return [p.strip() for p in media_str.split(",") if p.strip()]


def insert_order(reporter_name, contact, location, section, priority,
                 technician, description, problem_type="", building="",
                 unit="", media="", status="pending", source="manual") -> tuple[int, str, str]:
    if priority not in PRIORITIES:
        priority = "Ø¹Ø§Ø¯ÙŠØ©"
    now = now_local()
    with db() as c:
        cur = c.execute(
            """
            INSERT INTO work_orders
                (reporter_name, contact, location, building, unit, section,
                 problem_type, priority, technician, description, media,
                 status, source, token, created_at, completed_at)
            VALUES (?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)
            """,
            (
                reporter_name.strip(), contact.strip(), location.strip(),
                building.strip(), unit.strip(), section.strip() or "Ø£Ø®Ø±Ù‰",
                problem_type.strip(), priority, technician.strip(),
                description.strip(), media, status, source,
                secrets.token_hex(8), fmt(now),
                fmt(now) if status == "done" and source == "manual" else None,
            ),
        )
        oid = cur.lastrowid
        order_no = f"WO-{now:%Y%m}-{oid:04d}"
        reference_id = f"R{now:%Y%m%d}-{secrets.token_hex(3).upper()}"
        c.execute(
            "UPDATE work_orders SET order_no=?, reference_id=? WHERE id=?",
            (order_no, reference_id, oid),
        )
    return oid, order_no, reference_id


def get_order_or_404(request: Request, order_id: int):
    with db() as c:
        row = c.execute("SELECT * FROM work_orders WHERE id=?", (order_id,)).fetchone()
    if row is None:
        return None
    return row


# --------------------------------------------------------------------------- #
# Ù„ÙˆØ­Ø© Ø§Ù„ØªØ­ÙƒÙ… Ø§Ù„Ø±Ø¦ÙŠØ³ÙŠØ©
# --------------------------------------------------------------------------- #
@app.get("/", response_class=HTMLResponse)
def dashboard(request: Request, status: str = "", q: str = ""):
    sql = "SELECT * FROM work_orders"
    conds, params = [], []
    if status in STATUS_META:
        conds.append("status=?")
        params.append(status)
    if q.strip():
        like = f"%{q.strip()}%"
        conds.append(
            "(reporter_name LIKE ? OR location LIKE ? OR order_no LIKE ? "
            "OR description LIKE ? OR technician LIKE ? OR contact LIKE ?)"
        )
        params.extend([like] * 6)
    if conds:
        sql += " WHERE " + " AND ".join(conds)
    sql += " ORDER BY id DESC LIMIT 500"

    with db() as c:
        orders = c.execute(sql, params).fetchall()
        stats = c.execute(
            """
            SELECT
                COUNT(*)                                     AS total,
                COUNT(CASE WHEN status='pending'  THEN 1 END) AS pending,
                COUNT(CASE WHEN status='approved' THEN 1 END) AS approved,
                COUNT(CASE WHEN status='done'     THEN 1 END) AS done,
                COUNT(CASE WHEN status='rejected' THEN 1 END) AS rejected,
                COUNT(CASE WHEN priority='Ø·Ø§Ø±Ø¦Ø©'  THEN 1 END) AS urgent
            FROM work_orders
            """
        ).fetchone()

    return templates.TemplateResponse(
        request, "dashboard.html",
        context(request, orders=orders, stats=stats, q=q, status=status),
    )


# --------------------------------------------------------------------------- #
# Ø¥Ù†Ø´Ø§Ø¡ Ø£Ù…Ø± Ø¹Ù…Ù„
# --------------------------------------------------------------------------- #
@app.get("/orders/new", response_class=HTMLResponse)
def new_order_page(request: Request):
    return templates.TemplateResponse(request, "new_order.html", context(request))


@app.post("/orders/create")
def create_order(
    reporter_name: str = Form(...),
    contact: str = Form(""),
    location: str = Form(""),
    building: str = Form(""),
    unit: str = Form(""),
    section: str = Form(""),
    problem_type: str = Form(""),
    priority: str = Form("Ø¹Ø§Ø¯ÙŠØ©"),
    technician: str = Form(""),
    description: str = Form(""),
    media_files: list[UploadFile] = File(default=[]),
):
    if not reporter_name.strip() or not description.strip():
        return RedirectResponse("/orders/new?err=1", status_code=303)
    # Ø¨Ù†Ø§Ø¡ Ø§Ù„Ù…ÙˆÙ‚Ø¹ Ø§Ù„Ù…ÙØµÙ„ Ù…Ù† Ø§Ù„Ù…Ø¨Ù†Ù‰ + Ø§Ù„ÙˆØ­Ø¯Ø© Ø£Ùˆ Ø§Ù„Ø­Ù‚Ù„ Ø§Ù„ÙŠØ¯ÙˆÙŠ
    if not location.strip():
        location = " / ".join(p for p in [building, unit] if p.strip())
    saved = save_media(media_files)
    oid, _order_no, _ref = insert_order(
        reporter_name, contact, location, section, priority,
        technician, description,
        problem_type=problem_type, building=building, unit=unit,
        media=",".join(saved), status="approved", source="manual",
    )
    return RedirectResponse(f"/orders/{oid}?created=1", status_code=303)


# --------------------------------------------------------------------------- #
# ØªÙØ§ØµÙŠÙ„ Ø£Ù…Ø± Ø§Ù„Ø¹Ù…Ù„
# --------------------------------------------------------------------------- #
@app.get("/orders/{order_id}", response_class=HTMLResponse)
def order_detail(order_id: int, request: Request):
    row = get_order_or_404(request, order_id)
    if row is None:
        return templates.TemplateResponse(request, "404.html", context(request), status_code=404)
    return templates.TemplateResponse(
        request, "order_detail.html",
        context(request, o=row, media=media_list(row["media"])),
    )


@app.get("/orders/{order_id}/edit", response_class=HTMLResponse)
def order_edit_page(order_id: int, request: Request):
    row = get_order_or_404(request, order_id)
    if row is None:
        return templates.TemplateResponse(request, "404.html", context(request), status_code=404)
    return templates.TemplateResponse(
        request, "order_edit.html",
        context(request, o=row, media=media_list(row["media"])),
    )


@app.post("/orders/{order_id}/update")
def update_order(order_id: int, status: str = Form(...), technician: str = Form(""),
                 notes: str = Form("")):
    if status not in STATUS_META:
        raise HTTPException(status_code=400, detail="invalid status")
    now = now_local()
    with db() as c:
        existing = c.execute(
            "SELECT status, completed_at FROM work_orders WHERE id=?", (order_id,)
        ).fetchone()
        if existing is None:
            raise HTTPException(status_code=404, detail="not found")
        completed_at = existing["completed_at"]
        if status == "done" and existing["status"] != "done":
            completed_at = fmt(now)
        elif status != "done":
            completed_at = None
        c.execute(
            """
            UPDATE work_orders
            SET status=?, technician=?, notes=?,
                completed_at=CASE WHEN ?!='' THEN ? ELSE ? END
            WHERE id=?
            """,
            (
                status,
                technician.strip() or existing["technician"],
                notes.strip(),
                completed_at, completed_at, completed_at,
                order_id,
            ),
        )
    return RedirectResponse(f"/orders/{order_id}?updated=1", status_code=303)


@app.post("/orders/{order_id}/edit")
def order_edit_submit(
    order_id: int,
    reporter_name: str = Form(...),
    contact: str = Form(""),
    location: str = Form(""),
    building: str = Form(""),
    unit: str = Form(""),
    section: str = Form(""),
    problem_type: str = Form(""),
    priority: str = Form("Ø¹Ø§Ø¯ÙŠØ©"),
    technician: str = Form(""),
    description: str = Form(""),
):
    if not reporter_name.strip():
        return RedirectResponse(f"/orders/{order_id}/edit?err=1", status_code=303)
    if not location.strip():
        location = " / ".join(p for p in [building, unit] if p.strip())
    with db() as c:
        c.execute(
            """
            UPDATE work_orders
            SET reporter_name=?, contact=?, location=?, building=?, unit=?,
                section=?, problem_type=?, priority=?, technician=?, description=?
            WHERE id=?
            """,
            (
                reporter_name.strip(), contact.strip(), location.strip(),
                building.strip(), unit.strip(), section.strip(),
                problem_type.strip(), priority, technician.strip(),
                description.strip(), order_id,
            ),
        )
    return RedirectResponse(f"/orders/{order_id}?updated=1", status_code=303)


@app.post("/orders/{order_id}/delete")
def delete_order(order_id: int):
    with db() as c:
        c.execute("DELETE FROM work_orders WHERE id=?", (order_id,))
    return RedirectResponse("/?deleted=1", status_code=303)


# --------------------------------------------------------------------------- #
# Ø·Ø¨Ø§Ø¹Ø© Ø£Ù…Ø± Ø§Ù„Ø¹Ù…Ù„ (Ø¨Ø§Ù„Ù‡ÙŠØ¯Ø± Ø§Ù„Ø±Ø³Ù…ÙŠ)
# --------------------------------------------------------------------------- #
@app.get("/orders/{order_id}/print", response_class=HTMLResponse)
def print_order(order_id: int, request: Request, autoprint: int = 0):
    row = get_order_or_404(request, order_id)
    if row is None:
        return templates.TemplateResponse(request, "404.html", context(request), status_code=404)
    return templates.TemplateResponse(
        request, "print_order.html",
        context(request, o=row, autoprint=bool(autoprint), media=media_list(row["media"])),
    )


# --------------------------------------------------------------------------- #
# Ø§Ù„Ø·Ù„Ø¨Ø§Øª Ø§Ù„ÙˆØ§Ø±Ø¯Ø© (ØªØ¯Ù‚ÙŠÙ‚ Ø¨Ù„Ø§ØºØ§Øª QR)
# --------------------------------------------------------------------------- #
@app.get("/incoming", response_class=HTMLResponse)
def incoming(request: Request):
    with db() as c:
        rows = c.execute(
            "SELECT * FROM work_orders WHERE status='pending' ORDER BY id ASC"
        ).fetchall()
    return templates.TemplateResponse(request, "incoming.html", context(request, rows=rows))


@app.post("/incoming/{order_id}/approve")
def approve_request(order_id: int):
    with db() as c:
        c.execute(
            "UPDATE work_orders SET status='approved' WHERE id=? AND status='pending'",
            (order_id,),
        )
    return RedirectResponse(f"/orders/{order_id}?approved=1", status_code=303)


@app.post("/incoming/{order_id}/reject")
def reject_request(order_id: int):
    with db() as c:
        c.execute(
            "UPDATE work_orders SET status='rejected' WHERE id=? AND status='pending'",
            (order_id,),
        )
    return RedirectResponse("/incoming?rejected=1", status_code=303)


# --------------------------------------------------------------------------- #
# Ø±Ù…ÙˆØ² QR
# --------------------------------------------------------------------------- #
@app.get("/qr/{order_id}")
def qr_for_order(order_id: int, request: Request):
    row = get_order_or_404(request, order_id)
    if row is None:
        return Response(status_code=404)
    return make_qr_png(f"{base_url(request)}/track/{row['token']}")


@app.get("/poster-qr")
def poster_qr(request: Request, location: str = ""):
    url = f"{base_url(request)}/report"
    if location:
        from urllib.parse import quote
        url += f"?location={quote(location)}"
    return make_qr_png(url)


@app.get("/poster", response_class=HTMLResponse)
def poster_page(request: Request, location: str = ""):
    return templates.TemplateResponse(request, "poster.html", context(request, location=location))


# --------------------------------------------------------------------------- #
# Ø§Ù„Ù†Ù…ÙˆØ°Ø¬ Ø§Ù„Ø¹Ø§Ù… (ÙŠÙÙØªØ­ Ø¨Ù…Ø³Ø­ QR)
# --------------------------------------------------------------------------- #
@app.get("/report", response_class=HTMLResponse)
def public_report(request: Request, location: str = "", ok: int = 0, err: int = 0):
    return templates.TemplateResponse(
        request, "public_report.html",
        context(request, location=location, ok=ok, err=err),
    )


@app.post("/report", response_class=HTMLResponse)
def public_report_submit(
    request: Request,
    reporter_name: str = Form(...),
    contact: str = Form(""),
    location: str = Form(""),
    building: str = Form(""),
    unit: str = Form(""),
    section: str = Form(""),
    problem_type: str = Form(""),
    priority: str = Form("Ø¹Ø§Ø¯ÙŠØ©"),
    description: str = Form(""),
    media_files: list[UploadFile] = File(default=[]),
):
    if not reporter_name.strip() or not description.strip():
        return RedirectResponse("/report?err=1", status_code=303)
    if not location.strip():
        location = " / ".join(p for p in [building, unit] if p.strip())
    saved = save_media(media_files)
    _oid, order_no, reference_id = insert_order(
        reporter_name, contact, location, section, priority,
        "", description,
        problem_type=problem_type, building=building, unit=unit,
        media=",".join(saved), status="pending", source="qr",
    )
    return templates.TemplateResponse(
        request, "thanks.html",
        context(request, order_no=order_no, reference_id=reference_id, media=saved),
    )


# --------------------------------------------------------------------------- #
# ØµÙØ­Ø© ØªØªØ¨Ø¹ Ø£Ù…Ø± Ø§Ù„Ø¹Ù…Ù„ (ØªÙØªØ­ Ø¨Ù…Ø³Ø­ QR)
# --------------------------------------------------------------------------- #
@app.get("/track/{token}", response_class=HTMLResponse)
def track(token: str, request: Request):
    with db() as c:
        row = c.execute(
            "SELECT * FROM work_orders WHERE token=?", (token,)
        ).fetchone()
    if row is None:
        return templates.TemplateResponse(request, "404.html", context(request), status_code=404)
    return templates.TemplateResponse(
        request, "track.html",
        context(request, o=row, media=media_list(row["media"])),
    )


# --------------------------------------------------------------------------- #
# Ø®Ø¯Ù…Ø© Ø§Ù„Ù…Ù„ÙØ§Øª Ø§Ù„Ù…Ø±ÙÙˆØ¹Ø© (Ø§Ù„Ù…ÙŠØ¯ÙŠØ§)
# --------------------------------------------------------------------------- #
@app.get("/media/{filename}")
def serve_media(filename: str):
    # Ù…Ù†Ø¹ ØªØ¬Ø§ÙˆØ² Ø§Ù„Ù…Ø³Ø§Ø± (path traversal)
    if "/" in filename or "\\" in filename or ".." in filename:
        raise HTTPException(status_code=404, detail="not found")
    path = UPLOAD_DIR / filename
    if not path.is_file():
        raise HTTPException(status_code=404, detail="not found")
    ext = path.suffix.lower()
    mime = ALLOWED_MEDIA.get(ext, "application/octet-stream")
    return Response(content=path.read_bytes(), media_type=mime)


# --------------------------------------------------------------------------- #
# Ø§Ù„ØªÙ‚Ø±ÙŠØ± Ø§Ù„Ø´Ù‡Ø±ÙŠ Ø§Ù„Ù‚Ø§Ø¨Ù„ Ù„Ù„Ø·Ø¨Ø§Ø¹Ø©
# --------------------------------------------------------------------------- #
@app.get("/monthly", response_class=HTMLResponse)
def monthly_report(request: Request, year: int = 0, month: int = 0):
    t = now_local()
    year = year or t.year
    month = min(max(month or t.month, 1), 12)
    ym = f"{year:04d}-{month:02d}"

    with db() as c:
        rows = c.execute(
            "SELECT * FROM work_orders WHERE substr(created_at,1,7)=? ORDER BY id ASC",
            (ym,),
        ).fetchall()
        years = [
            int(r[0])
            for r in c.execute(
                "SELECT DISTINCT substr(created_at,1,4) AS y FROM work_orders ORDER BY y"
            ).fetchall()
        ]
    if not years or years[-1] < t.year:
        years.append(t.year)

    by_section, by_priority, by_status, by_source = {}, {}, {}, {}
    total_urgent = 0
    for r in rows:
        s = r["section"] or "ØºÙŠØ± Ù…Ø­Ø¯Ø¯"
        by_section[s] = by_section.get(s, 0) + 1
        by_priority[r["priority"]] = by_priority.get(r["priority"], 0) + 1
        by_status[r["status"]] = by_status.get(r["status"], 0) + 1
        by_source[r["source"]] = by_source.get(r["source"], 0) + 1
        if r["priority"] == "Ø·Ø§Ø±Ø¦Ø©":
            total_urgent += 1

    totals = {
        "total": len(rows),
        "done": by_status.get("done", 0),
        "approved": by_status.get("approved", 0),
        "pending": by_status.get("pending", 0),
        "rejected": by_status.get("rejected", 0),
        "urgent": total_urgent,
        "rate": round(by_status.get("done", 0) * 100 / len(rows)) if rows else 0,
    }

    return templates.TemplateResponse(
        request, "monthly.html",
        context(
            request,
            rows=rows, year=year, month=month, years=sorted(years),
            by_section=by_section, by_priority=by_priority,
            by_status=by_status, by_source=by_source, totals=totals,
        ),
    )


# --------------------------------------------------------------------------- #
# Ø­Ø§Ù„Ø© Ø§Ù„Ø®Ø§Ø¯Ù…
# --------------------------------------------------------------------------- #
@app.get("/healthz")
def healthz():
    return {"status": "ok", "time": fmt(now_local()), "orders_db": str(DB_PATH)}


# --------------------------------------------------------------------------- #
# Ù†Ù‚Ø·Ø© Ø§Ù„Ø¯Ø®ÙˆÙ„
# --------------------------------------------------------------------------- #
if __name__ == "__main__":
    port = int(os.environ.get("PORT", "8000"))
    uvicorn.run(
        app,
        host="0.0.0.0",
        port=port,
        proxy_headers=True,
        forwarded_allow_ips="*",
    )
