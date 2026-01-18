# MD to PDF 변환 팁

md-to-pdf를 사용하여 마크다운을 PDF로 변환할 때 발생하는 문제들과 해결 방법을 정리합니다.

---

## 1. MathJax 3 설정 (필수)

YAML frontmatter에 다음 설정을 추가합니다:

```yaml
---
script:
  - content: |
      window.MathJax = {
        tex: {
          inlineMath: [['$', '$'], ['\\(', '\\)']],
          displayMath: [['$$', '$$'], ['\\[', '\\]']],
          processEscapes: true,
          processEnvironments: true
        },
        svg: {
          fontCache: 'global'
        },
        startup: {
          pageReady: function() {
            return MathJax.startup.defaultPageReady().then(function() {
              console.log('MathJax rendering complete');
            });
          }
        }
      };
  - url: https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-svg.js
css: |-
  body { font-size: 18px; line-height: 1.8; }
  code, pre { font-size: 14px; }
---
```

**중요**: `startup.pageReady` 함수를 반드시 포함해야 합니다. 이 함수가 없으면 MathJax가 렌더링을 완료하기 전에 PDF가 생성됩니다.

---

## 2. `\tilde` 명령어 문제

### 문제
`\tilde{x}` 명령어가 작동하지 않습니다. `\t`가 탭 문자로 해석될 수 있기 때문입니다.

### 해결책
`\tilde` 대신 `\widetilde` 사용:

```latex
# 안 됨
$$\tilde{g}_B$$

# 됨
$$\widetilde{g}_B$$
```

---

## 3. 언더스코어 이스케이프 (가장 중요!)

### 문제
수식 내에 여러 개의 언더스코어(`_`)가 있으면, 마크다운 파서가 이를 이탤릭 마커로 해석하여 수식이 깨집니다.

예를 들어:
```latex
$$\widetilde{g}_B = \sum_{i=1}^{B}$$
```
위 수식에서 `_B = \sum_` 부분이 `_..._` 이탤릭으로 해석됩니다.

### 해결책
**두 개 이상의 언더스코어가 있는 수식**에서는 언더스코어를 `\_`로 이스케이프합니다:

```latex
# 안 됨 (깨짐)
$$\widetilde{g}_B = \frac{1}{B} \sum_{i=1}^{B} \widetilde{g}^{(i)}$$

# 됨 (작동)
$$\widetilde{g}\_B = \frac{1}{B} \sum\_{i=1}^{B} \widetilde{g}^{(i)}$$
```

### 이스케이프가 필요한 경우
- 같은 수식 내에 `_`가 2개 이상 있을 때
- 특히 `}_`와 `_{` 패턴이 함께 있을 때
- `\underbrace{}_{}`와 같은 패턴

### 이스케이프가 필요 없는 경우
- 수식 내에 `_`가 1개만 있을 때
- 예: `$E[\widetilde{g}_B] = g$` (작동함)

---

## 4. 틸드(~) 문자 문제

### 문제
숫자 사이의 `~`가 취소선(`~~text~~`)으로 해석되어 텍스트가 사라집니다.

```markdown
# 안 됨 - "07과 815"로 표시됨
(0~7과 8~15)

# 됨
(0-7과 8-15)
```

### 해결책
숫자 범위를 나타낼 때 `~` 대신 `-` 사용

---

## 5. 폰트 크기 권장 설정

```css
body { font-size: 18px; line-height: 1.8; }
code, pre { font-size: 14px; }
table:first-of-type { font-size: 14px; }  /* 첫 번째 표 (표지용) */
table:not(:first-of-type) { font-size: 10px; }  /* 나머지 표 (데이터용) */
```

- **본문**: 18px (가독성 좋음)
- **코드**: 14px (본문보다 작아야 시각적 계층 명확)
- **표지 표**: 14px
- **데이터 표**: 10px (넓은 표 대응)

---

## 6. 넓은 표 처리

표가 페이지를 넘어갈 때:

```css
table:not(:first-of-type) {
  font-size: 10px !important;
  width: 100% !important;
  table-layout: fixed;
  word-wrap: break-word;
}
table:not(:first-of-type) td, table:not(:first-of-type) th {
  font-size: 10px !important;
  line-height: 1.4 !important;
  padding: 4px !important;
}
```

---

## 7. 첫 페이지 스타일링

```css
/* 제목 (여러 줄일 경우) */
h1:first-of-type, h1:nth-of-type(2) {
  text-align: center;
  line-height: 1.8;
  font-size: 38px;
}
h1:nth-of-type(2) { margin-bottom: 40px; }

/* 이미지 간격 */
img:first-of-type { margin-top: 20px; margin-bottom: 50px; }

/* 부제목 (이탤릭) */
em:first-of-type { display: block; text-align: center; margin-bottom: 30px; }

/* 첫 번째 표 */
table:first-of-type { font-size: 14px; margin-bottom: 40px; }
```

---

## 8. 문제 해결 체크리스트

수식이 깨질 때 확인할 사항:

1. **MathJax 설정 확인**: frontmatter에 MathJax 3 설정이 있는가?
2. **pageReady 함수 확인**: startup.pageReady 함수가 포함되어 있는가?
3. **\tilde 사용 여부**: `\tilde`를 `\widetilde`로 변경했는가?
4. **언더스코어 개수**: 수식에 `_`가 2개 이상이면 `\_`로 이스케이프했는가?
5. **틸드 문자**: 숫자 범위에 `~` 대신 `-`를 사용했는가?

---

## 9. 사용 명령어

```bash
# md-to-pdf 설치
npm install -g md-to-pdf

# PDF 변환
md-to-pdf "파일명.md"
```

---

## 10. 전체 예시

```markdown
---
script:
  - content: |
      window.MathJax = {
        tex: {
          inlineMath: [['$', '$'], ['\\(', '\\)']],
          displayMath: [['$$', '$$'], ['\\[', '\\]']],
          processEscapes: true,
          processEnvironments: true
        },
        svg: { fontCache: 'global' },
        startup: {
          pageReady: function() {
            return MathJax.startup.defaultPageReady();
          }
        }
      };
  - url: https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-svg.js
css: |-
  body { font-size: 18px !important; line-height: 1.8 !important; }
  code, pre { font-size: 14px !important; }
  h1:first-of-type, h1:nth-of-type(2) { text-align: center; font-size: 38px; }
  table:first-of-type { font-size: 14px; }
  table:not(:first-of-type) { font-size: 10px !important; width: 100% !important; }
---

# 제목

인라인 수식: $E = mc^2$

디스플레이 수식:
$$\widetilde{g}\_B = \frac{1}{B} \sum\_{i=1}^{B} \widetilde{g}^{(i)}$$

숫자 범위: 0-7과 8-15
```
