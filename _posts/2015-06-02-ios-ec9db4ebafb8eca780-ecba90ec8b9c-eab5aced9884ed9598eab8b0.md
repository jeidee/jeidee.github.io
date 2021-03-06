---
author: jeidee
categories:
- ios
date: '2015-06-02'
guid: https://erlnote.wordpress.com/?p=408
id: 408
permalink: /2015/06/02/ios-%ec%9d%b4%eb%af%b8%ec%a7%80-%ec%ba%90%ec%8b%9c-%ea%b5%ac%ed%98%84%ed%95%98%ea%b8%b0/
tags:
- cache
- image cache
- md5
- NSURL
- thumbnail
title: ios 이미지 캐시 구현하기
url: /2015/06/02/ios-ec9db4ebafb8eca780-ecba90ec8b9c-eab5aced9884ed9598eab8b0
---

URL에서 이미지를 가져와 출력하는 것은 매우 성능이 좋지 않다. 상황에 따라서는 수 초간(또는 그 이상) 이미지를 불러오지 못하는 경우도 생길 수 있고, 비동기로 많은 이미지를 가져올 경우 상황은 더욱 좋지 않게 된다.

이런 경우 대부분 이미지를 캐시해 놓고 이후 요청에서는 캐시에서 이미지를 불러오도록 하는데, 메모리와 디스크를 병행해서 캐시해야 프로세스를 재시작해도 캐시를 지속적으로 사용할 수 있는 이득을 볼 수 있다.

간단한 플로우는 다음과 같다.

1) 메모리 캐시에서 이미지를 검색한다.
  
2) 없을 경우, 디스크 캐시에서 이미지를 검색한다.
  
3) 없을 경우, URL에서 이미지를 비동기 로드한다.
  
4) 메모리 캐시와 디스크 캐시에 해당 이미지를 저장한다.
  
5) 다음 번 요청시에는 메모리 캐시에서 이미지를 불러온다.
  
6) 프로세스 재지작 이후 요청시에는 디스크 캐시에서 불러온 후 메모리 캐시에 추가한다.

캐시키는 URL 문자열을 md5 해시 문자열로 변환해 사용한다.

소스코드는 다음과 같고 캐시의 최대 사이즈 제한 기능이 빠져있으므로 실제로 사용할 경우 해당 기능을 추가 구현해야 한다.

ImageCache.h

```objc
  
#import <UIKit/UIKit.h>

@interface ImageCache : NSObject
  
{
      
NSMutableDictionary* mMemCache;
  
}

+(ImageCache*)getInstance;

-(void) loadFromUrl: (NSURL\*) url callback:(void (^)(UIImage \*image))callback;

-(NSString\*)md5:(NSString \*)str;

-(UIImage\*)saveToDisk:(UIImage\*)image withKey:(NSString*)key;
  
-(UIImage\*)loadFromDisk:(NSString\*)key;

-(UIImage\*)saveToMemory:(UIImage\*)image withKey:(NSString*)key;
  
-(UIImage\*)loadFromMemory:(NSString\*)key;

-(UIImage\*)makeThumbnail:(UIImage\*)image;

@end
  
```

ImageCache.m

```objc
  
#import "ImageCache.h"
  
#import <CommonCrypto/CommonDigest.h>

@implementation ImageCache

// 싱글턴
  
+(ImageCache*)getInstance {
      
static dispatch\_once\_t pred;
      
static ImageCache* instance = nil;

dispatch_once(&pred, ^{
          
instance = [[ImageCache alloc] init];
      
});

return instance;
  
}

-(id)init {
      
self = [super init];

if (self) {
          
// url 캐시 설정
          
// 메모리 : 10MB, 디스크 : 50MB
          
NSURLCache \*urlCache = [[NSURLCache alloc] initWithMemoryCapacity:10 \* 1024 \* 1024 diskCapacity:50 \* 1024 * 1024 diskPath:nil];
          
[NSURLCache setSharedURLCache:urlCache];

// 메모리 캐시 설정
          
// @todo : 최대 사이즈와 적정 사이즈 조절 기능 필요
          
mMemCache = [[NSMutableDictionary alloc] initWithCapacity:100];
      
}

return self;
  
}

&#8211; (void) loadFromUrl: (NSURL\*) url callback:(void (^)(UIImage \*image))callback {
      
dispatch\_queue\_t queue = dispatch\_get\_global\_queue(DISPATCH\_QUEUE\_PRIORITY\_HIGH, 0ul);
      
dispatch_async(queue, ^{

UIImage* image = nil;

if (url != nil) {
              
NSString* key = [self md5:[url absoluteString]];

// 메모리 캐시에서 먼저 검색
              
UIImage* cachedImage = [self loadFromMemory:key];

// 메모리 캐시에 없을 경우 디스크 캐시에서 검색
              
if (cachedImage == nil) {
                  
cachedImage = [self loadFromDisk:key];
                  
// 메모리 캐시에 추가
                  
[self saveToMemory:cachedImage withKey:key];
              
}

// 캐시된 이미지가 없을 경우 url에서 직접 가져옴
              
if (cachedImage != nil) {
                  
image = cachedImage;
              
} else {
                  
NSData * imageData = [NSData dataWithContentsOfURL:url];
                  
image = [UIImage imageWithData:imageData];
                  
// 썸네일 이미지로 변환
                  
image = [self makeThumbnail:image];

// 메모리와 디스크 캐시에 추가
                  
[self saveToMemory:image withKey:key];
                  
[self saveToDisk:image withKey:key];
              
}
          
}

if (image == nil) {
              
image = [UIImage imageNamed:@"unnamed"];
          
}

dispatch\_async(dispatch\_get\_main\_queue(), ^{
              
callback(image);
          
});
      
});
  
}

// url을 사용해서 md5 해시 문자열 생성
  
-(NSString\*)md5:(NSString \*)str {
      
const char *cStr = [str UTF8String];
      
unsigned char result[CC\_MD5\_DIGEST_LENGTH];
      
CC_MD5( cStr, strlen(cStr), result );
      
return [NSString stringWithFormat:@"%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X",
              
result[0], result[1],
              
result[2], result[3],
              
result[4], result[5],
              
result[6], result[7],
              
result[8], result[9],
              
result[10], result[11],
              
result[12], result[13],
              
result[14], result[15]
              
];
  
}

-(UIImage\*)saveToDisk:(UIImage\*)image withKey:(NSString*)key {
      
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
      
NSString *documentsDirectory = [paths objectAtIndex:0];

NSString *path = [NSString stringWithFormat:@"%@/%@", documentsDirectory, key];

UIImage* thumbnail = [self makeThumbnail:image];

[UIImagePNGRepresentation(thumbnail) writeToFile:path atomically:YES];

return thumbnail;
  
}

-(UIImage\*)loadFromDisk:(NSString\*)key {
      
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
      
NSString *documentsDirectory = [paths objectAtIndex:0];

NSString *path = [NSString stringWithFormat:@"%@/%@", documentsDirectory, key];

UIImage* image = [[UIImage alloc] initWithContentsOfFile:path];
      
return image;
  
}

-(UIImage\*)loadFromMemory:(NSString \*)key {
      
if (mMemCache == nil) return nil;

UIImage* cachedImage = [mMemCache objectForKey:key];
      
return cachedImage;
  
}

-(UIImage\*)saveToMemory:(UIImage\*)image withKey:(NSString*)key {
      
if (mMemCache == nil) {
          
mMemCache = [[NSMutableDictionary alloc] initWithCapacity:100];
      
}

UIImage* thumbnail = [self makeThumbnail:image];

[mMemCache setObject:thumbnail forKey:key];

return thumbnail;
  
}

-(UIImage\*)makeThumbnail:(UIImage\*)image {
      
// 썸네일을 먼저 만들어서 저장한다.
      
// @todo: 썸네일 이미지 사이즈는 별도 정책에 따를 것!
      
CGSize destSize = CGSizeMake(150.0f, 150.0f);
      
UIGraphicsBeginImageContext(destSize);
      
[image drawInRect:CGRectMake(0, 0, destSize.width, destSize.height)];

UIImage* thumbnail = UIGraphicsGetImageFromCurrentImageContext();

UIGraphicsEndImageContext();

return thumbnail;
  
}

@end
  
```