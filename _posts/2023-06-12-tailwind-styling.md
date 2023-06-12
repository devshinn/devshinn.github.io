---
title: Neatly styling tailwindcss with variables
excerpt: tailwindcss styling
tags:
  - react/next
  - tailwindcss
  - twin.macro

categories:
  - react/next
toc_sticky: true
published: true
sitemap:
  changefreq: daily
  priority: 1.0
---
인라인스타일링 할 경우 코드 가독성이 떨어져

twin.macro(styled-components + tailwindcss)를 자주 사용하여 스타일링 하였는데,

nextjs 13버전의 app dir 에서는 `server-component, client-component` 를 `use client`로 명시 해줘야 하는데 , `webpack`이 "use client" 를 삭제 하기때문에 아직까지 twin.macro를 사용할수 없다.

그래서 대충 만들어 사용 하기로 ... 

물론 styled-components에 주입은 어렵지만, tailwindcss만 사용 할때는 나름 괜찮은 것 같다.

### 사용 예시

```ts
import { st } from '@/lib';

import { st } from '@/lib';
export default function Home() {
  return (
    <>
        <div className={exStyle1()}>
          <h1 className={exStyle1_1}> heoll</h1>
          <h1 className={exStyle2({ bg: 'red' })}> heoll</h1>
          <h1 className={exStyle3({ bg: 'red' })}> heoll</h1>
          <h1 className={exStyle4({ bg: true })}> heoll</h1> 
        </div>
    </>
  );
}

// 삼항연산이나 비교 논리연산 사용 가능
const exStyle1 = st`text-gray-400 p-2`;
const exStyle1_1 = st`text-gray-400 p-2`();
const exStyle2 = st<{ bg: string }>`text-gray-400 ${(prop) => prop.bg}`; // or => ${(prop)prop.bg && 'bg-gray-100'}
const exStyle3 = st<{ bg: string }>`text-gray-400 ${(prop) =>  prop.bg === 'red' ? 'bg-red-400' : 'bg-gray-500'}`;
const exStyle4 = st<{ bg: boolean }>`text-gray-400 ${(prop) =>  prop.bg && 'bg-red-100'}`; // 삼항연산도 가능


```

### st.tsx

```ts
// lib/st.tsx

type TemplateValues<T> =
  | ((props: T) => Partial<T> | string | false)
  | Partial<T>
  | string;

export function st<T>(
  strings: TemplateStringsArray,
  ...objs: TemplateValues<T>[]
) {
  const getStyle = (props?: T): string => {
    let textValue = strings.reduce(
      (value: string, current: string, index: number) => {
        return value + current;
      }
    );

    const ifText = objs.map((f) => {
      if (f instanceof Function) {
        const f_value = f(props as T);
        return f_value ? f_value : null;
      } else return '';
    });
    return (combineText(textValue) + ' ' + ifText.join(' ')).trim();
  };

  return (props?: T) => getStyle(props);
}
const combineText =(input: string)=> {
  const lines = input.split('\n'); 
  const result = lines
    .map((line) => line.trim()) 
    .filter((line) => line.length > 0) 
    .join(' '); 

  return result;
}
```

![](https://raw.githubusercontent.com/bradlc/vscode-tailwindcss/master/packages/vscode-tailwindcss/.github/banner.png)

**vsocde 의 Tailwind CSS IntelliSense를 사용하기 위한 설정**

```json
.vscode/settings.json

{
  "scss.validate": false,
  "editor.quickSuggestions": {
    "strings": true
  },
  "tailwindCSS.includeLanguages": {
    "typescript": "javascript",
    "typescriptreact": "javascript"
  },
  "editor.autoClosingQuotes": "always",
  "tailwindCSS.experimental.classRegex": [
    "st`([^`]*)", // tw`...`
    "st<.*?>`([^`]*)`", // tw<...>`...`
  ]
}

```

```

```
