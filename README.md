# UserActivity-iOS
How to use ?

## 1. Provider
CXProviderConfiguration
Will support => [NSNumber numberWithInteger:CXHandleTypeGeneric]
```Objective-C
NSDictionary* infoDict =[[NSBundle mainBundle] infoDictionary];
    NSString *localizedName = [infoDict objectForKey:@"CFBundleName"];//APP Name
    CXProviderConfiguration *configuration = [[CXProviderConfiguration alloc] initWithLocalizedName:localizedName];
    configuration.supportsVideo = YES;
    configuration.maximumCallsPerCallGroup = 1;
    configuration.supportedHandleTypes = [NSSet setWithObjects:[NSNumber numberWithInteger:CXHandleTypeGeneric], [NSNumber numberWithInteger:CXHandleTypePhoneNumber], nil];
```

## 2. Info.Plist
Info.plist
```Objective-C
<key>NSUserActivityTypes</key>
<array>
	<string>INStartAudioCallIntent</string>
	<string>INStartVideoCallIntent</string>
</array>
```

## 3.AppDelegate
AppDelegate.h
```Objective-C
-(BOOL)application:(UIApplication *)application willContinueUserActivityWithType:(NSString *)userActivityType{
    return YES;
}


- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler{
    

        if (![userActivity.activityType isEqualToString:@"INStartVideoCallIntent"] && ![userActivity.activityType isEqualToString:@"INStartAudioCallIntent"]) {
            return NO;
        }
        
        BOOL isVideo = NO;
        NSString *phoneNumber = @"";
        if ([userActivity.activityType isEqualToString:@"INStartVideoCallIntent"]) {
            isVideo = YES;
            
            INInteraction *interaction = userActivity.interaction;
            INStartVideoCallIntent *startAudioCallIntent = (INStartVideoCallIntent *)interaction.intent;
            INPerson *contact = startAudioCallIntent.contacts[0];
            INPersonHandle *personHandle = contact.personHandle;
            
            phoneNumber = personHandle.value;
            
            NSArray * tmpStrings = [phoneNumber componentsSeparatedByString:@":"];
            if(tmpStrings.count > 1){
                NSArray * tmpString2 = [tmpStrings[1] componentsSeparatedByString:@"@"];
                if(tmpString2.count > 0){
                    phoneNumber = tmpString2[0];
                }
            }
            
        }else{
            
            INInteraction *interaction = userActivity.interaction;
            INStartAudioCallIntent *startAudioCallIntent = (INStartAudioCallIntent *)interaction.intent;
            INPerson *contact = startAudioCallIntent.contacts[0];
            INPersonHandle *personHandle = contact.personHandle;
            
            phoneNumber = personHandle.value;
            
            NSArray * tmpStrings = [phoneNumber componentsSeparatedByString:@":"];
            if(tmpStrings.count > 1){
                NSArray * tmpString2 = [tmpStrings[1] componentsSeparatedByString:@"@"];
                if(tmpString2.count > 0){
                    phoneNumber = tmpString2[0];
                }
            }
            
        }
    
    NSDictionary * mucontact = [[ServiceModel sharedInstance] getContactFromPhoneNumber:phoneNumber];
    
    if(mucontact != nil){
        NSString * phNumber = [NSString stringWithFormat:@"%@", [mucontact objectForKey:@"phoneNumber"]];
        if(phNumber.length > 0){
            UIStoryboard * strb = [UIStoryboard storyboardWithName:@"Main" bundle:nil];
            self.curDialViewController = [strb instantiateViewControllerWithIdentifier:@"EzyDialViewController"];
            [self.curDialViewController setMuContact:nil];
            
            [self.window addSubview:self.curDialViewController.view];
        }
    }
        
      
    return YES;
    
}

```
