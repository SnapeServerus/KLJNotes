## watch 小记
### 前言
上帝说：“要有光“，于是就有了光。领导说：”要支持apple watch“，于是。。。
在目前这种情况下，watch仅仅是watch，如果没有一种成熟的颠覆性的技术辅助它，那么它将很难取代手机，所以我本身对让一个消息接收的app支持apple watch的决定持保留意见。然而在技术面前，暂时可以抛掉这些，程序员会关心苹果又公开了watch的哪些API，又有那些酷炫的功能可以实现，那现在来简单关心一下。
### watch os 2 要点
* 灵活使用group来进行界面上的排版，完成设计稿的内容
* 可以使用NSURLSession进行网络访问
* watch extension在watchos2之前是存储在手机端，所以可以通过打开App Groups来共享文件，而在watchos2上extension是存储在watch一端。
* 通过WatchConnectivityFramework可以进行手机与手表上的双向通信,而且通过这个framework可以从watch端直接唤醒手机端的app，所以这里自然而然就可以想到把iPhone当成是服务器，Apple Watch当成客户端，当然如果要实现对数据实时性要求不高的需求或者说是有信息从iPhone推送至Apple Watch，这个框架也提供了相对应的API，官方文档宣称会在合适的时机对这部分数据进行传输-。-
  * 具体的学习方法当然需要阅读官方文档以及头文件了，两者都已经写详细，这里不做赘述
  * 在阅读头文件的时候发现一些API是有一些前提条件的，例如：
```
/** -------------------------- Interactive Messaging ---------------------------
 *  Interactive messages can only be sent between two actively running apps.
 *  They require the counterpart app to be reachable.
 */
 
 /** --------------------------- Background Transfers ---------------------------
 *  Background transfers continue transferring when the sending app exits. The
 *  counterpart app (other side) is not required to be running for background
 *  transfers to continue. The system will transfer content at opportune times.
 */
 
 
```

* 所以参考了一下资料，对这部分进行一下简单的封装：
```

//
//  ALTWatchSessionManager.m
//  Alertover
//
//  Created by BJY on 16/3/2.
//  Copyright © 2016年 BJY. All rights reserved.
//

#import "ALTWatchSessionManager.h"
#import <WatchConnectivity/WatchConnectivity.h>

NS_ASSUME_NONNULL_BEGIN

@interface ALTWatchSessionManager () <WCSessionDelegate>

@property (nonatomic, strong) WCSession *session;

@end

@implementation ALTWatchSessionManager

#pragma mark - Singleton

+ (instancetype)sharedInstance {
    static ALTWatchSessionManager *instance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[ALTWatchSessionManager alloc] init];
    });
      
    return instance;
  }

- (void)startSession {
    if ([WCSession isSupported]) {
        self.session = [WCSession defaultSession];
        self.session.delegate = self;
        [self.session activateSession];
    }
  }

#pragma mark - Watch Connectivity Interactive sender

- (void)sendMessage:(NSDictionary<NSString *, id> *)message replyHandler:(nullable void (^)(NSDictionary<NSString *, id> *replyMessage))replyHandler errorHandler:(nullable void (^)(NSError *error))errorHandler {
      
    if (![self validReachableSession]) {
        return ;
    }
      
    [[self validReachableSession] sendMessage:message replyHandler:replyHandler errorHandler:errorHandler];
    
}

- (void)sendMessageData:(NSData *)data replyHandler:(nullable void (^)(NSData *replyMessageData))replyHandler errorHandler:(nullable void (^)(NSError *error))errorHandler {
      
    if (![self validReachableSession]) {
        return ;
    }
      
    [[self validReachableSession] sendMessageData:data replyHandler:replyHandler errorHandler:errorHandler];
  }

#pragma mark - Watch Connectivity background sender

- (BOOL)updateApplicationContext:(NSDictionary<NSString *, id> *)applicationContext error:(NSError **)error {
      
    return [_session updateApplicationContext:applicationContext error:error];
    
}

- (WCSessionUserInfoTransfer *)transferUserInfo:(NSDictionary<NSString *, id> *)userInfo {
      
    return [_session transferUserInfo:userInfo];
    
}

- (WCSessionFileTransfer *)transferFile:(NSURL *)file metadata:(nullable NSDictionary<NSString *, id> *)metadata {
      
    return [_session transferFile:file metadata:metadata];
  }

#pragma mark - WCSessionDelegate

- (void)session:(WCSession *)session didReceiveMessage:(NSDictionary<NSString *, id> *)message replyHandler:(nonnull void (^)(NSDictionary<NSString *,id> * _Nonnull))replyHandler {
    
    dispatch_async(dispatch_get_main_queue(), ^{
       
    });
    
}

- (void)session:(WCSession *)session didReceiveMessageData:(NSData *)messageData replyHandler:(nonnull void (^)(NSData * _Nonnull))replyHandler {
      
    dispatch_async(dispatch_get_main_queue(), ^{
          
    });
    
}

#pragma mark - validReachableSession getter

- (WCSession *)validReachableSession {
    if ([_session isReachable] && _session.paired && _session.watchAppInstalled) {
        return _session;
    }
      
    return nil;
  }

@end


NS_ASSUME_NONNULL_END
```

### 参考资料
* https://github.com/ipader/SwiftGuide/tree/master/Apple%20Watch
* http://www.kristinathai.com/watchos-2-how-to-communicate-between-devices-using-watch-connectivity/
* http://onevcat.com/2015/08/watchos2/
* https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/WatchKitProgrammingGuide/SharingData.html#//apple_ref/doc/uid/TP40014969-CH29-SW1

### 随便聊聊
一个程序员在学习到新技术或者说解决了一个问题总会有抑制不住的兴奋，然而真正令他们感觉到成就感的还是在他们能够对一个真正成熟有用的产品作出贡献，产品作出的正向反馈会让他们更加有动力深究技术，从而形成良性循环。我还没有到那个阶段，也不知道什么时候能完成真正的进阶。