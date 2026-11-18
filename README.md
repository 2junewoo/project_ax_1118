# Hermes Agent 세팅 (11.18)

Nous Research의 오픈소스 에이전트 **Hermes Agent**를 맥(Apple M5)에 설치하고 모델을 연결한 기록입니다. Claude Code와 함께 진행했습니다.

## 설치

- 공식 설치 스크립트를 먼저 내려받아 내용을 확인한 뒤 실행
  ```
  curl -fsSL https://hermes-agent.nousresearch.com/install.sh -o install.sh
  bash install.sh --non-interactive
  ```
- 결과: Hermes Agent v0.23.2 (Python 3.14.9), 코드 `~/.hermes/hermes-agent/`, 명령 `~/.local/bin/hermes`, 설정 `~/.hermes/config.yaml`

## 모델 연결

| 단계 | 내용 |
| --- | --- |
| 지금 | ChatGPT 무료 계정 OAuth (`openai-codex` 제공자) → 모델 `gpt-5.6-luna` |
| 나중 | Claude Team 요금제 월 API 크레딧이 열리면(가입 7일 뒤, 11/22쯤) Console 조직 연결 → API 키로 `anthropic` 제공자 전환 |

```
hermes config set auth.adopt_external_logins false   # Claude Code 로그인 자동 사용 끄기
hermes auth add openai-codex --browser                # ChatGPT 로그인 (브라우저 승인)
hermes config set model.provider openai-codex
hermes config set model.default gpt-5.6-luna
```

### 알게 된 점

- **Claude 구독 로그인은 못 씀**: Hermes 문서상 Claude 계정 OAuth는 Max + 추가 사용량 크레딧 전용이고, 기본 사용량이 아닌 추가 크레딧만 소모. Pro는 불가, Team은 언급 없음. Claude Code 로그인을 자동으로 가져다 쓰면 의도치 않게 추가 요금이 나갈 수 있어 꺼 둠.
- **Team 요금제 API 크레딧**: Max·Team 구독에 월 API 크레딧이 붙음(Claude Code·앱 추가 사용량에는 못 씀, API·Agent SDK용). 다 쓰면 멈출 뿐 추가 청구 없음 → Hermes에 적합.
- **ChatGPT 기기 코드 로그인**은 ChatGPT 보안 설정에서 따로 켜야 함 → `--browser`(PKCE) 방식으로 우회.
- **무료 계정 모델 제한**: `gpt-5.5`, `gpt-5.4-mini` 등은 HTTP 400 "not supported when using Codex with a ChatGPT account". `gpt-5.6-luna`는 됨.

## 확인

```
hermes chat -q "현재 폴더의 파일 목록을 터미널 도구로 확인하고 한 줄로 알려줘." --oneshot
```
→ 터미널 도구를 호출해 `docs, sessions`를 답함 (도구 호출 정상)

## 텔레그램 연결

1. @BotFather에서 `/newbot`으로 봇 생성, @userinfobot으로 내 텔레그램 ID 확인
2. `~/.hermes/.env`에 `TELEGRAM_BOT_TOKEN`, `TELEGRAM_ALLOWED_USERS`(내 ID만) 설정 — 파일 권한 600
3. 게이트웨이를 launchd 서비스로 설치 (로그인 시 자동 시작·죽으면 재시작)
   ```
   hermes gateway install --start-now --start-on-login
   hermes gateway status
   ```
4. 휴대폰에서 봇에게 "안녕" → 4.5초 만에 답장 (`~/.hermes/logs/gateway.log`에서 inbound/response 확인)

- 토큰을 채팅에 붙여 넣는 실수 → BotFather `/revoke`로 새 토큰 발급 후 사용
- `hermes gateway setup`은 입력 화면이 필요해 Claude Code의 `!`로는 플랫폼 선택이 안 됨 → `.env` 직접 설정
- 위험한 명령은 approvals `smart` 모드라 텔레그램으로 승인 요청이 옴
- 맥이 잠자기 중이면 응답 없음 (밀린 메시지는 깨어나면 처리, 최대 24시간)

## 파일

| 경로 | 내용 |
| --- | --- |
| `docs/` | Claude Code 작업 대화 기록 (개인정보 가림) |
| `sessions/` | 대화 기록 원본 (git 제외) |
