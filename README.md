# fizz-twitch-chat-source

**Fizz** コメント入力系部品 — Twitch チャットを匿名購読して正規化コメント
([fizz-protocol](https://github.com/aiviecast/fizz-protocol) の `Comment`) を
NDJSON で stdout に流す。

責務は一行: **Twitch IRC → comment ストリーム**。
匿名 nick (`justinfan{N}`) の読み取り専用 — OAuth 不要、チャット送信なし。
dedupe は [fizz-comment-dedupe](https://github.com/aiviecast/fizz-comment-dedupe) の責務。

## Usage

```bash
almide build src/main.almd -o fizz-twitch-chat-source

TWITCH_CHANNEL=somechannel ./fizz-twitch-chat-source \
  | fizz-comment-dedupe \
  | ...
```

stdout: 1 行 1 コメント (Codec JSON)。ログ・診断はすべて stderr。

## 挙動

- `irc.chat.twitch.tv:6667` (平文 IRC) に接続。旧 openaituber は
  wss:443 だったが、匿名・読み取り専用なので平文で等価
- `CAP REQ :twitch.tv/tags twitch.tv/commands` で display-name / color /
  id / RECONNECT を受ける
- PRIVMSG → `Comment` (id は Twitch メッセージ id、無ければ時刻+連番 fallback)
- PING → PONG keepalive (~5 分ごと)
- RECONNECT / EOF / read error → 5s 後に再接続

## Env

| var | 意味 |
|---|---|
| `TWITCH_CHANNEL` | 購読するチャンネル名 (`#` 有無どちらでも) |

## Tests

```bash
almide test
```

IRC 行パース / タグ unescape / PRIVMSG 正規化 / PING-PONG / バッファ行分解を
ネットワークなしでテストする。
