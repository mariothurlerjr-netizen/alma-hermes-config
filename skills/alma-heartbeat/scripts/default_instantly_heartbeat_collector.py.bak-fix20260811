#!/usr/bin/env python3
"""Secret-safe live Instantly/Postgres collector for default heartbeat.

Prints aggregate JSON only. It does not print env values or raw API payloads.
Run from any cwd on Mario's VPS:

    python ~/.hermes/skills/alma-heartbeat/scripts/default_instantly_heartbeat_collector.py
"""
from __future__ import annotations

import datetime as dt
import json
import os
import statistics
import subprocess
from pathlib import Path
from urllib import parse, request

AGENTIC_ENV = Path("/home/almarev/agentic/.env")
BASE = "https://api.instantly.ai/api/v2"
SPRINT_START = "2026-07-02"
WORKSPACE_ID = "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"


def load_env(path: Path = AGENTIC_ENV) -> None:
    if not path.exists():
        return
    for raw in path.read_text().splitlines():
        line = raw.strip()
        if not line or line.startswith("#") or "=" not in line:
            continue
        key, value = line.split("=", 1)
        os.environ.setdefault(key, value.strip().strip('"').strip("'"))


def brt_date(fmt: str) -> str:
    return subprocess.check_output(
        ["bash", "-lc", f"TZ=America/Sao_Paulo date {fmt}"], text=True
    ).strip()


def instantly(path: str, params: dict | None = None):
    key = os.environ.get("INSTANTLY_API_KEY")
    if not key:
        raise RuntimeError("INSTANTLY_API_KEY missing")
    url = BASE + path
    if params:
        url += "?" + parse.urlencode(params)
    req = request.Request(
        url,
        headers={
            "Authorization": f"Bearer {key}",
            "Content-Type": "application/json",
            "User-Agent": "curl/8.5.0",
        },
    )
    with request.urlopen(req, timeout=45) as resp:  # noqa: S310, trusted endpoint constant
        return json.loads(resp.read().decode())


def items(payload):
    if isinstance(payload, dict):
        return payload.get("items") or payload.get("data") or payload.get("results") or []
    return payload if isinstance(payload, list) else []


def campaign_name(row: dict) -> str:
    return row.get("campaign_name") or row.get("name") or ""


def bucket(name: str) -> str:
    low = name.lower()
    if name.startswith("ALMAREV-V3-"):
        return "rev_v3"
    if "almalocal" in low or "local" in low:
        return "local"
    if "alma" in low or "rev" in low:
        return "rev_legacy"
    return "other"


def summarize_analytics(rows: list[dict]) -> dict:
    out = {
        b: {"sends": 0, "replies": 0, "bounces": 0, "spam": 0, "campaigns": {}}
        for b in ["rev_v3", "rev_legacy", "local", "other", "total"]
    }
    for row in rows:
        name = campaign_name(row)
        b = bucket(name)
        sends = int(row.get("emails_sent_count") or row.get("sent") or 0)
        replies = int(row.get("reply_count") or row.get("replies") or 0)
        bounces = int(row.get("bounced_count") or row.get("bounces") or 0)
        spam = int(
            row.get("spam_count")
            or row.get("complaint_count")
            or row.get("spam_complaint_count")
            or 0
        )
        for key in (b, "total"):
            out[key]["sends"] += sends
            out[key]["replies"] += replies
            out[key]["bounces"] += bounces
            out[key]["spam"] += spam
        if sends or replies or bounces or spam:
            out[b]["campaigns"][name] = {
                "sends": sends,
                "replies": replies,
                "bounces": bounces,
                "spam": spam,
            }
    return out


def psql_scalar(query: str) -> str:
    pg = os.environ.get("POSTGRES_URL") or os.environ.get("DATABASE_URL")
    if not pg:
        pg = "postgresql://alma_admin:AlmAa10s2026@127.0.0.1:5432/alma_agentic"
    return subprocess.check_output(
        ["psql", pg, "-t", "-A", "-F", "|", "-c", query],
        stderr=subprocess.DEVNULL,
        text=True,
        timeout=20,
    ).strip()


def collect() -> dict:
    load_env()
    today = brt_date("+%F")
    collected_at = brt_date("-Is")

    campaigns = items(instantly("/campaigns", {"limit": 100}))
    accounts = items(instantly("/accounts", {"limit": 100}))
    analytics_today = items(instantly("/campaigns/analytics", {"start_date": today, "end_date": today}))
    analytics_sprint = items(
        instantly("/campaigns/analytics", {"start_date": SPRINT_START, "end_date": today})
    )

    active_campaigns = []
    status_counts: dict[str, int] = {}
    for campaign in campaigns:
        status = campaign.get("status")
        status_counts[str(status)] = status_counts.get(str(status), 0) + 1
        if status == 1 or str(status).lower() in {"active", "running"}:
            active_campaigns.append(
                {
                    "name": campaign.get("name") or campaign.get("campaign_name"),
                    "status": status,
                    "daily_limit": campaign.get("daily_limit")
                    or campaign.get("daily_send_limit")
                    or campaign.get("limit"),
                }
            )

    scores = []
    for account in accounts:
        score = account.get("stat_warmup_score")
        if score is None:
            score = account.get("warmup_score")
        if score is not None:
            try:
                scores.append(float(score))
            except (TypeError, ValueError):
                pass

    pg_queries = {
        "lance_replies_interested": f"""
            SELECT count(*), count(*) FILTER (WHERE classification='interested')
            FROM lance_replies
            WHERE workspace_id='{WORKSPACE_ID}' AND received_at >= '{SPRINT_START}'
        """,
        "deals_meetings": f"""
            SELECT count(*) FROM deals
            WHERE workspace_id='{WORKSPACE_ID}'
              AND calendly_event_uri IS NOT NULL
              AND created_at >= '{SPRINT_START}'
        """,
        "claire_meetings": f"""
            SELECT count(*) FROM claire_meetings
            WHERE workspace_id='{WORKSPACE_ID}' AND meeting_at >= '{SPRINT_START}'
        """,
        "discoveries": f"""
            SELECT count(*) FROM discovery_notes
            WHERE workspace_id='{WORKSPACE_ID}'
              AND counts_as_discovery
              AND created_at >= '{SPRINT_START}'
        """,
        "aura_visits_email": """
            SELECT count(*),
                   count(*) FILTER (WHERE utm_medium='email' OR utm_source ILIKE '%instantly%')
            FROM web_events
            WHERE path ILIKE '%aura%' AND ts > now()-interval '7 days'
        """,
    }
    pg = {}
    for key, query in pg_queries.items():
        try:
            pg[key] = psql_scalar(query)
        except Exception as exc:  # noqa: BLE001
            pg[key] = "ERROR:" + str(exc)[:120]

    return {
        "collected_at_brt": collected_at,
        "today_brt": today,
        "campaigns_total": len(campaigns),
        "campaigns_active": len(active_campaigns),
        "campaign_status_counts": status_counts,
        "active_campaigns": active_campaigns,
        "accounts": {
            "total": len(accounts),
            "connected": sum(
                1
                for account in accounts
                if account.get("status") == 1 or str(account.get("status")).lower() == "connected"
            ),
            "setup_pending": sum(1 for account in accounts if account.get("setup_pending") is True),
            "warmup_active": sum(
                1
                for account in accounts
                if account.get("warmup_status") == 1 or account.get("warmup_enabled") is True
            ),
            "warmup_score": {
                "min": min(scores) if scores else None,
                "avg": statistics.fmean(scores) if scores else None,
                "max": max(scores) if scores else None,
            },
        },
        "today": summarize_analytics(analytics_today),
        "sprint": summarize_analytics(analytics_sprint),
        "postgres": pg,
        "gates": {
            "seed_tests_json_present": Path("/home/almarev/agentic/logs/seed-tests.json").exists(),
            "onda1_preauth_json_present": Path("/home/almarev/agentic/logs/onda1-preauth.json").exists(),
        },
    }


if __name__ == "__main__":
    print(json.dumps(collect(), ensure_ascii=False, indent=2))
