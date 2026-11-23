import asyncio
import json
import random
import sqlite3
import string
import time
import urllib.parse
from datetime import datetime
from typing import Dict, List, Optional

import httpx
from fastapi import FastAPI, HTTPException, Request, WebSocket, WebSocketDisconnect
from fastapi.responses import HTMLResponse, RedirectResponse
from fastapi.templating import Jinja2Templates
from pydantic import BaseModel

# ==============================================================================
# CONFIGURATION — FILL IN YOUR DISCORD CREDENTIALS & ROLE IDs HERE
# ==============================================================================
DISCORD_CLIENT_ID = "1574219654950551673"
DISCORD_CLIENT_SECRET = "UIfuUFkhhEHUi5ZaZQHxG4oVt2uiqBtr"
DISCORD_BOT_TOKEN = "MTU3NDIxOTY1NDk1MDU1MTY3Mw.GoFol6.OPAd4TanMUvbnZkM4K0w7RZ7Rdwk0NRVJxrDgQ"
DISCORD_REDIRECT_URI = "http://127.0.0.1:8000/auth/callback"

GUILD_ID = "1456324611951296606"  # Target Discord Server ID

# Discord Role IDs (Strings)
# Discord Role IDs (Lists of individual role ID strings)
LOW_RANK_ROLE_IDS = [
    "1456325604076032011",
    "1456325603908124775",
    "1456325602775798014",
    "1456325601941000272",
    "1456325600812728535",
]  # Can view & query investigations

MID_RANK_ROLE_IDS = [
    "1456325599466356920",
    "1456325579765714985",
]  # Can log & edit own investigations

HIGH_RANK_ROLE_IDS = [
    "1457031733810561035",
]  # Skips Agent ID, manages IDs, edits all cases        # Skips Agent ID, manages IDs, edits all cases

DB_PATH = "lapd_ia_database.db"
# ==============================================================================

app = FastAPI(title="LAPD | Internal Affairs System")
templates = Jinja2Templates(directory="templates")


# ------------------------------------------------------------------------------
# DATABASE SETUP (SQLite Persistent Storage)
# ------------------------------------------------------------------------------
def get_db():
    conn = sqlite3.connect(DB_PATH, check_same_thread=False)
    conn.row_factory = sqlite3.Row
    return conn


def init_db():
    conn = get_db()
    c = conn.cursor()

    # Agent IDs Table
    c.execute("""
        CREATE TABLE IF NOT EXISTS agent_ids (
            agent_id TEXT PRIMARY KEY,
            assigned_discord_id TEXT,
            assigned_username TEXT,
            expires_at INTEGER,
            revoked INTEGER DEFAULT 0,
            created_by TEXT,
            created_at TEXT
        )
    """)

    # User Sessions Table (Persistent until revoked)
    c.execute("""
        CREATE TABLE IF NOT EXISTS sessions (
            session_token TEXT PRIMARY KEY,
            discord_id TEXT,
            username TEXT,
            avatar_url TEXT,
            rank_level TEXT,
            agent_id TEXT,
            verified INTEGER DEFAULT 0
        )
    """)

    # Investigations Table
    c.execute("""
        CREATE TABLE IF NOT EXISTS investigations (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            case_number TEXT UNIQUE,
            accused_username TEXT NOT NULL,
            charges TEXT NOT NULL,
            status_tag TEXT NOT NULL,
            pages_json TEXT NOT NULL,
            author_discord_id TEXT NOT NULL,
            author_username TEXT NOT NULL,
            created_at TEXT NOT NULL,
            updated_at TEXT NOT NULL
        )
    """)

    # Seed one master starter Agent ID if table is empty for initial testing
    c.execute("SELECT COUNT(*) as cnt FROM agent_ids")
    if c.fetchone()["cnt"] == 0:
        c.execute(
            """
            INSERT INTO agent_ids (agent_id, expires_at, revoked, created_by, created_at)
            VALUES (?, ?, 0, 'SYSTEM_INIT', ?)
            """,
            ("IA-10001", int(time.time()) + 86400 * 365, datetime.utcnow().isoformat()),
        )

    conn.commit()
    conn.close()


init_db()


# ------------------------------------------------------------------------------
# WEBSOCKET CONNECTION MANAGER (Real-Time Live Updates & Instant Kick)
# ------------------------------------------------------------------------------
class ConnectionManager:
    def __init__(self):
        # Maps session_token -> WebSocket
        self.active_connections: Dict[str, WebSocket] = {}

    async def connect(self, session_token: str, websocket: WebSocket):
        await websocket.accept()
        self.active_connections[session_token] = websocket

    def disconnect(self, session_token: str):
        if session_token in self.active_connections:
            del self.active_connections[session_token]

    async def broadcast(self, message: dict):
        dead_tokens = []
        for token, ws in self.active_connections.items():
            try:
                await ws.send_json(message)
            except Exception:
                dead_tokens.append(token)
        for t in dead_tokens:
            self.disconnect(t)

    async def send_to_session(self, session_token: str, message: dict):
        ws = self.active_connections.get(session_token)
        if ws:
            try:
                await ws.send_json(message)
            except Exception:
                self.disconnect(session_token)


manager = ConnectionManager()


# ------------------------------------------------------------------------------
# PYDANTIC MODELS
# ------------------------------------------------------------------------------
class VerifyAgentRequest(BaseModel):
    session_token: str
    agent_id: str


class InvestigationCreate(BaseModel):
    session_token: str
    accused_username: str
    charges: str
    status_tag: str
    pages: List[dict]


class InvestigationUpdate(BaseModel):
    session_token: str
    accused_username: str
    charges: str
    status_tag: str
    pages: List[dict]


class GenerateAgentIDRequest(BaseModel):
    session_token: str
    valid_days: int = 30


class RevokeAgentIDRequest(BaseModel):
    session_token: str
    agent_id: str


# ------------------------------------------------------------------------------
# HELPER FUNCTIONS
# ------------------------------------------------------------------------------
def get_session(token: str):
    if not token:
        return None
    conn = get_db()
    c = conn.cursor()
    c.execute("SELECT * FROM sessions WHERE session_token = ?", (token,))
    row = c.fetchone()
    conn.close()
    return dict(row) if row else None


def generate_token(length=40):
    return "".join(random.choices(string.ascii_letters + string.digits, k=length))


def generate_ia_code():
    digits = "".join(random.choices(string.digits, k=5))
    return f"IA-{digits}"


# ------------------------------------------------------------------------------
# ROUTES: FRONTEND & DISCORD OAUTH2 + BOT VERIFICATION
# ------------------------------------------------------------------------------
@app.get("/", response_class=HTMLResponse)
async def serve_frontend(request: Request):
    return templates.TemplateResponse(request=request, name="index.html")


@app.get("/auth/discord")
async def discord_login():
    params = {
        "client_id": DISCORD_CLIENT_ID,
        "redirect_uri": DISCORD_REDIRECT_URI,
        "response_type": "code",
        "scope": "identify guilds",
    }
    url = f"https://discord.com/api/oauth2/authorize?{urllib.parse.urlencode(params)}"
    return RedirectResponse(url)


@app.get("/auth/callback")
async def discord_callback(code: str):
    async with httpx.AsyncClient() as client:
        # 1. Exchange code for user access token
        token_resp = await client.post(
            "https://discord.com/api/oauth2/token",
            data={
                "client_id": DISCORD_CLIENT_ID,
                "client_secret": DISCORD_CLIENT_SECRET,
                "grant_type": "authorization_code",
                "code": code,
                "redirect_uri": DISCORD_REDIRECT_URI,
            },
            headers={"Content-Type": "application/x-www-form-urlencoded"},
        )
        if token_resp.status_code != 200:
            return RedirectResponse("/?auth_error=oauth_failed")

        access_token = token_resp.json().get("access_token")

        # 2. Fetch Discord User Identity
        user_resp = await client.get(
            "https://discord.com/api/users/@me",
            headers={"Authorization": f"Bearer {access_token}"},
        )
        user_data = user_resp.json()
        discord_id = user_data["id"]
        username = user_data.get("global_name") or user_data["username"]
        avatar_hash = user_data.get("avatar")
        avatar_url = (
            f"https://cdn.discordapp.com/avatars/{discord_id}/{avatar_hash}.png"
            if avatar_hash
            else "https://cdn.discordapp.com/embed/avatars/0.png"
        )

        # 3. Use DISCORD BOT API to check if user is in target Guild & inspect Roles
        member_resp = await client.get(
            f"https://discord.com/api/v10/guilds/{GUILD_ID}/members/{discord_id}",
            headers={"Authorization": f"Bot {DISCORD_BOT_TOKEN}"},
        )

        # If user is not inside the specified Discord Server
        if member_resp.status_code != 200:
            return RedirectResponse("/?auth_error=not_in_server")

        member_data = member_resp.json()
        roles = member_data.get("roles", [])

        # Determine Rank Hierarchy
        user_role_set = {str(r).strip() for r in roles}
        high_set = {str(r).strip() for r in HIGH_RANK_ROLE_IDS}
        mid_set = {str(r).strip() for r in MID_RANK_ROLE_IDS}
        low_set = {str(r).strip() for r in LOW_RANK_ROLE_IDS}

        if user_role_set & high_set:
            rank_level = "HIGH"
        elif user_role_set & mid_set:
            rank_level = "MID"
        elif user_role_set & low_set:
            rank_level = "LOW"
        else:
            print("[AUTH DEBUG] User is in guild, but NONE of the configured rank role IDs matched.")
            return RedirectResponse("/?auth_error=not_in_server")

        # Create persistent session
        session_token = generate_token()
        # High Ranks bypass Agent ID check automatically
        verified = 1 if rank_level == "HIGH" else 0
        agent_id = "COMMAND-EXEMPT" if rank_level == "HIGH" else None

        conn = get_db()
        c = conn.cursor()
        c.execute(
            """
            INSERT INTO sessions (session_token, discord_id, username, avatar_url, rank_level, agent_id, verified)
            VALUES (?, ?, ?, ?, ?, ?, ?)
            """,
            (session_token, discord_id, username, avatar_url, rank_level, agent_id, verified),
        )
        conn.commit()
        conn.close()

        return RedirectResponse(f"/?token={session_token}")


# ------------------------------------------------------------------------------
# API: SESSION & AGENT ID VERIFICATION
# ------------------------------------------------------------------------------
@app.get("/api/session/{session_token}")
async def check_session(session_token: str):
    sess = get_session(session_token)
    if not sess:
        return {"valid": False, "revoked": False}

    # Check if user's associated Agent ID was revoked or expired
    if sess["rank_level"] != "HIGH" and sess["verified"] == 1 and sess["agent_id"]:
        conn = get_db()
        c = conn.cursor()
        c.execute("SELECT * FROM agent_ids WHERE agent_id = ?", (sess["agent_id"],))
        row = c.fetchone()
        if not row or row["revoked"] == 1 or row["expires_at"] < int(time.time()):
            c.execute("DELETE FROM sessions WHERE session_token = ?", (session_token,))
            conn.commit()
            conn.close()
            return {"valid": False, "revoked": True}
        conn.close()

    return {"valid": True, "session": sess}


@app.post("/api/verify-agent")
async def verify_agent_id(payload: VerifyAgentRequest):
    sess = get_session(payload.session_token)
    if not sess:
        raise HTTPException(status_code=401, detail="Invalid session.")

    code = payload.agent_id.strip().upper()
    if not code.startswith("IA-") or len(code) != 8:
        raise HTTPException(status_code=400, detail="Invalid Agent ID format. Expected IA-XXXXX.")

    conn = get_db()
    c = conn.cursor()
    c.execute("SELECT * FROM agent_ids WHERE agent_id = ?", (code,))
    record = c.fetchone()

    if not record:
        conn.close()
        raise HTTPException(status_code=404, detail="Agent ID does not exist in IA registry.")

    if record["revoked"] == 1:
        conn.close()
        raise HTTPException(status_code=403, detail="This Agent ID has been permanently revoked.")

    if record["expires_at"] < int(time.time()):
        conn.close()
        raise HTTPException(status_code=403, detail="This Agent ID has expired.")

    # Bind Agent ID to this user if unassigned, or verify ownership
    if record["assigned_discord_id"] and record["assigned_discord_id"] != sess["discord_id"]:
        conn.close()
        raise HTTPException(status_code=403, detail="Agent ID is already bound to another officer.")

    c.execute(
        """
        UPDATE agent_ids
        SET assigned_discord_id = ?, assigned_username = ?
        WHERE agent_id = ?
        """,
        (sess["discord_id"], sess["username"], code),
    )

    c.execute(
        """
        UPDATE sessions
        SET verified = 1, agent_id = ?
        WHERE session_token = ?
        """,
        (code, payload.session_token),
    )
    conn.commit()
    conn.close()

    updated_sess = get_session(payload.session_token)
    return {"status": "verified", "session": updated_sess}


# ------------------------------------------------------------------------------
# API: INVESTIGATIONS (CRUD + LIVE BROADCAST)
# ------------------------------------------------------------------------------
@app.get("/api/investigations")
async def list_investigations(session_token: str):
    sess = get_session(session_token)
    if not sess or sess["verified"] != 1:
        raise HTTPException(status_code=401, detail="Unauthorized.")

    conn = get_db()
    c = conn.cursor()
    c.execute("SELECT * FROM investigations ORDER BY id DESC")
    rows = [dict(r) for r in c.fetchall()]
    conn.close()

    for r in rows:
        r["pages"] = json.loads(r["pages_json"])
        del r["pages_json"]
    return {"investigations": rows}


@app.post("/api/investigations")
async def create_investigation(payload: InvestigationCreate):
    sess = get_session(payload.session_token)
    if not sess or sess["verified"] != 1:
        raise HTTPException(status_code=401, detail="Unauthorized.")

    if sess["rank_level"] not in ("MID", "HIGH"):
        raise HTTPException(status_code=403, detail="Insufficient rank to log investigations.")

    if not payload.accused_username.strip() or not payload.charges.strip() or not payload.status_tag.strip():
        raise HTTPException(status_code=400, detail="All fields are strictly required.")

    if not payload.pages or len(payload.pages) == 0:
        raise HTTPException(status_code=400, detail="At least one documentation page is required.")

    case_num = f"CASE-{datetime.utcnow().strftime('%y%m')}-{random.randint(1000, 9999)}"
    now_iso = datetime.utcnow().strftime("%Y-%m-%d %H:%M UTC")

    conn = get_db()
    c = conn.cursor()
    c.execute(
        """
        INSERT INTO investigations (
            case_number, accused_username, charges, status_tag, pages_json,
            author_discord_id, author_username, created_at, updated_at
        ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
        """,
        (
            case_num,
            payload.accused_username.strip(),
            payload.charges.strip(),
            payload.status_tag.strip(),
            json.dumps(payload.pages),
            sess["discord_id"],
            sess["username"],
            now_iso,
            now_iso,
        ),
    )
    new_id = c.lastrowid
    conn.commit()
    c.execute("SELECT * FROM investigations WHERE id = ?", (new_id,))
    created = dict(c.fetchone())
    conn.close()

    created["pages"] = json.loads(created["pages_json"])
    del created["pages_json"]

    # Broadcast live to all connected clients
    await manager.broadcast({"type": "INVESTIGATION_CREATED", "data": created})
    return {"status": "created", "investigation": created}


@app.put("/api/investigations/{inv_id}")
async def update_investigation(inv_id: int, payload: InvestigationUpdate):
    sess = get_session(payload.session_token)
    if not sess or sess["verified"] != 1:
        raise HTTPException(status_code=401, detail="Unauthorized.")

    conn = get_db()
    c = conn.cursor()
    c.execute("SELECT * FROM investigations WHERE id = ?", (inv_id,))
    existing = c.fetchone()
    if not existing:
        conn.close()
        raise HTTPException(status_code=404, detail="Investigation not found.")

    # Permission check: High Rank can edit any; Mid Rank can edit ONLY their own
    is_owner = existing["author_discord_id"] == sess["discord_id"]
    is_high_rank = sess["rank_level"] == "HIGH"

    if not (is_high_rank or (sess["rank_level"] == "MID" and is_owner)):
        conn.close()
        raise HTTPException(status_code=403, detail="You do not have permission to edit this case.")

    now_iso = datetime.utcnow().strftime("%Y-%m-%d %H:%M UTC")
    c.execute(
        """
        UPDATE investigations
        SET accused_username = ?, charges = ?, status_tag = ?, pages_json = ?, updated_at = ?
        WHERE id = ?
        """,
        (
            payload.accused_username.strip(),
            payload.charges.strip(),
            payload.status_tag.strip(),
            json.dumps(payload.pages),
            now_iso,
            inv_id,
        ),
    )
    conn.commit()
    c.execute("SELECT * FROM investigations WHERE id = ?", (inv_id,))
    updated = dict(c.fetchone())
    conn.close()

    updated["pages"] = json.loads(updated["pages_json"])
    del updated["pages_json"]

    # Broadcast live update
    await manager.broadcast({"type": "INVESTIGATION_UPDATED", "data": updated})
    return {"status": "updated", "investigation": updated}


# ------------------------------------------------------------------------------
# API: HIGH RANK COMMAND PANEL (GENERATE & REVOKE AGENT IDs)
# ------------------------------------------------------------------------------
@app.get("/api/admin/agent-ids")
async def get_all_agent_ids(session_token: str):
    sess = get_session(session_token)
    if not sess or sess["verified"] != 1 or sess["rank_level"] != "HIGH":
        raise HTTPException(status_code=403, detail="High Rank clearance required.")

    conn = get_db()
    c = conn.cursor()
    c.execute("SELECT * FROM agent_ids ORDER BY created_at DESC")
    rows = [dict(r) for r in c.fetchall()]
    conn.close()
    return {"agent_ids": rows}


@app.post("/api/admin/generate-id")
async def generate_new_agent_id(payload: GenerateAgentIDRequest):
    sess = get_session(payload.session_token)
    if not sess or sess["verified"] != 1 or sess["rank_level"] != "HIGH":
        raise HTTPException(status_code=403, detail="High Rank clearance required.")

    conn = get_db()
    c = conn.cursor()
    while True:
        new_code = generate_ia_code()
        c.execute("SELECT 1 FROM agent_ids WHERE agent_id = ?", (new_code,))
        if not c.fetchone():
            break

    expires_at = int(time.time()) + (payload.valid_days * 86400)
    created_at = datetime.utcnow().strftime("%Y-%m-%d %H:%M UTC")

    c.execute(
        """
        INSERT INTO agent_ids (agent_id, expires_at, revoked, created_by, created_at)
        VALUES (?, ?, 0, ?, ?)
        """,
        (new_code, expires_at, sess["username"], created_at),
    )
    conn.commit()
    c.execute("SELECT * FROM agent_ids WHERE agent_id = ?", (new_code,))
    record = dict(c.fetchone())
    conn.close()

    await manager.broadcast({"type": "AGENT_ID_LIST_UPDATED"})
    return {"status": "generated", "record": record}


@app.post("/api/admin/revoke-id")
async def revoke_agent_id(payload: RevokeAgentIDRequest):
    sess = get_session(payload.session_token)
    if not sess or sess["verified"] != 1 or sess["rank_level"] != "HIGH":
        raise HTTPException(status_code=403, detail="High Rank clearance required.")

    conn = get_db()
    c = conn.cursor()
    c.execute("UPDATE agent_ids SET revoked = 1 WHERE agent_id = ?", (payload.agent_id,))

    # Find all active sessions tied to this Agent ID and terminate them immediately
    c.execute("SELECT session_token FROM sessions WHERE agent_id = ?", (payload.agent_id,))
    affected_sessions = [r["session_token"] for r in c.fetchall()]
    c.execute("DELETE FROM sessions WHERE agent_id = ?", (payload.agent_id,))
    conn.commit()
    conn.close()

    # Immediately push revocation notice to affected user's browser via WebSocket
    for tok in affected_sessions:
        await manager.send_to_session(
            tok,
            {
                "type": "ACCESS_REVOKED",
                "message": "Your access has been revoked",
            },
        )

    await manager.broadcast({"type": "AGENT_ID_LIST_UPDATED"})
    return {"status": "revoked", "agent_id": payload.agent_id}


# ------------------------------------------------------------------------------
# WEBSOCKET ENDPOINT
# ------------------------------------------------------------------------------
@app.websocket("/ws/{session_token}")
async def websocket_endpoint(websocket: WebSocket, session_token: str):
    await manager.connect(session_token, websocket)
    try:
        while True:
            await websocket.receive_text()
    except WebSocketDisconnect:
        manager.disconnect(session_token)


if __name__ == "__main__":
    import uvicorn

    uvicorn.run("main:app", host="127.0.0.1", port=8000, reload=True)
