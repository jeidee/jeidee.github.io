---
author: jeidee
categories:
- ios
date: '2015-05-19'
guid: https://erlnote.wordpress.com/?p=398
id: 398
permalink: /2015/05/19/ios-uiimageview%eb%a5%bc-%ec%9b%90%ed%98%95%ec%9c%bc%eb%a1%9c-%ec%b6%9c%eb%a0%a5%ed%95%98%ea%b8%b0/
tags:
- UIImageView
title: ios UIImageView를 원형으로 출력하기
url: /2015/05/19/ios-uiimagevieweba5bc-ec9b90ed9895ec9cbceba19c-ecb69ceba0a5ed9598eab8b0
---

```objc
          
cell.imgAvatar.layer.cornerRadius = cell.imgAvatar.frame.size.height / 2;
          
cell.imgAvatar.layer.masksToBounds = YES;
          
cell.imgAvatar.layer.borderWidth = 0;

```

테이블셀의 UIImaveView인 imgAvatar를 원형으로 출력하는 코드이다.

## 출처

  * [UIImage in a circle](http://stackoverflow.com/questions/4414221/uiimage-in-a-circle)