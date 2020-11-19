# Sublime Text3에서 Markdown 문서 작성하기


## 들어가며
  
  블로깅을 하기로 하고서 편리한 Markdown Editor를 찾아 헤맸습니다. 굉장히 다양한 Markdown Tool이 있었습니다. 한국인 개발자가 만든 하루 패드(Haroo pad)도 매우 평이 좋았습니다. 어떤 것을 쓸까 고민하던 중에 평소 자주 사용하는 Sublime Text에 Markdown 용 Plugin이 있지 않을까 하는 생각이 들었습니다. 역시나 다양한 Plugin들이 존재했고, 그중 몇 가지를 소개하려 합니다. 
- [나무위키:마크다운](https://namu.wiki/w/%EB%A7%88%ED%81%AC%EB%8B%A4%EC%9A%B4)
- [What is MarkDown?](https://guides.github.com/features/mastering-markdown)

---

## Markdown Editing & Markdown Preview

Markdown Editing은 Markdown 문법으로 문서작성을 할 수 있도록 도와줍니다. 세 가지 문법(Standard Markdown, GitHub flavored Markdown, Multi Markdown) 을 지원하고, Syntax Highlighting과 몇 가지 Color scheme 제공합니다.

Markdown Preview는 작성한 Markdown 문서를 브라우저를 통하여 Preview 할 수 있는 기능을 제공합니다. 

### 사용방법
1. Sublime Text3를 실행합니다.
2. <kbd>Clt</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd>를 누르고 *Package Control: Install Package*를 선택합니다.
3. 각각의 Plugin을 찾아 설치합니다.
    - Markdown Editing
    - Markdown Preview
4. Preferences -> Key Bindings에 단축키를 추가합니다.

```json
[{
    ...
},{
    "keys": ["alt+m"],
    "command": "markdown_preview",
    "args": {
        "target": "browser",
        "parser": "markdown"
    }
}]
```
5. Markdown 문서 작성 후 <kbd>ALT+</kbd>+<kbd>M</kbd>을 눌러 내용을 확인합니다.
6. 하얀색 테마가 마음에 안들면 검은색으로 바꿉시다.
  - 아래 내용을  Preferences -> Package Settings -> Markdown Editing -> Markdown GFM Settings - User에 추가합시다.

```json
{
    //"color_scheme": "Packages/MarkdownEditing/MarkdownEditor.tmTheme",
    "color_scheme": "Packages/MarkdownEditing/MarkdownEditor-Dark.tmTheme",
    // "color_scheme": "Packages/MarkdownEditing/MarkdownEditor-Yellow.tmTheme",
}  
```

- Reference: Markdown Editing & Markdown Preview 설치 관련 [http://cheng.logdown.com/posts/2015/06/30/sublime-text-3-markdown](http://cheng.logdown.com/posts/2015/06/30/sublime-text-3-markdown)

---

## LiveReload(Optional)

Markdown Editing, Markdown Preview만 이용하면 한 가지 단점이 있습니다. 매번 수정사항이 생길 때마다 브라우저에서  F5 버튼을 눌러서 Refresh를 해줘야 한다는 점입니다. 이를 보완하기 위해 개발된 Plugin으로 사용하면 실시간으로 변경사항을 브라우저에서 확인할 수 있습니다. 다만 브라우저 확장 프로그램을 설치해야 한다는 번거로움이 있습니다. 

### 사용방법
1. Sublime Text3를 실행합니다.
2. <kbd>Clt</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd>를 누르고 Package Control: Install Package를 선택합니다.
3. LiveReload plugin을 찾아 설치합니다. (Markdown Editing, Markdown Preview도 설치된 상태여야 합니다)
4. Chrome을 기본 브라우저로 활용하는 경우 확장 프로그램으로 liveReload를 찾아 설치합니다. 
  - [확장 프로그램 설치 링크](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei?hl=ko)
5. Preferences -> Package Settings -> LiveReload -> Setting - User 에 아래 내용을 추가합니다.
```json
{ 
   "enabled_plugins": [ 
       "SimpleReloadPlugin", 
       "SimpleRefresh" 
   ]
}
```

6. <kbd>Alt</kbd>+<kbd>M</kbd>를 눌러 Preview를 브라우저에 띄워놓은 채로 내용을 수정한 뒤 저장(<kbd>Clt</kbd>+<kbd>S</kbd>)해봅시다. 새로고침 없이도 수정사항이 바로 반영됨을 확인할 수 있습니다.
* 확장자를 .md로 저장하지 않으면 동작하지 않습니다. 

- Reference: [http://stackoverflow.com/questions/25886011/how-do-i-install-livereload-on-sublime-text-3](http://stackoverflow.com/questions/25886011/how-do-i-install-livereload-on-sublime-text-3)

---

## Markdown Live Preview

Markdown Preview의 대안으로, 결과물을 Sublime Text 자체에서 창을 Split 하여 보여줍니다. 게다가 실시간으로 변경사항이 반영됩니다. 사용해본 결과 굉장히 편리하나 개인적으로는 위에서 언급한 방법을 선호합니다. 각자 취향에 맞게 사용하면 좋을 것 같습니다.

1. Sublime Text3를 실행합니다.
2. <kbd>Clt</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd>를 누르고 Package Control: Install Package를 선택합니다.
3. 각각의 Plugin을 찾아 설치합니다.
    - Markdown Editing
    - MarkdownLivePreview
4. <kbd>ALT</kbd>+<kbd>M</kbd>을 누르면 새창이 열리고 Split된 화면에서 결과물을 실시간으로 확인 할 수 있습니다.

- Reference: [https://github.com/math2001/MarkdownLivePreview](https://github.com/math2001/MarkdownLivePreview)

---

끝.
