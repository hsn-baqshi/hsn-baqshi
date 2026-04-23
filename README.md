- 👋 Hi, I’m @hsn-baqshi. I am an engineer, artist and developer
- 👀 I’m interested in Game Development, Data Science, video games, cars, and sciences in general
- 🌱 I’m currently working on an indie game 
- 📫 You can reach me @ albaqshiha@gmail.com

<!---
hsn-baqshi/hsn-baqshi is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->


"""
title: Send only last N tokens to LiteLLM
author: you
version: 0.2
description: Replaces outgoing messages with a single user message = last N tokens of the full chat (serialized).
required_open_webui_version: 0.5.0
"""

from __future__ import annotations

from typing import Any, List, Optional

from pydantic import BaseModel, Field


class Filter:
    class Valves(BaseModel):
        priority: int = Field(
            default=0,
            description="Lower runs earlier in the filter chain.",
        )
        n_tokens: int = Field(
            default=512,
            ge=1,
            description="Max tokens forwarded upstream (tail of full serialized chat).",
        )
        mode: str = Field(
            default="whole_chat_tail",
            description="'whole_chat_tail' = only last n_tokens of entire chat sent; 'last_user_only' = trim last user message only.",
            json_schema_extra={"enum": ["whole_chat_tail", "last_user_only"]},
        )
        use_hf_tokenizer: bool = Field(
            default=True,
            description="Use Hugging Face tokenizer when available.",
        )
        hf_model_id: str = Field(
            default="meta-llama/Llama-3.2-3B-Instruct",
            description="Tokenizer repo (match your model family).",
        )
        chat_separator: str = Field(
            default="\n\n",
            description="Between messages when flattening the chat.",
        )

    def __init__(self) -> None:
        self.valves = self.Valves()
        self._tokenizer = None
        self._tokenizer_failed = False

    def _get_tokenizer(self):
        if self._tokenizer_failed:
            return None
        if self._tokenizer is not None:
            return self._tokenizer
        try:
            from transformers import AutoTokenizer

            self._tokenizer = AutoTokenizer.from_pretrained(self.valves.hf_model_id)
        except Exception:
            self._tokenizer_failed = True
            self._tokenizer = None
        return self._tokenizer

    @staticmethod
    def _message_text_parts(messages: List[dict]) -> List[str]:
        parts: List[str] = []
        for m in messages:
            role = m.get("role") or "unknown"
            content = m.get("content")
            if isinstance(content, str):
                if content.strip():
                    parts.append(f"{role.upper()}: {content}")
            elif isinstance(content, list):
                # Multimodal: pull plain text chunks only
                for block in content:
                    if not isinstance(block, dict):
                        continue
                    if block.get("type") == "text":
                        t = block.get("text") or ""
                        if isinstance(t, str) and t.strip():
                            parts.append(f"{role.upper()}: {t}")
        return parts

    def _serialize_chat(self, messages: List[dict]) -> str:
        parts = self._message_text_parts(messages)
        return self.valves.chat_separator.join(parts)

    def _tail_tokens(self, text: str, n: int) -> str:
        if self.valves.use_hf_tokenizer:
            tok = self._get_tokenizer()
            if tok is not None:
                ids = tok.encode(text, add_special_tokens=False)
                if len(ids) <= n:
                    return text
                return tok.decode(ids[-n:], skip_special_tokens=True)
        # Word fallback (~ not tokens)
        words = text.split()
        if len(words) <= n:
            return text
        return " ".join(words[-n:])

    @staticmethod
    def _last_user_string(messages: List[dict]) -> tuple[Optional[int], Optional[str]]:
        for i in range(len(messages) - 1, -1, -1):
            if messages[i].get("role") != "user":
                continue
            c = messages[i].get("content")
            if isinstance(c, str):
                return i, c
            return i, None
        return None, None

    async def inlet(self, body: dict, __user__: Optional[dict] = None) -> dict:
        msgs: List[dict] = list(body.get("messages") or [])
        n = self.valves.n_tokens

        if self.valves.mode == "last_user_only":
            idx, text = self._last_user_string(msgs)
            if idx is None or text is None:
                return body
            msgs[idx]["content"] = self._tail_tokens(text, n)
            body["messages"] = msgs
            return body

        # whole_chat_tail: single outbound user message = tail of full chat
        flat = self._serialize_chat(msgs)
        if not flat.strip():
            return body
        tail = self._tail_tokens(flat, n)
        body["messages"] = [{"role": "user", "content": tail}]
        return body

    async def stream(self, event: dict) -> dict:
        return event

    async def outlet(self, body: dict) -> dict:
        return body
