---
title: Telegram Bot, Github, Netlify로 자체 무료 Read It Later 구축
description: 웹 스크래핑이 점점 귀찮아져서 자체 무료 Read It Later를 구축했다.
slug: telegram-github-netlify-free-scraping-bot
date: 2026-05-14T05:28:54+09:00
image: 20260518061455.png
categories: [Project]
tags: [Telegram, Netlify, Bot, Obsidian, Read-It-Later, 스크래핑, 옵시디언, 텔레그램, GitHub]
weight: 0
created: 2026-02-24 21:23
updated: 2026-09-22 06:12
---

# 읽어야 될 글들이 너무 많아졌다.
인터넷 서핑을 하다보면 AI 대전환 시대여서 그런지 하루가 멀다하고 새로운 기술들이 쏟아져 나온다.<br>기술을 습득하는 것도 어려운 일이지만, 그런 기술을 습득하기 위해 관련 글이나 유튜브를 찾는 것, 그리고 나중에 읽어야 겠다고 어딘가 링크를 저장해 놓는 것도 보통 일이 아니다.<br>괜찮은 정보라고 생각되거나 언젠가 봐야겠다고 생각이 드는 것들은 URL을 '카카오톡 나에게 보내기'를 통해서 보내 놓거나 [Pushbullet](https://www.pushbullet.com/)을 이용했다.<br>그런데, 어느 순간부터 정보의 양이 많아지기 시작해서 이전에 내가 어떤 정보들을 보내놨는지 또 찾아야 하는 일들이 발생했다. 😇<br><br>그렇게 흔히 'Read It Later' 라 불리는 앱들을 찾기 시작한다.<br>과거에는 Omnivore, Pocket, [Raindrop.io](https://raindrop.io/) 과 같은 서비스들이 있었는데, 현재는 대부분 다 서비스를 종료했고, Raindrop만 살아남았다.<br>한편 Raindrop의 경우 나중에 읽을 URL을 북마크 형태로 정리하기에는 좋지만, 유료로 사용해야만 웹 스크래핑 서비스를 제공한다. 😂<br>PC나 모바일에서 [Obsidian Web Clipper](https://obsidian.md/clipper) 확장을 이용해서 스크래핑하면 로컬 옵시디언 볼트에 md 파일로 잘 들어가긴 하지만 모바일에서는 꽤나 귀찮다.<br>그래서 URL을 어딘가 던져주면 웹 스크래핑을 해서 내 옵시디언 볼트에 넣어주는 앱이 없을까 하다 결과적으로 직접 만들었다. 

![안드로이드 FireFox 확장 Obsidian Web Clipper](20260514055531.png)

# 무료로 셀프 Read It Later 구축
내 기준 최대한 자세히 설명하면 다음과 같다. 😇

## 1단계: 준비물 (계정 및 프로그램)
먼저 4개의 서비스 계정과 2개의 프로그램이 필요하다.
1. **필요한 계정 (모두 무료)**
	- [**GitHub**](https://github.com/): 내 코드 파일들을 보관하는 온라인 창고
	- [**Netlify**](https://www.netlify.com/): GitHub에 올린 코드를 실제 웹 서비스로 돌려주는 서버
	- [**Koofr**](https://koofr.eu/): 추출된 마크다운 파일(.md)이 저장되는 온라인 저장소입니다. (WebDAV 지원)
	- **Telegram**: 봇과 대화하고 링크를 전달하는 인터페이스
<br>현재 블로그도 GitHub와 Netlify로 무료 계정으로 운영중이다.<br>또, Obsidian이 각 기기(휴대폰, 태블릿, PC)마다 설치되어 있으며, Remotely Save 플러그인을 통해 Koofr의 WebDAV에 연동되어 있다.<br>나의 경우 Telegram 클라이언트로 [iMe](https://play.google.com/store/apps/details?id=com.iMe.android)를 사용중이다.

2. **필요한 프로그램**
	- [**Visual Studio Code (VS Code)**](https://code.visualstudio.com/): 코드를 수정하고 편집하는 메모장 같은 프로그램
	- [**GitHub Desktop**](https://desktop.github.com/download/): 복잡한 명령어 대신 마우스 클릭만으로 코드를 GitHub에 올릴 수 있게 해주는 프로그램
<br>VS Code가 무겁게 느껴지는 환경이라면 [Notepad++](https://notepad-plus-plus.org/) 정도만 써도 무방하다.

---

## 2단계: 텔레그램 봇 및 ID 준비
봇을 만들기 위해 텔레그램에서 두 가지 정보를 얻어야 한다.

1. **봇 생성**: `@BotFather`를 검색해 대화를 시작하고 `/newbot`을 입력하여 봇을 만든다. <br>마지막에 주는 **HTTP API Token**을 따로 적어둔다.    
2. **내 ID 확인**: `@userinfobot`을 검색해 메시지를 보내면 숫자(예: `12345678`)를 알려준다.<br>이게 `ALLOWED_USER_ID`가 된다.
<br>텔레그램 봇의 경우 Private 봇이 따로 있지 않기 때문에 봇을 호출했을 때, 나만 사용할 수 있도록 내 ID를 검증한 후에 작동하도록 하기 위함이다.

![iMe에서 @BotFather 호출 후 설정화면면](20260514062748.png) ![iMe에서는 프로필을 누르면 My ID가 바로 조회된다](20260514062810.png)

---

## 3단계: 폴더 구조 잡기

VS Code를 열고 컴퓨터에 새로운 폴더를 하나 만든다. 구조는 반드시 아래와 같아야 한다.

```
my-scrap-bot/ (내 폴더 이름)
├── app/
│   └── api/
│       └── telegram/
│           └── route.ts  <-- 최종 코드
├── package.json          <-- 프로그램 설정 파일
└── .gitignore            <-- 보안을 위해 업로드 제외할 목록
```

---

## 4단계: 핵심 코드 파일 작성

각 파일을 메모장(VS Code)으로 열고 아래 내용들을 복사해서 붙여넣는다.

### 1. `package.json` (설정 파일)

이 파일은 어떤 라이브러리가 필요한지 정의한다. **Next.js 15.4.2** 버전을 명시하여 보안 문제를 해결한 상태다.<br>처음엔 [Vercel](https://vercel.com/)을 이용하거나 Vercel CLI를 이용해서 GitHub 없이 바로 빌드를 하려고 했는데, 이 Next.js 버전으로 인한 보안 오류가 어마어마 하게 났다. 😇<br>빌드 오류가 수십번 났는데, Next.js 버전 숫자 하나하나 가지고 따지고 들어서 결국엔 지쳐서 Netlify로 옮겼는데, 동일한 오류가 나나 싶더니 Netlify 자체적으로 AI로 빌드 오류를 잡아주는 기능이 있어서 2~3번의 Failed 이후에 Complete 메세지를 보게 됐다. 😀

```
{
  "name": "my-scrap-bot",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "@mozilla/readability": "^0.5.0",
    "iconv-lite": "^0.6.3",
    "jsdom": "^24.0.0",
    "next": "15.4.2",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "turndown": "^7.1.3",
    "webdav": "^5.6.0"
  },
  "devDependencies": {
    "@types/jsdom": "^21.1.6",
    "@types/node": "^20.0.0",
    "@types/react": "^19.0.0",
    "@types/turndown": "^5.0.4",
    "typescript": "^5.0.0"
  }
}
```

![Netlify 자체 빌드 오류 AI Assistant : Why did it fail?](20260518054354.png)

### 2. `app/api/telegram/route.ts` (최종 로직)

몇 번의 테스트를 거쳐 EUC-KR 인코딩 문제 등을 모두 해결한 아래 **코드**를 이 위치에 그대로 붙여넣으면 된다.<br>아래 코드 중 2가지 정도는 본인이 원하는 대로 수정할 수 있다.
- 옵시디언에 저장될 md 파일의 프론트매터 tags
	- tags: ["ReadItLater"]
- Koofr Webdav에 스크래핑된 md 파일이 저장될 위치
	- await client.putFileContents(`/ReadItLater/${fileName}`, finalContent);

```
import { NextRequest, NextResponse } from 'next/server';
import { Readability } from '@mozilla/readability';
import { JSDOM } from 'jsdom';
// @ts-ignore
import TurndownService from 'turndown';
import { createClient } from 'webdav';
import iconv from 'iconv-lite';

const TELEGRAM_TOKEN = process.env.TELEGRAM_TOKEN;
const KOOFR_EMAIL = process.env.KOOFR_EMAIL;
const KOOFR_APP_PASSWORD = process.env.KOOFR_APP_PASSWORD;
const ALLOWED_USER_ID = process.env.ALLOWED_USER_ID;
const KOOFR_WEBDAV_URL = 'https://app.koofr.net/dav/Koofr';

// jsdom, iconv-lite, webdav는 Node.js API를 사용하므로 Edge Runtime을 사용하지 않는다.
export const runtime = 'nodejs';
// Netlify/Vercel 등에서 지원하는 플랫폼이라면 함수 실행 제한 시간을 늘려본다.
// (플랫폼/플랜에 따라 무시될 수 있음 — 실제 제한은 호스팅 콘솔에서 확인 필요)
export const maxDuration = 60;

const DESKTOP_USER_AGENT =
  'Mozilla/5.0 (Windows NT 10.0; Win64; x64) ' +
  'AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36';

const MOBILE_USER_AGENT =
  'Mozilla/5.0 (iPhone; CPU iPhone OS 17_4 like Mac OS X) ' +
  'AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.4 Mobile/15E148 Safari/604.1';

const FETCH_TIMEOUT_MS = 20_000;

// 봇 차단/비정상 접근 안내 페이지에서 흔히 쓰이는 문구.
// HTTP 200이 와도 실제로는 차단 페이지인 경우를 잡아내기 위한 용도.
const BLOCK_PAGE_HINTS = [
  '비정상적인 접근',
  '자동화된 접근',
  '접근이 제한',
  '정상적인 경로로 접속',
  '로봇이 아닙니다',
  'captcha',
  'Access Denied',
  'Attention Required',
];

/**
 * m. 서브도메인 등 모바일 전용 페이지는 모바일 UA로 접근해야
 * 정상적인 모바일 레이아웃(및 본문)을 받을 확률이 높다.
 * 데스크톱 UA로 접근하면 "PC로 접속해주세요" 같은 안내 페이지만 오는 경우가 있다.
 */
function pickUserAgent(hostname: string): string {
  return /^m\.|\.m\.|^mobile\./i.test(hostname) ? MOBILE_USER_AGENT : DESKTOP_USER_AGENT;
}

function looksLikeBlockPage(sampleHtml: string): boolean {
  return BLOCK_PAGE_HINTS.some(hint => sampleHtml.includes(hint));
}

async function fetchWithTimeout(
  url: string,
  headers: Record<string, string>,
  timeoutMs = FETCH_TIMEOUT_MS,
): Promise<Response> {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), timeoutMs);
  try {
    return await fetch(url, { redirect: 'follow', headers, signal: controller.signal });
  } finally {
    clearTimeout(timer);
  }
}

type FetchResult = { buffer: Buffer; contentType: string | null; finalUrl: string };

/**
 * 동일 URL에 대해 UA/Referer 조합을 바꿔가며 최대 2회 시도한다.
 * - 1차 시도: 호스트에 맞는 UA(m.* 이면 모바일 UA) + 구글 리퍼러
 * - 2차 시도: 반대쪽 UA + 자기 사이트 리퍼러 + (1차에서 받은 쿠키가 있으면 함께 전송)
 * 응답이 비정상적으로 짧거나 "차단 안내" 문구가 보이면 실패로 간주하고 다음 시도로 넘어간다.
 * 두 시도 모두 실패하면 마지막 에러를 던진다 — 상위에서 사용자에게 구체적인 원인을 보여줄 수 있게 한다.
 */
async function fetchWithRetry(url: string): Promise<FetchResult> {
  let hostname = '';
  try {
    hostname = new URL(url).hostname;
  } catch {
    // no-op
  }

  const attempts: Array<{ ua: string; referer: string }> = [
    { ua: pickUserAgent(hostname), referer: 'https://www.google.com/' },
    {
      ua: hostname.startsWith('m.') ? DESKTOP_USER_AGENT : MOBILE_USER_AGENT,
      referer: hostname ? `https://${hostname}/` : 'https://www.google.com/',
    },
  ];

  let lastError: unknown = new Error('알 수 없는 오류로 모든 시도가 실패했습니다.');
  let carriedCookie: string | undefined;

  for (const attempt of attempts) {
    try {
      const headers: Record<string, string> = {
        'User-Agent': attempt.ua,
        Accept: 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
        'Accept-Language': 'ko-KR,ko;q=0.9,en-US;q=0.7,en;q=0.5',
        'Accept-Encoding': 'gzip, deflate, br',
        Referer: attempt.referer,
        Connection: 'keep-alive',
        'Upgrade-Insecure-Requests': '1',
        'Sec-Fetch-Dest': 'document',
        'Sec-Fetch-Mode': 'navigate',
        'Sec-Fetch-Site': 'same-origin',
        'Sec-Fetch-User': '?1',
      };
      if (carriedCookie) headers['Cookie'] = carriedCookie;

      const response = await fetchWithTimeout(url, headers);

      const setCookie = response.headers.get('set-cookie');
      if (setCookie) carriedCookie = setCookie;

      if (!response.ok) {
        lastError = new Error(`HTTP ${response.status} ${response.statusText} (요청: ${url})`);
        continue;
      }

      const contentType = response.headers.get('content-type');
      const buffer = Buffer.from(await response.arrayBuffer());
      const finalUrl = response.url || url;

      if (buffer.length < 500) {
        lastError = new Error('응답 본문이 비정상적으로 짧습니다 (차단/오류 페이지로 추정).');
        continue;
      }

      const sample = buffer.subarray(0, 8000).toString('utf-8');
      if (looksLikeBlockPage(sample)) {
        lastError = new Error('사이트가 비정상 접근으로 판단해 차단 페이지를 반환한 것으로 보입니다.');
        continue;
      }

      return { buffer, contentType, finalUrl };
    } catch (error) {
      lastError =
        error instanceof Error && error.name === 'AbortError'
          ? new Error(`요청 시간 초과 (${FETCH_TIMEOUT_MS}ms, 요청: ${url})`)
          : error;
    }
  }

  throw lastError;
}

/**
 * 직접 요청이 계속 403/차단으로 막히는 사이트(예: 뽐뿌)를 위한 최후 수단.
 * ppomppu 같은 곳은 UA/Referer를 바꿔도 뚫리지 않는 경우가 많은데, 이는 헤더 문제가
 * 아니라 서버리스(클라우드) IP 대역 자체를 차단하는 WAF 정책일 가능성이 높다.
 * r.jina.ai는 자체 크롤러가 대신 페이지를 가져와 정제된 마크다운으로 반환해주는
 * 무료 공개 서비스라, 우리 서버 IP가 차단당해도 우회할 수 있다.
 * (단, jina 쪽도 대상 사이트로부터 언젠가 막힐 수는 있다 — 100% 보장은 아님)
 */
async function fetchViaJinaReader(url: string): Promise<{ title: string; markdown: string } | null> {
  try {
    const readerUrl = `https://r.jina.ai/${url}`;
    const response = await fetchWithTimeout(
      readerUrl,
      {
        Accept: 'text/plain',
        'X-Return-Format': 'markdown',
      },
      25_000,
    );

    if (!response.ok) return null;

    const text = await response.text();
    if (!text || text.trim().length < 200) return null;

    const titleMatch = text.match(/^Title:\s*(.+)$/m);
    const markdownStart = text.search(/^Markdown Content:/m);
    const markdown =
      markdownStart >= 0
        ? text.slice(markdownStart).replace(/^Markdown Content:\s*/, '')
        : text;

    return {
      title: normalizeText(titleMatch?.[1]),
      markdown: markdown.trim(),
    };
  } catch (error) {
    console.error('Jina reader fallback failed:', error);
    return null;
  }
}

/**
 * HTML 앞부분은 charset 선언 자체가 ASCII이므로 latin1로 읽어도 안전하다.
 * meta charset과 과거형 http-equiv/content 선언을 모두 지원한다.
 */
function getDeclaredCharset(buffer: Buffer, contentType: string | null): string | null {
  const headerMatch = contentType?.match(/charset\s*=\s*["']?([^\s;"']+)/i);
  if (headerMatch) return headerMatch[1].toLowerCase();

  const head = buffer.subarray(0, 64 * 1024).toString('latin1');
  const metaCharset = head.match(/<meta\b[^>]*\bcharset\s*=\s*["']?([^\s"'/>;]+)/i);
  if (metaCharset) return metaCharset[1].toLowerCase();

  const httpEquiv = head.match(
    /<meta\b[^>]*\bcontent\s*=\s*["'][^"']*charset\s*=\s*([^\s;"']+)[^"']*["'][^>]*>/i,
  );
  return httpEquiv?.[1]?.toLowerCase() ?? null;
}

function normalizeCharset(charset: string | null): string | null {
  if (!charset) return null;
  const value = charset.trim().toLowerCase().replace(/["']/g, '');

  if (['utf-8', 'utf8'].includes(value)) return 'utf-8';
  if (
    value.includes('euc-kr') ||
    value.includes('cp949') ||
    value.includes('ms949') ||
    value.includes('x-windows-949') ||
    value.includes('ks_c_5601') ||
    value.includes('ks-c-5601')
  ) {
    // CP949는 EUC-KR의 확장 문자까지 포함하므로 국내 레거시 사이트에 더 안전하다.
    return 'cp949';
  }

  return iconv.encodingExists(value) ? value : null;
}

function isValidUtf8(buffer: Buffer): boolean {
  try {
    new TextDecoder('utf-8', { fatal: true }).decode(buffer);
    return true;
  } catch {
    return false;
  }
}

function countBrokenCharacters(value: string): number {
  return (
    (value.match(/\uFFFD/g) || []).length +
    (value.match(/[\u0000-\u0008\u000B\u000C\u000E-\u001F]/g) || []).length
  );
}

/**
 * 최신 네이버처럼 선언값과 실제 바이트가 어긋나는 경우를 막기 위해
 * 유효한 UTF-8 바이트는 UTF-8을 최우선으로 사용한다.
 * UTF-8이 아니면 선언 인코딩과 CP949 후보 중 손상 문자가 적은 결과를 택한다.
 */
function decodeHtml(buffer: Buffer, contentType: string | null): string {
  // UTF-8 BOM
  if (buffer.length >= 3 && buffer[0] === 0xef && buffer[1] === 0xbb && buffer[2] === 0xbf) {
    return iconv.decode(buffer, 'utf-8');
  }

  if (isValidUtf8(buffer)) return iconv.decode(buffer, 'utf-8');

  const declared = normalizeCharset(getDeclaredCharset(buffer, contentType));
  const encodings = Array.from(new Set([declared, 'cp949'].filter(Boolean))) as string[];
  const candidates = encodings.map(encoding => ({
    encoding,
    html: iconv.decode(buffer, encoding),
  }));

  candidates.sort((a, b) => countBrokenCharacters(a.html) - countBrokenCharacters(b.html));
  return candidates[0]?.html ?? iconv.decode(buffer, 'utf-8');
}

function normalizeText(value: string | null | undefined): string {
  return (value || '')
    .normalize('NFC')
    .replace(/\uFFFD+/g, '')
    .replace(/[\u0000-\u001F\u007F]/g, ' ')
    .replace(/\s+/g, ' ')
    .trim();
}

function yamlString(value: string): string {
  return JSON.stringify(normalizeText(value));
}

function makeSafeFileName(title: string): string {
  // 네이버 블로그 제목은 글자 수/문자 제한이 없어서 국기 이모지(🇸🇬 같은 것도 실제로는
  // 코드포인트 2개가 결합된 특수 문자), 《》 같은 특수 괄호, 기타 심볼이 자유롭게
  // 들어간다. 이런 문자들은 WebDAV 경로에서 깨지거나 서버가 처리하지 못해 404를
  // 유발할 수 있으므로, 화이트리스트 방식(글자/숫자/공백/기본 문장부호만 허용)으로
  // 한 글자씩 걸러낸다 — 알려진 위험 문자만 골라서 막는 것보다 훨씬 안전하다.
  const ALLOWED_PUNCTUATION = new Set([
    '-', '_', '.', ',', '(', ')', '[', ']', '!', '?', '~', "'",
  ]);

  const normalized = normalizeText(title);
  let cleaned = '';
  for (const ch of normalized) {
    // \p{L}: 모든 언어의 글자, \p{N}: 숫자, \p{Zs}: 공백류
    if (/[\p{L}\p{N}\p{Zs}]/u.test(ch) || ALLOWED_PUNCTUATION.has(ch)) {
      cleaned += ch;
    } else {
      // 이모지(국기 포함), 《》 등 특수 괄호, 기타 심볼은 전부 하이픈으로.
      cleaned += '-';
    }
  }

  cleaned = cleaned
    // 화이트리스트를 통과했더라도 경로/URL에서 특별한 의미를 가지는 문자는
    // 한 번 더 확실하게 걸러낸다 (' 나 ( ) 등은 남기되, 실제 구분자 역할을 하는 것만).
    .replace(/[\\/:*?"<>|%#&+;@=]/g, '-')
    .replace(/-{2,}/g, '-')
    .replace(/\.+$/g, '')
    .replace(/\s+/g, ' ')
    .trim()
    .slice(0, 150);

  return `${cleaned || 'Untitled'}.md`;
}

/**
 * Koofr(WebDAV) 업로드. 파일명에 sanitize로 걸러내지 못한 문자가 남아 있거나
 * 그 외 이유로 PUT이 실패하면, 타임스탬프 기반의 확실히 안전한 이름으로
 * 한 번 더 시도해서 최소한 내용은 보존한다.
 */
async function uploadToKoofr(
  client: ReturnType<typeof createClient>,
  fileName: string,
  content: string,
): Promise<string> {
  try {
    await client.putFileContents(`/ReadItLater/${fileName}`, content, {
      overwrite: true,
    });
    return fileName;
  } catch (error) {
    console.error(`Koofr upload failed for "${fileName}", retrying with safe name:`, error);
    const fallbackName = `Clip-${Date.now()}.md`;
    await client.putFileContents(`/MiNi_PKM/Clippings/${fallbackName}`, content, {
      overwrite: true,
    });
    return fallbackName;
  }
}

/**
 * 네이버 블로그/카페 PC 버전은 실제 본문이 iframe(예: iframe#mainFrame)으로
 * 별도 URL에서 로드된다. m. 서브도메인이 아닌 경우에만 시도한다.
 */
function findMainIframeSrc(document: Document, baseUrl: string): string | null {
  let host = '';
  try {
    host = new URL(baseUrl).hostname;
  } catch {
    return null;
  }
  if (!/(^|\.)naver\.com$/.test(host) || host.startsWith('m.')) return null;

  const iframe =
    document.querySelector('iframe#mainFrame') ||
    document.querySelector('iframe#cafe_main') ||
    document.querySelector('iframe[name="mainFrame"]');

  const src = iframe?.getAttribute('src');
  if (!src) return null;

  try {
    return new URL(src, baseUrl).href;
  } catch {
    return null;
  }
}

/**
 * Readability가 실패하거나 텍스트가 거의 없는데 원본에는 이미지가 많은 경우
 * (뽐뿌처럼 이미지 위주 게시물에서 자주 발생) 대비용 대체 추출기.
 * 광고/댓글/네비게이션성 클래스명을 제외하고, 이미지 수 + 텍스트 길이 기준으로
 * 가장 "본문 같은" 블록을 고른다. 정교하지는 않지만 Readability가 완전히
 * 빈 손으로 돌아오는 것보다는 낫다.
 */
function pickBestContentBlock(document: Document): Element | null {
  const EXCLUDE_CLASS_HINTS =
    /(ad|banner|comment|reply|sidebar|related|footer|header|^nav$|snb|gnb|share|copyright)/i;

  const candidates = Array.from(document.querySelectorAll('div, td, article, section, main'));
  let best: Element | null = null;
  let bestScore = 0;

  for (const el of candidates) {
    const cls = (el.className || '').toString();
    if (EXCLUDE_CLASS_HINTS.test(cls)) continue;

    const imgCount = el.querySelectorAll('img').length;
    const textLen = normalizeText(el.textContent).length;
    const score = imgCount * 200 + textLen;

    if (score > bestScore && (imgCount > 0 || textLen > 100)) {
      bestScore = score;
      best = el;
    }
  }

  return best;
}

type ExtractedArticle = { title: string; content: string; excerpt: string; byline: string };

/**
 * Readability를 우선 시도하되, 텍스트 임계값을 낮춰 이미지 위주 게시물도
 * 통과할 확률을 높인다. 결과가 비어있거나 지나치게 부실하면(원본엔 이미지가
 * 있는데 추출 결과엔 없는 경우 등) pickBestContentBlock으로 대체한다.
 */
function extractArticle(document: Document): ExtractedArticle | null {
  let article: ReturnType<Readability['parse']> = null;

  try {
    // Readability는 전달받은 document를 변형하므로, 대체 추출을 위해
    // 원본은 그대로 두고 클론에 대해 파싱한다.
    const clone = document.cloneNode(true) as Document;
    article = new Readability(clone, { charThreshold: 120 }).parse();
  } catch (error) {
    console.error('Readability parse error:', error);
  }

  const readabilityText = normalizeText(article?.textContent);
  const readabilityImgCount = article?.content ? (article.content.match(/<img/gi) || []).length : 0;
  const rawImgCount = document.querySelectorAll('img').length;

  const readabilityLooksWeak =
    !article?.content || (readabilityText.length < 80 && readabilityImgCount === 0 && rawImgCount > 0);

  if (readabilityLooksWeak) {
    const fallbackEl = pickBestContentBlock(document);
    if (fallbackEl) {
      return {
        title: normalizeText(article?.title),
        content: fallbackEl.innerHTML,
        excerpt: normalizeText(fallbackEl.textContent).slice(0, 200),
        byline: normalizeText(article?.byline),
      };
    }
  }

  if (!article?.content) return null;

  return {
    title: normalizeText(article.title),
    content: article.content,
    excerpt: normalizeText(article.excerpt),
    byline: normalizeText(article.byline),
  };
}

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    const message = body.message;

    if (!message?.text) return NextResponse.json({ ok: true });

    const chatId = message.chat.id;
    if (ALLOWED_USER_ID && String(chatId) !== String(ALLOWED_USER_ID)) {
      return NextResponse.json({ ok: true });
    }

    const urlMatch = message.text.match(/https?:\/\/[^\s]+/g);
    if (!urlMatch) return NextResponse.json({ ok: true });

    let targetUrl = urlMatch[0];
    try {
      const urlObj = new URL(targetUrl);
      ['utm_source', 'utm_medium', 'utm_campaign', 'ref'].forEach(param =>
        urlObj.searchParams.delete(param),
      );
      targetUrl = urlObj.toString();
    } catch {
      // URL 생성 실패 시 Telegram에서 추출한 원문을 그대로 사용한다.
    }

    await sendTelegramMessage(chatId, '🔍 원본 콘텐츠 및 고화질 이미지 분석 중...');

    let title = 'Untitled';
    let markdownContent = '';
    let description = '';
    let author = 'Unknown';
    let effectiveUrl = targetUrl;
    let primaryError: unknown = null;

    try {
      const initial = await fetchWithRetry(targetUrl);

      let html = decodeHtml(initial.buffer, initial.contentType);
      effectiveUrl = initial.finalUrl;
      let dom = new JSDOM(html, { url: effectiveUrl });
      let document = dom.window.document;

      // 네이버 블로그/카페 PC 버전처럼 실제 본문이 iframe에 있는 경우, 그 iframe을 따라가서
      // 실제 콘텐츠 페이지를 다시 받아온다.
      const hopUrl = findMainIframeSrc(document, effectiveUrl);
      if (hopUrl) {
        try {
          const hop = await fetchWithRetry(hopUrl);
          html = decodeHtml(hop.buffer, hop.contentType);
          effectiveUrl = hop.finalUrl;
          dom = new JSDOM(html, { url: effectiveUrl });
          document = dom.window.document;
        } catch (hopError) {
          // iframe 후속 요청이 실패해도 원본 문서로 계속 시도해본다.
          console.error('iframe hop fetch failed:', hopError);
        }
      }

      // Readability가 DOM을 변경하기 전에 신뢰도 높은 제목 후보를 보관한다.
      const ogTitle = normalizeText(
        document.querySelector('meta[property="og:title"]')?.getAttribute('content'),
      );
      const originalDocumentTitle = normalizeText(document.title);

      ['script', 'style', 'noscript', 'footer', 'nav'].forEach(selector => {
        document.querySelectorAll(selector).forEach(element => element.remove());
      });

      // 본문 iframe을 무조건 제거하면 일부 네이버 카페 페이지의 실제 글도 사라질 수 있다.
      // 광고/추적용 iframe만 제거하고, 같은 네이버 계열 iframe은 남긴다.
      document.querySelectorAll('iframe').forEach(iframe => {
        const src = iframe.getAttribute('src') || '';
        let keep = false;
        try {
          const host = new URL(src, effectiveUrl).hostname;
          keep = host === 'naver.com' || host.endsWith('.naver.com');
        } catch {
          keep = false;
        }
        if (!keep) iframe.remove();
      });

      document.querySelectorAll('a').forEach(anchor => {
        const href = anchor.getAttribute('href');
        if (!href || href === '#' || anchor.querySelector('img')) {
          anchor.replaceWith(...Array.from(anchor.childNodes));
        }
      });

      const article = extractArticle(document);

      if (!article?.content) {
        throw new Error('본문을 추출할 수 없습니다 (Readability 및 대체 추출 모두 실패).');
      }

      title = ogTitle || article.title || originalDocumentTitle || 'Untitled';

      const turndownService = new TurndownService({
        headingStyle: 'atx',
        hr: '---',
        bulletListMarker: '-',
        codeBlockStyle: 'fenced',
      });

      turndownService.addRule('absoluteImages', {
        filter: 'img',
        replacement: function (_content, node: any) {
          const src =
            node.getAttribute('data-lazy-src') ||
            node.getAttribute('data-source') ||
            node.getAttribute('data-src') ||
            node.getAttribute('src');

          if (!src) return '';

          try {
            let absoluteUrl = new URL(src.split(' ')[0], effectiveUrl).href;

            if (absoluteUrl.includes('pstatic.net') || absoluteUrl.includes('blogfiles')) {
              const cleanUrl = new URL(absoluteUrl);
              if (cleanUrl.searchParams.has('type')) cleanUrl.searchParams.set('type', 'w1');
              absoluteUrl = cleanUrl.toString();
            }

            const alt = normalizeText(node.getAttribute('alt')) || 'image';
            return `\n![${alt.replace(/[\[\]]/g, '')}](${absoluteUrl})\n`;
          } catch {
            return '';
          }
        },
      });

      markdownContent = turndownService.turndown(article.content);
      description = article.excerpt;
      author = article.byline || 'Unknown';
    } catch (error) {
      // 직접 접근이 막히거나(403 등) 본문 추출에 실패한 경우, 여기서 바로 실패
      // 처리하지 않고 Jina Reader 우회 경로를 마지막으로 한 번 더 시도해본다.
      primaryError = error;
      console.error('Primary scrape failed, trying Jina reader fallback:', error);

      const jina = await fetchViaJinaReader(targetUrl);
      if (!jina || !jina.markdown) {
        const detail = primaryError instanceof Error ? primaryError.message : String(primaryError);
        await sendTelegramMessage(
          chatId,
          `❌ 처리 중 에러가 발생했습니다.\n\n원인: ${detail.slice(0, 300)}\n(대체 경로(Jina Reader) 시도도 실패)`,
        );
        return NextResponse.json({ ok: true });
      }

      title = jina.title || title;
      markdownContent = jina.markdown;
      description = '';
      author = 'Unknown';
    }

    try {
      const now = new Date();

      const frontmatter = `---
title: ${yamlString(title)}
description: ${yamlString(description)}
source: ${yamlString(effectiveUrl)}
author: ${yamlString(author)}
created: ${now.toISOString().split('T')[0]}
scraped_at: ${yamlString(now.toLocaleString('ko-KR', { timeZone: 'Asia/Seoul' }))}
tags: ["ReadItLater"]
---

`;

      const finalContent = `${frontmatter}# ${title}\n\n${markdownContent}`;
      const fileName = makeSafeFileName(title);

      const client = createClient(KOOFR_WEBDAV_URL, {
        username: KOOFR_EMAIL,
        password: KOOFR_APP_PASSWORD,
      });
      const savedFileName = await uploadToKoofr(client, fileName, finalContent);

      await sendTelegramMessage(chatId, `✅ 아카이빙 완료!\n\n📄 ${savedFileName}`, true);
    } catch (error) {
      console.error('Upload error:', error);
      const detail = error instanceof Error ? error.message : String(error);
      await sendTelegramMessage(
        chatId,
        `❌ 저장 중 에러가 발생했습니다.\n\n원인: ${detail.slice(0, 300)}`,
      );
    }

    return NextResponse.json({ ok: true });
  } catch (error) {
    console.error('Webhook error:', error);
    return NextResponse.json({ ok: false }, { status: 500 });
  }
}

async function sendTelegramMessage(chatId: number, text: string, disablePreview = false) {
  const url = `https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage`;
  await fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      chat_id: chatId,
      text,
      disable_web_page_preview: disablePreview,
    }),
  });
}
```

---

## 5단계: GitHub Desktop으로 업로드

1. **GitHub Desktop**을 연다.
    
2. `Create New Repository on your local drive`를 선택하고 방금 만든 폴더를 지정한다.
    
3. `Publish Repository` 버튼을 눌러 내 GitHub 계정으로 코드를 보낸다.
    <br>※ 여기서 Repository는 비공개(Private)를 추천한다.

---

## 6단계: Netlify 설정 및 배포

이제 GitHub에 업로드된 코드를 실제 서버로 옮길 차례다.

1. **Netlify**에 로그인하고 `Add new site` -> `Import from an existing project`를 눌러 내 GitHub의 해당 저장소를 연결한다.
    
2. **중요: 환경 변수 설정**
    
    - `Site configuration` -> `Environment variables`로 들어가서 다음 5개를 입력한다.
        
    - `TELEGRAM_TOKEN`: 텔레그램 BotFather에게 받은 토큰
        
    - `KOOFR_EMAIL`: Koofr 이메일(로그인 계정)
        
    - `KOOFR_APP_PASSWORD`: Koofr 설정에서 만든 'App Password'
	    - 로그인 비밀번호가 아니라 Koofr 로그인 이후에 설정 > 비밀번호 화면에서 발급가능한 '앱 비밀번호'다.
        
    - `ALLOWED_USER_ID`: 내 텔레그램 숫자 ID
        
    - `NODE_VERSION`: `22`
	    - NODE_VERSION의 경우 이번 Read It Later 외에 다른 앱들도 같이 이용하는 사람이라면 변수가 아니라 로컬에서 VS Code에서 netlify.toml 같은 파일을 만들어서 설정해주는 것도 방법이다. 내가 그렇게 사용하고 있다.
        
3. 배포가 완료되면 Netlify에서 주는 주소(예: `https://mybot.netlify.app`)를 복사한다.
    

---

## 7단계: 봇 활성화 (Webhook 설정)

마지막으로 텔레그램에게 "누가 메시지를 보내면 이 주소(Netlify)로 전달해줘!"라고 알려줘야 한다.<br>브라우저 주소창에 아래 형식을 맞춰 입력하고 엔터를 친다.

`https://api.telegram.org/bot<내 토큰>/setWebhook?url=<내 Netlify 주소>/api/telegram`

화면에 `{"ok":true,"result":true,"description":"Webhook was set"}`가 뜨면 성공이다!

---

## ✅ 완료! 이제 어떻게 하면 되나?

1. 텔레그램에서 내가 만든 봇에게 아무 링크를 보낸다.
    
2. 봇이 "인코딩 분석 중..."이라고 답장하고 잠시 후 "저장 완료!"라고 하면 성공이다.
    
3. **Koofr**의 `ReadItLater` 폴더를 확인해 보면, 한글이 깨지지 않은 깔끔한 마크다운 파일이 들어있을 것이다.
    
4. 이 파일을 **옵시디언**에서 열어보면 사진과 함께 정갈하게 정리되어 있는 것을 볼 수 있다.<br>나의 경우 Remotely Save 플러그인을 통해서 URL 스크래핑 이후에 주기적으로 동기화해주고 있다.

![텔레그램 봇에 URL을 붙여넣기만 하면 된다.](20260518061455.png)

![옵시디언에 자동으로 들어오는 md 파일](20260518061921.png)

---
- 2026.05.28 스크래핑 실패 사이트 확인
	- 네이버 블로그 등 보통의 사이트를 스크래핑이 잘 되지만 '[브런치](https://brunch.co.kr/)'는 강력한 보안 및 크롤링 방지 정책이 적용되어 있어 오류가 발생한다.<br>Obsidian Web Clipper로 클리핑 하면 그만이지만, [Oracle Cloud Free Tier](https://www.oracle.com/cloud/)를 받은 만큼 Netlify가 아닌 자체 서버에서 스크래핑 해보는 걸로 했다. 
- 2026.08.04 네이버 블로그, 카페 등 제목에 특수문자가 있는 경우 스크래핑 오류 확인
	- '《'와 같은 특수문자가 들어가 있는 경우 스크래핑에 실패하는 오류를 확인하여 이를 방지하기 위해 코드를 일부 수정했다.
	- 향후 디버깅을 위해 에러 원인도 좀 더 세분화해서 알려주도록 업데이트 했다.