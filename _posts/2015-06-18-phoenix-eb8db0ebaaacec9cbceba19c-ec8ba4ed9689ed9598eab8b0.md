---
author: jeidee
categories:
- elixir
date: '2015-06-18'
guid: https://erlnote.wordpress.com/?p=509
id: 509
permalink: /2015/06/18/phoenix-%eb%8d%b0%eb%aa%ac%ec%9c%bc%eb%a1%9c-%ec%8b%a4%ed%96%89%ed%95%98%ea%b8%b0/
tags:
- phoenix
- web server
title: phoenix 데몬으로 실행하기
url: /2015/06/18/phoenix-eb8db0ebaaacec9cbceba19c-ec8ba4ed9689ed9598eab8b0
---

run.sh파일을 하나 만들고 다음과 같이 입력한다.
  
포트는 원하는 포트로 입력하면 된다.

```
  
MIX_ENV=prod PORT=4001 elixir &#8211;detached -S mix phoenix.server
  
```

## 출처

  * [How to start phoenix server as daemon?](http://stackoverflow.com/questions/30061044/how-to-start-phoenix-server-as-daemon)