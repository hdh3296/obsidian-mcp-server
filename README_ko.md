# Obsidian MCP Server

[![TypeScript](https://img.shields.io/badge/TypeScript-^5.8.3-blue.svg)](https://www.typescriptlang.org/)
[![Model Context Protocol](https://img.shields.io/badge/MCP%20SDK-^1.13.0-green.svg)](https://modelcontextprotocol.io/)
[![Version](https://img.shields.io/badge/Version-2.0.7-blue.svg)](./CHANGELOG.md)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Status](https://img.shields.io/badge/Status-Production-brightgreen.svg)](https://github.com/cyanheads/obsidian-mcp-server/issues)
[![GitHub](https://img.shields.io/github/stars/cyanheads/obsidian-mcp-server?style=social)](https://github.com/cyanheads/obsidian-mcp-server)

**AI 에이전트와 개발 도구에 원활한 Obsidian 통합 기능을 제공하세요!**

Obsidian vault에 대한 포괄적인 접근을 제공하는 MCP(Model Context Protocol) 서버입니다. [Obsidian Local REST API 플러그인](https://github.com/coddingtonbear/obsidian-local-rest-api)을 통해 LLM과 AI 에이전트가 노트와 파일을 읽고, 쓰고, 검색하고, 관리할 수 있게 해줍니다.

[`cyanheads/mcp-ts-template`](https://github.com/cyanheads/mcp-ts-template)을 기반으로 구축된 이 서버는 강력한 오류 처리, 로깅, 보안 기능을 갖춘 모듈화된 아키텍처를 따릅니다.

## 🚀 핵심 기능: Obsidian 도구 🛠️

이 서버는 AI가 Obsidian vault와 상호작용할 수 있는 전문 도구들을 제공합니다:

| 도구 이름                                                                              | 설명                                                     | 주요 기능                                                                                                                                           |
| :------------------------------------------------------------------------------------- | :------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`obsidian_read_note`](./src/mcp-server/tools/obsidianReadNoteTool/)                   | 지정된 노트의 내용과 메타데이터를 가져옵니다.                     | - `markdown` 또는 `json` 형식으로 읽기.<br/>- 대소문자 구분 없는 경로 폴백.<br/>- 파일 통계 포함 (생성/수정 시간).                              |
| [`obsidian_update_note`](./src/mcp-server/tools/obsidianUpdateNoteTool/)               | 전체 파일 작업을 사용하여 노트를 수정합니다.                      | - 콘텐츠 `append`, `prepend`, 또는 `overwrite`.<br/>- 파일이 존재하지 않으면 생성 가능.<br/>- 경로, 활성 노트, 또는 주기적 노트로 대상 지정. |
| [`obsidian_search_replace`](./src/mcp-server/tools/obsidianSearchReplaceTool/)         | 대상 노트 내에서 검색 및 바꾸기 작업을 수행합니다.                | - 문자열 또는 정규식 검색 지원.<br/>- 대소문자 구분, 전체 단어, 모든 항목 바꾸기 옵션.                                                       |
| [`obsidian_global_search`](./src/mcp-server/tools/obsidianGlobalSearchTool/)           | 전체 vault에서 검색을 수행합니다.                               | - 텍스트 또는 정규식 검색.<br/>- 경로 및 수정 날짜로 필터링.<br/>- 페이지네이션 결과.                                                         |
| [`obsidian_list_notes`](./src/mcp-server/tools/obsidianListNotesTool/)                 | 지정된 vault 폴더 내의 노트와 하위 디렉터리를 나열합니다.          | - 파일 확장자 또는 이름 정규식으로 필터링.<br/>- 디렉터리의 형식화된 트리 뷰 제공.                                                            |
| [`obsidian_manage_frontmatter`](./src/mcp-server/tools/obsidianManageFrontmatterTool/) | 노트의 YAML frontmatter를 원자적으로 관리합니다.                 | - frontmatter 키 `get`, `set`, 또는 `delete`.<br/>- 메타데이터 변경을 위해 전체 파일 재작성 방지.                                            |
| [`obsidian_manage_tags`](./src/mcp-server/tools/obsidianManageTagsTool/)               | 노트의 태그를 추가, 제거, 또는 나열합니다.                        | - YAML frontmatter와 인라인 콘텐츠 모두에서 태그 관리.                                                                                          |
| [`obsidian_delete_note`](./src/mcp-server/tools/obsidianDeleteNoteTool/)               | vault에서 지정된 노트를 영구적으로 삭제합니다.                    | - 안전을 위한 대소문자 구분 없는 경로 폴백.                                                                                                     |

---

## 목차

| [개요](#개요) | [기능](#기능) | [구성](#구성) |
| [프로젝트 구조](#프로젝트-구조) | [Vault 캐시 서비스](#vault-캐시-서비스) |
| [도구](#도구) | [리소스](#리소스) | [개발](#개발) | [라이선스](#라이선스) |

## 개요

Obsidian MCP Server는 Model Context Protocol(MCP)을 이해하는 애플리케이션(MCP 클라이언트) - 고급 AI 어시스턴트(LLM), IDE 확장, 또는 사용자 정의 스크립트 같은 - 이 Obsidian vault와 직접적이고 안전하게 상호작용할 수 있도록 하는 브리지 역할을 합니다.

복잡한 스크립팅이나 수동 상호작용 대신, 도구들은 이 서버를 활용하여:

- **vault 관리 자동화**: 프로그래밍 방식으로 노트 읽기, 콘텐츠 업데이트, frontmatter와 태그 관리, 파일 검색, 디렉터리 나열, 파일 삭제.
- **AI 워크플로우에 Obsidian 통합**: LLM이 연구, 작성, 또는 코딩 작업의 일부로 지식 베이스에 접근하고 수정할 수 있게 함.
- **사용자 정의 Obsidian 도구 구축**: 새로운 방식으로 vault 데이터와 상호작용하는 외부 애플리케이션 생성.

강력한 `mcp-ts-template`을 기반으로 구축된 이 서버는 MCP 표준을 통해 Obsidian 기능을 노출하는 표준화되고 안전하며 효율적인 방법을 제공합니다. vault 내부에서 실행되는 강력한 [Obsidian Local REST API 플러그인](https://github.com/coddingtonbear/obsidian-local-rest-api)과 통신함으로써 이를 달성합니다.

> **개발자 참고사항**: 이 저장소에는 코드베이스 패턴, 파일 위치, 코드 스니펫에 대한 빠른 참조를 제공하는 LLM 코딩 에이전트용 개발자 치트시트 역할을 하는 [.clinerules](.clinerules) 파일이 포함되어 있습니다.

## 기능

### 핵심 유틸리티

`cyanheads/mcp-ts-template`에서 제공하는 강력한 유틸리티를 활용합니다:

- **로깅**: 민감한 데이터 편집 기능이 있는 구조화되고 구성 가능한 로깅 (파일 로테이션, 콘솔, MCP 알림).
- **오류 처리**: 중앙집중식 오류 처리, 표준화된 오류 유형 (`McpError`), 자동 로깅.
- **구성**: 포괄적인 검증이 포함된 환경 변수 로딩 (`dotenv`).
- **입력 검증/무독화**: 스키마 검증을 위한 `zod`와 사용자 정의 무독화 로직 사용.
- **요청 컨텍스트**: 고유한 요청 ID를 통한 작업 추적 및 상관관계.
- **타입 안전성**: TypeScript와 Zod 스키마에 의한 강력한 타입 적용.
- **HTTP 전송 옵션**: SSE, 세션 관리, CORS 지원, 플러그인 가능한 인증 전략(JWT 및 OAuth 2.1)이 있는 내장 Hono 서버.

### Obsidian 통합

- **Obsidian Local REST API 통합**: `ObsidianRestApiService`에서 관리하는 HTTP 요청을 통해 Obsidian Local REST API 플러그인과 직접 통신.
- **포괄적인 명령 커버리지**: 주요 vault 작업을 MCP 도구로 노출 ([도구](#도구) 섹션 참조).
- **Vault 상호작용**: 읽기, 업데이트(추가, 앞에 삽입, 덮어쓰기), 검색(전역 텍스트/정규식, 검색/바꾸기), 나열, 삭제, frontmatter 및 태그 관리 지원.
- **타겟 유연성**: 도구는 경로, Obsidian에서 현재 활성화된 파일, 또는 주기적 노트(일간, 주간 등)로 파일을 타겟할 수 있음.
- **Vault 캐시 서비스**: 성능과 복원력을 향상시키는 지능형 인메모리 캐시. vault 콘텐츠를 캐시하고, 라이브 API가 실패할 경우 전역 검색 도구에 대한 폴백을 제공하며, 동기화를 유지하기 위해 주기적으로 새로 고침.
- **안전 기능**: 파일 작업을 위한 대소문자 구분 없는 경로 폴백, 수정 유형(추가, 덮어쓰기 등) 간의 명확한 구분.

## 설치

### 전제 조건

1.  **Obsidian**: Obsidian이 설치되어 있어야 합니다. #DONE✅
2.  **Obsidian Local REST API 플러그인**: Obsidian vault 내에서 [Obsidian Local REST API 플러그인](https://github.com/coddingtonbear/obsidian-local-rest-api)을 설치하고 활성화해야 합니다. #DONE✅
3.  **API 키**: Obsidian의 Local REST API 플러그인 설정에서 API 키를 구성해야 합니다. 서버를 구성하는 데 이 키가 필요합니다. #DONE✅
4.  **Node.js & npm**: Node.js(v18 이상 권장)와 npm이 설치되어 있어야 합니다.#DONE✅

## 구성

### MCP 클라이언트 설정

MCP 클라이언트의 구성 파일(예: `cline_mcp_settings.json`)에 다음을 추가하세요. 이 구성은 `npx`를 사용하여 서버를 실행하며, 아직 존재하지 않는 경우 패키지를 자동으로 다운로드하고 설치합니다:

```json
{
  "mcpServers": {
    "obsidian-mcp-server": {
      "command": "npx",
      "args": ["obsidian-mcp-server"],
      "env": {
        "OBSIDIAN_API_KEY": "YOUR_API_KEY_FROM_OBSIDIAN_PLUGIN",
        "OBSIDIAN_BASE_URL": "http://127.0.0.1:27123",
        "OBSIDIAN_VERIFY_SSL": "false",
        "OBSIDIAN_ENABLE_CACHE": "true"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

**참고**: Obsidian Local REST API 플러그인이 기본적으로 자체 서명된 인증서를 사용하기 때문에 여기서 SSL 확인이 false로 설정되어 있습니다. 프로덕션 환경에서 배포하는 경우, 암호화된 HTTPS 엔드포인트를 사용하고 자체 서명된 인증서를 신뢰하도록 서버를 구성한 후 `OBSIDIAN_VERIFY_SSL`을 `true`로 설정하는 것을 고려하세요.

소스에서 설치한 경우, `command`와 `args`를 로컬 빌드를 가리키도록 변경하세요:

```json
{
  "mcpServers": {
    "obsidian-mcp-server": {
      "command": "node",
      "args": ["/path/to/your/obsidian-mcp-server/dist/index.js"],
      "env": {
        "OBSIDIAN_API_KEY": "YOUR_OBSIDIAN_API_KEY",
        "OBSIDIAN_BASE_URL": "http://127.0.0.1:27123",
        "OBSIDIAN_VERIFY_SSL": "false",
        "OBSIDIAN_ENABLE_CACHE": "true"
      }
    }
  }
}
```

### 환경 변수

환경 변수를 사용하여 서버를 구성하세요. 이러한 환경 변수는 MCP 클라이언트 구성/설정(예: Cline용 `cline_mcp_settings.json`, Claude Desktop용 `claude_desktop_config.json`) 내에서 설정됩니다.

| 변수                                  | 설명                                                             | 필수                 | 기본값                   |
| :------------------------------------ | :-------------------------------------------------------------- | :------------------- | :----------------------- |
| **`OBSIDIAN_API_KEY`**                | Obsidian Local REST API 플러그인의 API 키.                        | **예**              | `undefined`              |
| **`OBSIDIAN_BASE_URL`**               | Obsidian Local REST API의 기본 URL.                             | **예**              | `http://127.0.0.1:27123` |
| `MCP_TRANSPORT_TYPE`                  | 서버 전송: `stdio` 또는 `http`.                                   | 아니오                | `stdio`                  |
| `MCP_HTTP_PORT`                       | HTTP 서버용 포트.                                                | 아니오                | `3010`                   |
| `MCP_HTTP_HOST`                       | HTTP 서버용 호스트.                                              | 아니오                | `127.0.0.1`              |
| `MCP_ALLOWED_ORIGINS`                 | CORS용 쉼표로 구분된 오리진. **프로덕션용 설정.**                    | 아니오                | (없음)                   |
| `MCP_AUTH_MODE`                       | 인증 전략: `jwt` 또는 `oauth`.                                   | 아니오                | (없음)                   |
| **`MCP_AUTH_SECRET_KEY`**             | JWT용 32자 이상 비밀키. **`jwt` 모드에 필수.**                     | **예 (`jwt`인 경우)** | `undefined`              |
| `OAUTH_ISSUER_URL`                    | OAuth 2.1 발급자의 URL.                                         | **예 (`oauth`인 경우)** | `undefined`            |
| `OAUTH_AUDIENCE`                      | OAuth 토큰용 대상 클레임.                                         | **예 (`oauth`인 경우)** | `undefined`            |
| `OAUTH_JWKS_URI`                      | JSON Web Key Set용 URI (선택사항, 생략시 발급자에서 도출).          | 아니오                | (도출됨)                 |
| `MCP_LOG_LEVEL`                       | 로깅 수준 (`debug`, `info`, `error` 등).                         | 아니오                | `info`                   |
| `OBSIDIAN_VERIFY_SSL`                 | SSL 확인을 비활성화하려면 `false`로 설정.                          | 아니오                | `true`                   |
| `OBSIDIAN_ENABLE_CACHE`               | 인메모리 vault 캐시를 활성화하려면 `true`로 설정.                   | 아니오                | `true`                   |
| `OBSIDIAN_CACHE_REFRESH_INTERVAL_MIN` | vault 캐시 새로 고침 간격(분).                                    | 아니오                | `10`                     |

### Obsidian API에 연결

MCP 서버를 Obsidian vault에 연결하려면 기본 URL(`OBSIDIAN_BASE_URL`)과 API 키(`OBSIDIAN_API_KEY`)를 구성해야 합니다. Obsidian Local REST API 플러그인은 연결하는 두 가지 방법을 제공합니다:

1.  **암호화됨 (HTTPS) - 기본값**:

    - 플러그인은 안전한 `https://` 엔드포인트(예: `https://127.0.0.1:27124`)를 제공합니다.
    - 이는 자체 서명된 인증서를 사용하므로 기본적으로 연결 오류가 발생합니다.
    - **이를 수정하려면**, `OBSIDIAN_VERIFY_SSL` 환경 변수를 `"false"`로 설정해야 합니다. 이는 서버가 자체 서명된 인증서를 신뢰하도록 지시합니다.

2.  **암호화되지 않음 (HTTP) - 단순함을 위해 권장**:
    - Obsidian 내의 플러그인 설정에서 "암호화되지 않은 (HTTP) 서버"를 활성화할 수 있습니다.
    - 이는 더 간단한 `http://` 엔드포인트(예: `http://127.0.0.1:27123`)를 제공합니다.
    - 이 URL을 사용할 때는 SSL 확인에 대해 걱정할 필요가 없습니다.

**MCP 클라이언트용 `env` 구성 예시:**

_암호화되지 않은 HTTP URL 사용 (권장):_

```json
"env": {
  "OBSIDIAN_API_KEY": "YOUR_API_KEY_FROM_OBSIDIAN_PLUGIN",
  "OBSIDIAN_BASE_URL": "http://127.0.0.1:27123"
}
```

_암호화된 HTTPS URL 사용:_

```json
"env": {
  "OBSIDIAN_API_KEY": "YOUR_API_KEY_FROM_OBSIDIAN_PLUGIN",
  "OBSIDIAN_BASE_URL": "https://127.0.0.1:27124",
  "OBSIDIAN_VERIFY_SSL": "false"
}
```

## 프로젝트 구조

코드베이스는 `src/` 디렉터리 내에서 모듈화된 구조를 따릅니다:

```
src/
├── index.ts           # 진입점: 서버 초기화 및 시작
├── config/            # 구성 로딩 (환경 변수, 패키지 정보)
│   └── index.ts
├── mcp-server/        # 핵심 MCP 서버 로직 및 기능 등록
│   ├── server.ts      # 서버 설정, 전송 처리, 도구/리소스 등록
│   ├── resources/     # MCP 리소스 구현 (현재 없음)
│   ├── tools/         # MCP 도구 구현 (도구별 하위 디렉터리)
│   └── transports/    # Stdio 및 HTTP 전송 로직
│       └── auth/      # 인증 전략 (JWT, OAuth)
├── services/          # 외부 API 또는 내부 캐싱용 추상화
│   └── obsidianRestAPI/ # Obsidian Local REST API용 타입 지정 클라이언트
├── types-global/      # 공유 TypeScript 타입 정의 (오류 등)
└── utils/             # 공통 유틸리티 함수 (로거, 오류 핸들러, 보안 등)
```

자세한 파일 트리는 `npm run tree`를 실행하거나 [docs/tree.md](docs/tree.md)를 참조하세요.

## Vault 캐시 서비스

이 서버에는 vault와의 상호작용 시 성능과 복원력을 향상시키도록 설계된 지능형 **인메모리 캐시**가 포함되어 있습니다.

### 목적과 이점

- **성능**: 파일 콘텐츠와 메타데이터를 캐시함으로써 서버는 특히 대형 vault에서 검색 작업을 훨씬 빠르게 수행할 수 있습니다. 이는 Obsidian Local REST API에 대한 직접 요청 수를 줄여 더 빠른 경험을 제공합니다.
- **복원력**: 캐시는 `obsidian_global_search` 도구의 폴백 역할을 합니다. 라이브 API 검색이 실패하거나 시간 초과되면 서버가 원활하게 캐시를 사용하여 결과를 제공하여 Obsidian API가 일시적으로 응답하지 않더라도 검색 기능이 계속 사용 가능하도록 보장합니다.
- **효율성**: 캐시는 효율적으로 설계되었습니다. 시작 시 초기 빌드를 수행한 다음 파일 수정을 확인하여 주기적으로 백그라운드에서 새로 고침하므로 지속적이고 무거운 API 폴링 없이도 합리적으로 최신 상태를 유지합니다.

### 작동 방식

1.  **초기화**: 활성화되면 `VaultCacheService`가 vault의 모든 `.md` 파일에 대한 인메모리 맵을 구축하여 내용과 수정 시간을 저장합니다.
2.  **주기적 새로 고침**: 캐시는 구성 가능한 간격(기본 10분)으로 자동으로 새로 고침됩니다. 새로 고침 중에는 마지막 확인 이후 새로 생성되거나 수정된 파일의 콘텐츠만 가져옵니다.
3.  **사전 업데이트**: `obsidian_update_file`과 같은 도구를 통해 파일이 수정된 후 서비스는 해당 특정 파일에 대한 캐시를 사전에 업데이트하여 즉시 일관성을 보장합니다.
4.  **검색 폴백**: `obsidian_global_search` 도구는 먼저 라이브 API 검색을 시도합니다. 실패하면 자동으로 인메모리 캐시 검색으로 폴백합니다.

### 구성

캐시는 기본적으로 활성화되어 있지만 환경 변수를 통해 구성할 수 있습니다:

- **`OBSIDIAN_ENABLE_CACHE`**: 캐시 서비스를 활성화하거나 비활성화하려면 `true`(기본값) 또는 `false`로 설정.
- **`OBSIDIAN_CACHE_REFRESH_INTERVAL_MIN`**: 주기적 백그라운드 새로 고침 간격을 분 단위로 정의. 기본값은 `10`.

## 도구

Obsidian MCP Server는 Model Context Protocol을 통해 호출할 수 있는 vault와 상호작용하기 위한 도구 모음을 제공합니다.

| 도구 이름                     | 설명                                                 | 주요 인수                                                 |
| :---------------------------- | :-------------------------------------------------- | :------------------------------------------------------- |
| `obsidian_read_note`          | 노트의 내용과 메타데이터를 검색합니다.                     | `filePath`, `format?`, `includeStat?`                    |
| `obsidian_update_note`        | 추가, 앞에 삽입, 또는 덮어쓰기로 파일을 수정합니다.        | `targetType`, `content`, `targetIdentifier?`, `wholeFileMode` |
| `obsidian_search_replace`     | 노트에서 검색 및 바꾸기 작업을 수행합니다.                | `targetType`, `replacements`, `useRegex?`, `replaceAll?` |
| `obsidian_global_search`      | 전체 vault에서 콘텐츠를 검색합니다.                      | `query`, `searchInPath?`, `useRegex?`, `page?`, `pageSize?` |
| `obsidian_list_notes`         | 폴더의 노트와 하위 디렉터리를 나열합니다.                 | `dirPath`, `fileExtensionFilter?`, `nameRegexFilter?`    |
| `obsidian_manage_frontmatter` | 노트의 frontmatter에서 키를 가져오거나 설정하거나 삭제합니다. | `filePath`, `operation`, `key`, `value?`                |
| `obsidian_manage_tags`        | 노트의 태그를 추가, 제거, 또는 나열합니다.                | `filePath`, `operation`, `tags`                          |
| `obsidian_delete_note`        | vault에서 노트를 영구적으로 삭제합니다.                   | `filePath`                                               |

_참고: 모든 도구는 포괄적인 오류 처리를 지원하고 구조화된 JSON 응답을 반환합니다._

## 리소스

**MCP 리소스는 이 버전에서 구현되지 않았습니다.**

이 서버는 현재 vault 조작을 위한 대화형 도구 제공에 중점을 둡니다. 향후 개발에서는 리소스 기능(예: 노트나 검색 결과를 읽기 가능한 리소스로 노출)을 도입할 수 있습니다.

## 개발

### 빌드 및 테스트

개발을 시작하려면 저장소를 복제하고 종속성을 설치한 후 다음 스크립트를 사용하세요:

```bash
# 종속성 설치
npm install

# 프로젝트 빌드 (TS를 dist/의 JS로 컴파일하고 실행 가능하게 만들기)
npm run rebuild

# stdio 전송을 사용하여 로컬에서 서버 시작
npm start:stdio

# http 전송을 사용하여 서버 시작
npm run start:http

# Prettier를 사용하여 코드 포맷팅
npm run format

# MCP Inspector 도구를 사용하여 서버의 기능 검사
npm run inspect:stdio
# 또는 http 전송의 경우:
npm run inspect:http
```

## 라이선스

이 프로젝트는 Apache License 2.0에 따라 라이선스가 부여됩니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 참조하세요.

---

<div align="center">
<a href="https://modelcontextprotocol.io/">Model Context Protocol</a>로 구축됨
</div>