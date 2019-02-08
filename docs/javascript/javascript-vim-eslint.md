---
layout: default
title: vim eslint 적용하기
parent: Javascript
nav_order: 3
---

# vim eslint 적용하기

주력으로 자바스크립트(ECMA6) 사용하는 개발자라면 eslint를 global로 설치하는게 편합니다.

관련 모듈을 설치해 줍니다.

``` bash
$ npm install --global eslint \
                       eslint-config-airbnb \
                       babel-eslint \
                       eslint-plugin-react
```

.vimrc에 다음과 같이 추가해줍니다. (없으면 만들어서 추가해줍니다. vim ~/.vimrc)

``` bash
let g:syntastic_mode_map = { 'mode': 'active',
                            \ 'active_filetypes': ['python', 'javascript'],
                            \ 'passive_filetypes': [] }

set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*

let g:syntastic_always_populate_loc_list = 1
let g:syntastic_auto_loc_list = 1
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0
let g:syntastic_javascript_checkers = ['eslint']
```

.eslintrc.json에 다음내용들이 포함되어 있는지 확인합니다(버전에 따라 모듈이름이 다를 수 있습니다.)

``` bash
{
  "extends": "airbnb-base",
  "parser": "babel-eslint",
  "plugins": ["react", "import"]
}
```
