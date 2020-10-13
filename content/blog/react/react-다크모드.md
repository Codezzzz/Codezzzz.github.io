---
title: react 다크모드
date: 2020-10-13 14:10:33
category: react
thumbnail: './images/React.png'
draft: false
---

![](./images/React.png)

# React custom hook (다크모드 적용)

## Hook만들기

```ts
import React, { useState } from 'react'

type returnType = [string, () => void]

const useTheme = (): returnType => {
  // 브라우저 테마 정보 확인
  const isBrowserDarkMode =
    window.matchMedia &&
    window.matchMedia('(prefers-color-scheme: dark)').matches

  let initTheme = isBrowserDarkMode ? 'dark' : 'light'

  const localSettingTheme = localStorage.getItem('theme')

  if (localSettingTheme) {
    initTheme = localSettingTheme
  }

  const [theme, setTheme] = useState(initTheme)

  const setMode = (mode: string) => {
    window.localStorage.setItem('theme', mode)
    setTheme(mode)
  }

  const toggleTheme = () => setMode(theme === 'light' ? 'dark' : 'light')

  return [theme, toggleTheme]
}

export default useTheme
```

## 사용하기

```ts
function App() {
  const [themeMode, toggleTheme] = useTheme()

  const theme = themeMode === 'light' ? light : dark

  return (

    <AppProvider>
      <ThemeProvider theme={theme}>
        <Main className="App">
        <button onClick={toggleTheme}>ToggleTheme</button>
      </ThemeProvider>
      </Main>
    </AppProvider>

  );
}

export default App;


const Main = styled.div`
background-color : ${ ({theme} : ThemeInterface) => theme.colors.bgColor };

font-size : 2rem;

& .toggleColor {
  color: ${ ({theme} : ThemeInterface) => theme.colors.titleColor };
}
```
