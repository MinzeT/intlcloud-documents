Unity 개발자가 Tencent Cloud GME API를 디버깅하고 연결하는 데 도움이 되도록 여기에 Unity 개발에 적합한 액세스 기술 문서를 소개합니다.

>?이 문서에 해당하는 GME SDK 버전은 2.4입니다.
## 사용 프로세스도
### 음성 채팅 프로세스도
![](https://main.qcloudimg.com/raw/bf2993148e4783caf331e6ffd5cec661.png)
### 오프라인 음성의 문자로 전환 프로세스도
![](https://main.qcloudimg.com/raw/4c875d05cd2b4eaefba676d2e4fc031d.png)


### GME 사용 중요사항

|중요 API     | API 의미|
| ------------- |:-------------:|
|Init    		|GME 초기화 	|
|Poll    		|이벤트 콜백 트리거	|
|EnterRoom	 	|방 입장  		|
|EnableMic	 	|마이크 켜기 	|
|EnableSpeaker		|스피커 켜기 	|

>**설명: **
- GME의 API 호출이 성공한 후 반환 값은 QAVError.OK이고 수치는 0입니다.
- GME의 API는 같은 스레드에서 호출되어야 합니다.
- GME가 방을 가입하려면 인증이 필요합니다. 인증 관련 내용을 참조하십시오.
- GME는 Poll API를 주기적으로 호출하여 이벤트 콜백을 트리거해야 합니다.
- GME 콜백 메시지는 콜백 메시지 리스트를 참조하십시오.
- 방에 입장한 후에 장치에 대한 조작을 실행할 수 있습니다.
- 이 문서에 해당하는 GME SDK 버전은 2.3입니다.


## 초기화 관련 API
초기화 전에 SDK는 미초기화 단계에 속하고 있습니다. 초기화 인증 후 SDK를 초기화해야 방에 입장할 수 있습니다.

|API     | API 의미   |
| ------------- |:-------------:|
|Init    	|GME 초기화	|
|Poll    	|이벤트 콜백 트리거	|
|Pause   	|시스템 일시 중지	|
|Resume 	|시스템 다시 시작	|
|Uninit    	|GME 반초기화 	|

### 인스턴스 획득
ITMGContext 방법을 사용해 Context 인스턴스를 획득하십시오. QAVContext.GetInstance()를 직접 사용해 인스턴스를 획득하지 마십시오.

### SDK 초기화
매개변수 획득은 [액세스 가이드](https://cloud.tencent.com/document/product/607/10782)를 참조하십시오.
해당 API는 Tencent Cloud 콘솔의 SdkAppId 번호를 매개변수로 사용하고 여기에 openId를 더해야 합니다. 해당 openId는 하나의 사용자를 유일하게 식별하는 것으로 규칙은 App 개발자가 자체적으로 제정하고 App 내에서 중복되지 않으면 됩니다(현재는 INT64만을 지원합니다).
SDK 초기화 후에만 방에 들어갈 수 있습니다.

#### 함수 프로토타입
```
ITMGContext Init(string sdkAppID, string openID)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| SDKAppID    	|String  |Tencent Cloud 콘솔의 sdkAppID 번호						|
| openID    	|String  |OpenID는 Int64 유형(string으로 전환 후 가져오기)만 지원하고 반드시 10000보다 크며 사용자 ID 식별에 사용합니다.	|

#### 샘플 코드  
```
int ret = ITMGContext.GetInstance().Init(str_appId, str_userId);
	if (ret != QAVError.OK) {
		return;
	}
```

### 이벤트 콜백 트리거
update에서의 주기적인 Poll 호출을 통해 이벤트 콜백을 트리거할 수 있습니다.
#### 함수 프로토타입

```
ITMGContext public abstract int Poll();
```
### 시스템 일시 중지
시스템에 일시 중지가 발생하는 동시에 엔진에 Pause 진행을 알려야 합니다.
#### 함수 프로토타입

```
ITMGContext public abstract int Pause()
```

### 시스템 복원
시스템 이벤트가 다시 시작되면 엔진에도 동시에 Resume 진행을 알려야 합니다.
#### 함수 프로토타입

```
ITMGContext  public abstract int Resume()
```






### SDK 반초기화
SDK 반초기화, 미초기화 상태에 들어갑니다. 계정을 변경할 경우 반초기화를 진행해야 합니다.
#### 함수 프로토타입

```
ITMGContext public abstract int Uninit()
```





## 음성 채팅 방 관련 API
초기화 후, SDK로 방 입장을 호출하여 방에 입장해야만 음성 채팅 통화를 진행할 수 있습니다.

|API     | API 의미   |
| ------------- |:-------------:|
|GenAuthBuffer    	|인증 초기화|
|EnterRoom   		|방 가입|
|EnterTeamRoom   	|팀 음성 방 가입|
|IsRoomEntered   	|방 입장 여부|
|ExitRoom 		|방 퇴장|
|ChangeRoomType 	|사용자 방 오디오 유형 수정|
|GetRoomType 		|사용자 방 오디오 유형 획득|



### 인증 정보
AuthBuffer를 생성하고 관련 기능의 암호화와 인증에 사용됩니다. 백 엔드 배포에 대한 세부 정보는 [인증 키](https://cloud.tencent.com/document/product/607/12218)를 참조하십시오.    
- 오프라인 음성이 인증을 획득할 때, 방 번호 매개변수는 null을 입력해야 합니다.
- 해당 API 반환 값은 Byte[] 유형입니다.

#### 함수 프로토타입
```
QAVAuthBuffer GenAuthBuffer(int appId, string roomId, string openId, string key)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| appId    		|int   		|Tencent Cloud 콘솔의 SDKAppID 번호		|
| roomId    		|string   		|방 번호, 최대 127자 지원(오프라인 음성 방 번호 매개변수는 null을 입력해야 함)|
| openId    	|String 	|사용자 ID					|
| key    		|string 	|Tencent Cloud [콘솔](https://console.cloud.tencent.com/gamegme)의 키				|



#### 샘플 코드  
```
byte[] GetAuthBuffer(string appId, string userId, string roomId)
    {
	return QAVAuthBuffer.GenAuthBuffer(int.Parse(appId), roomId, userId, "a495dca2482589e9");
}
```

### 방 입장
생성한 인증 정보를 사용해 방에 진입합니다. 방 가입 시 기본적으로 마이크 및 스피커를 켜지 않습니다. 방 입장 시간이 30초가 초과되면 콜백됩니다.

범위 음성 액세스 과정은 [범위 음성](https://cloud.tencent.com/document/product/607/17972)을 참조하십시오.


#### 함수 프로토타입
```
ITMGContext EnterRoom(string roomId, int roomType, byte[] authBuffer)
```

|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| roomId		|string    	|방 번호, 최대 127자 지원|
| roomType 	|ITMGRoomType		|방 오디오 유형		|
| authBuffer 	|Byte[] 	|인증 번호					|

방 오디오 유형은 [음질 선택](https://cloud.tencent.com/document/product/607/18522)을 참조하십시오.

#### 샘플 코드  

```
ITMGContext.GetInstance().EnterRoom(roomId, ITMG_ROOM_TYPE_FLUENCY, authBuffer);
```

### 방 가입 이벤트의 콜백
방 가입 후 위탁 함수를 통해 콜백해야 합니다. 그 중 result 및 error_info 2개의 정보가 포함되어 있습니다.
#### 함수 프로토타입
```
위탁 함수:
public delegate void QAVEnterRoomComplete(int result, string error_info);
이벤트 함수:
public abstract event QAVEnterRoomComplete OnEnterRoomCompleteEvent;
```

#### 샘플 코드
```
이벤트에 대해 수신:
ITMGContext.GetInstance().OnEnterRoomCompleteEvent += new QAVEnterRoomComplete(OnEnterRoomComplete);

수신후의 처리:
void OnEnterRoomComplete(int err, string errInfo)
    {
	if (err != 0) {
	    ShowWarnning (string.Format ("join room failed, err:{0}, errInfo:{1}", err, errInfo));
	    return;
	}else{
	    ShowWarnning (string.Format ("현재 음성 방 ID:{0}, 다른 터미널에서 해당 방에 가입해 통화하십시오",roomId ));
    }
}
```

### 방 입장 여부 판단
해당 API를 호출하여 방 입장 여부를 판단할 수 있으며 반환 값은 BOOL 유형입니다.
#### 함수 프로토타입  
```
ITMGContext abstract bool IsRoomEntered()
```
#### 샘플 코드
```
ITMGContext.GetInstance().IsRoomEntered();
```

### 방 퇴장
해당 API를 호출하여 방에서 퇴장할 수 있습니다. 이것은 비동기 API이며 반환 값은 AV_OK라면 비동기 전달이 성공했음을 의미합니다.

> 만약 응용프로그램 중 방 퇴장 후 즉시 입장하는 상황이 발생할 경우 API 호출 과정에서 개발자는 ExitRoom의 콜백 RoomExitComplete 알림을 기다릴 필요가 없으며 바로 API를 호출하면 됩니다.

#### 함수 프로토타입  
```
ITMGContext ExitRoom()
```
#### 샘플 코드  
```
ITMGContext.GetInstance().ExitRoom();
```

### 방 퇴장 콜백
콜백은 대리 기능으로 실행되어 사용자가 방에 퇴장한 후에 메시지를 전달합니다.
#### 함수 프로토타입  
```
위탁 함수:
public delegate void QAVExitRoomComplete();
이벤트 함수:
public abstract event QAVExitRoomComplete OnExitRoomCompleteEvent;
```
#### 샘플 코드
```
이벤트에 대해 수신:
ITMGContext.GetInstance().OnExitRoomCompleteEvent += new QAVExitRoomComplete(OnExitRoomComplete);
수신후의 처리:
void OnExitRoomComplete(){
    //방 퇴장 후의 처리
}
```

### 사용자 방 오디오 유형 획득
이 API는 사용자 방 오디오 유형을 획득하는 데 사용되며, 반환 값은 방 오디오 유형입니다. 반환 값이 0이면 사용자 방 오디오 유형에 오류가 발생하였음을 의미합니다. 방 오디오 유형은 EnterRoom API를 참조하십시오.

#### 함수 프로토타입  
```
ITMGContext ITMGRoom public  int GetRoomType()
```

#### 샘플 코드  
```
ITMGContext.GetInstance().GetRoom().GetRoomType();
```

### 사용자 방 오디오 유형 수정
해당 API는 사용자 방 오디오 유형을 수정하는 데 사용합니다.
#### 함수 프로토타입
```
ITMGContext ITMGRoom public void ChangeRoomType(ITMGRoomType roomtype)
```

|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| roomtype    |ITMGRoomType    |전환하려는 방 유형, 방 오디오 유형은 EnterRoom API를 참조하십시오.|

#### 샘플 코드
```
ITMGContext.GetInstance().GetRoom().ChangeRoomType(ITMG_ROOM_TYPE_FLUENCY);
```



### 방 오디오 유형 수정 콜백
방 유형을 자발적으로 설정합니다. 방 유형 설정 후 위탁을 통해 수정한 관련 정보를 전송합니다.

|리턴한 매개변수     | 의미  |
| ------------- |:-------------:|
| result    |0은 성공을 의미합니다.|
| error_info    |실패한 경우 관련 오류 정보를 전송합니다.|

```
위탁 함수:
public delegate void QAVOnChangeRoomtypeCallback(int result, string error_info);

이벤트 함수:
public abstract event QAVCallback OnChangeRoomtypeCallback;
```
#### 샘플 코드  
```
이벤트에 대해 수신:
ITMGContext.GetInstance().OnChangeRoomtypeCallback += new QAVOnChangeRoomtypeCallback(OnChangeRoomtypeCallback);
수신후의 처리:
void OnChangeRoomtypeCallback(){
    //방 유형 설정 완료 후의 처리
}
```

### 방 유형 변화 알림
사용자가 자발적으로 방 유형을 수정하거나 방 안의 다른 사용자가 방 유형을 수정하는 경우, 방 유형에 변화가 있으면 해당 콜백 함수는 호출되고 해당 콜백 함수를 통해 비즈니스 레이어에 방 유형 변화를 알리고 방 유형을 리턴합니다. EnterRoom API를 참조하십시오.
```
위탁 함수:
public delegate void QAVOnRoomTypeChangedEvent(int roomtype);

이벤트 함수:
public abstract event QAVOnRoomTypeChangedEvent OnRoomTypeChangedEvent;
```
#### 샘플 코드  
```
이벤트에 대해 수신:
ITMGContext.GetInstance().OnRoomTypeChangedEvent += new QAVOnRoomTypeChangedEvent(OnRoomTypeChangedEvent);
수신후의 처리:
void OnRoomTypeChangedEvent(){
    //방 유형 변경 후의 처리
}
```



### 멤버 상태 변화
이 이벤트는 상태 변화 시에만 알리며 상태 변화가 없을 경우 알리지 않습니다. 멤버 상태를 실시간으로 획득하려면 상위 레이어에서 알림을 수신하였을 때 캐시를 하십시오. 이벤트 메시지는 ITMG_MAIN_EVNET_TYPE_USER_UPDATE이고 매개변수 intent는 event_ID 및 user_list 두 개 정보를 포함하며 OnEvent 함수에서 이벤트 메시지를 판단합니다.
오디오 이벤트의 알림에는 임계값이 있습니다. 이 임계값을 초과해야 알림이 발송됩니다. 2초 이후에도 오디오 패킷을 수신하지 못해야 "어떤 멤버가 오디오 패킷 발송을 중지합니다."라는 메시지가 발송됩니다.

|event_id     | 의미         |관리 내용|
| ------------- |:-------------:|-------------|
|ITMG_EVENT_ID_USER_ENTER    				|멤버 방 입장			|앱에 멤버 리스트 관리		|
|ITMG_EVENT_ID_USER_EXIT    				|멤버 방 퇴장			|앱에 멤버 리스트 관리		|
|ITMG_EVENT_ID_USER_HAS_AUDIO    		|멤버 오디오 패킷 발송		|앱에 통화 멤버 리스트 관리	|
|ITMG_EVENT_ID_USER_NO_AUDIO    			|멤버 오디오 패킷 발송 중지	|앱에 통화 멤버 리스트 관리	|

#### 샘플 코드
```
위탁 함수:
public delegate void QAVEndpointsUpdateInfo(int eventID, int count, [MarshalAs(UnmanagedType.LPArray, SizeParamIndex = 1)]string[] openIdList);
이벤트 함수:
public abstract event QAVEndpointsUpdateInfo OnEndpointsUpdateInfoEvent;

이벤트에 대해 수신:
ITMGContext.GetInstance().OnEndpointsUpdateInfoEvent += new QAVEndpointsUpdateInfo(OnEndpointsUpdateInfo);
수신후의 처리:
void OnEndpointsUpdateInfo(int eventID, int count, string[] openIdList)
{
    //처리
		//개발자가 매개변수를 해석하여 메시지 event_ID 및 user_list 획득
		    switch (eventID)
 		    {
 		    case ITMG_EVENT_ID_USER_ENTER:
  			    //멤버 방 입장
  			    break;
 		    case ITMG_EVENT_ID_USER_EXIT:
  			    //멤버 방 퇴장
			    break;
		    case ITMG_EVENT_ID_USER_HAS_AUDIO:
			    //멤버 오디오 패킷 발송
			    break;
		    case ITMG_EVENT_ID_USER_NO_AUDIO:
			    //멤버 오디오 패킷 발송 중지
			    break;

		    default:
			    break;
 		    }
		break;
}

```


### 품질 모니터링 이벤트
품질 모니터링 이벤트, 이벤트 메시지는 ITMG_MAIN_EVENT_TYPE_CHANGE_ROOM_QUALITY이며, 반환한 매개변수는 weight, floss 및 delay이고 의미하는 정보는 다음과 같습니다. OnEvent 함수에서 이벤트 메시지에 대해 판단합니다.

|매개변수     | 의미         |
| ------------- |-------------|
|weight    				|범위는 1 ~ 50으로 50이면 음질이 매우 우수하며, 1이면 음질이 아주 떨어져 거의 사용할 수 없습니다. 0은 초기값으로 의미가 없습니다.|
|floss    				|패킷 손실률|
|delay    		|오디오 리치 지연 시간(ms)|


## 음성 채팅 오디오 API
SDK 초기화 후 방에 입장해야 방에서 실시간 오디오 음성 관련 API를 호출할 수 있습니다.
사용자가 인터페이스에서 마이크/스피커 켜기/끄기 버튼을 클릭할 때 다음 방식을 채택하길 권합니다.
- 대부분의 게임 앱의 경우, EnableMic 및 EnbaleSpeaker API를 호출하는 것을 권장합니다. 즉, EnableAudioCaptureDevice/EnableAudioSend 및 EnableAudioPlayDevice/EnableAudioRecv API를 동시에 호출해야 합니다.
- 소셜 앱과 같은 다른 유형의 모바일 앱은 캡처 장치를 시작/종료하면 전체 장치(캡처 및 재생)가 다시 시작됩니다. 이때 앱이 배경 음악을 재생 중이라면 그 음악은 재생이 중단됩니다. 업/다운스트림에 대한 제어를 통해 마이크 켜기/끄기를 진행하면 재생 장치가 중단되지 않습니다. 구체적인 호출 방식은 다음과 같습니다. 방에 입장할 때 EnableAudioCaptureDevice(true) && EnabledAudioPlayDevice(true)를 1회 호출하고, 마이크 켜기/끄기를 클릭할 때 EnableAudioSend/Recv만 호출하여 오디오 발송/수신 여부를 제어합니다.
- 단독으로 캡처를 릴리스하거나 장치를 재생하려면 API EnableAudioCaptureDevice 및 EnableAudioPlayDevice를 참조하십시오.
- Pause를 호출하여 오디오 엔진을 일시 중지하고 resume을 호출하여 오디오 엔진을 복원합니다.

### 소셜 앱 호출 프로세스도
![](https://main.qcloudimg.com/raw/53598680491501ab5a144e87ba932ccc.png)


|API     | API 의미   |
| ------------- |:-------------:|
|EnableMic    						|마이크 켜기/끄기|
|GetMicState    						|마이크 상태 획득|
|EnableAudioCaptureDevice    		|캡처 장치 켜기/끄기		|
|IsAudioCaptureDeviceEnabled    	|캡처 장치 상태 획득	|
|EnableAudioSend    				|오디오 업스트림 시작/종료	|
|IsAudioSendEnabled    				|오디오 업스트림 상태 획득	|
|GetMicLevel    						|실시간 마이크 음량 획득	|
|SetMicVolume    					|마이크 음량 설정		|
|GetMicVolume    					|마이크 음량 획득		|
|EnableSpeaker    					|스피커 켜기/끄기|
|GetSpeakerState    					|스피커 상태 획득|
|EnableAudioPlayDevice    			|재생 장치 켜기/끄기		|
|IsAudioPlayDeviceEnabled    		|재생 장치 상태 획득	|
|EnableAudioRecv    					|오디오 다운스트림 시작/종료	|
|IsAudioRecvEnabled    				|오디오 다운스트림 상태 획득	|
|GetSpeakerLevel    					|실시간 스피커 음량 획득	|
|SetSpeakerVolume    				|스피커 음량 설정		|
|GetSpeakerVolume    				|스피커 음량 획득		|
|EnableLoopBack    					|인이어 켜기/끄기			|



### 마이크 켜기/끄기
이 API는 마이크 켜기/끄기에 사용됩니다. 방 가입 시 기본적으로 마이크 및 스피커를 켜지 않습니다.
EnableMic = EnableAudioCaptureDevice + EnableAudioSend.
#### 함수 프로토타입  
```
ITMGAudioCtrl EnableMic(bool isEnabled)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| isEnabled    |boolean     |마이크를 켜야 하는 경우 매개변수 TRUE를 가져오고, 마이크를 꺼야 하는 경우 매개변수 FALSE를 가져옵니다.|

#### 샘플 코드  
```
마이크 켜기
ITMGContext.GetInstance().GetAudioCtrl().EnableMic(true);
```

### 마이크 상태 획득
이 API는 마이크 상태를 획득하는 데에 사용됩니다. 반환 값 0은 마이크가 꺼져 있음을 의미하며, 1은 마이크가 켜져 있음을 의미합니다.
#### 함수 프로토타입  
```
ITMGAudioCtrl GetMicState()
```
#### 샘플 코드  
```
micToggle.isOn = ITMGContext.GetInstance().GetAudioCtrl().GetMicState();
```

### 캡처 장치 켜기/끄기
해당 API는 캡처 장치 켜기/끄기에 사용합니다. 방 입장 시 기본적으로 장치를 켜지 않습니다.
- 방에 입장한 후에야 해당 API를 호출할 수 있고 방을 나가면 자동으로 장치를 끕니다.
- 모바일에서 캡처 장치 켜는 동시에 일반적으로 권한 신청, 음량 유형 조정 등을 조작이 수반됩니다.

#### 함수 프로토타입
```
ITMGAudioCtrl int EnableAudioPlayDevice(bool isEnabled)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| isEnabled    |bool     |캡처 장치를 켜야 하는 경우 매개변수 TRUE를 가져오고, 캡처 장치를 종료해야 하는 경우 FALSE를 가져옵니다.|

#### 샘플 코드  
```
캡처 장치 켜기
ITMGContext.GetInstance().GetAudioCtrl().EnableAudioCaptureDevice(true);
```

### 캡처 장치 상태 획득
이 API는 캡처 장치 상태 획득에 사용됩니다.

#### 함수 프로토타입
```
ITMGAudioCtrl bool IsAudioCaptureDeviceEnabled()
```
#### 샘플 코드

```
bool IsAudioCaptureDevice = ITMGContext.GetInstance().GetAudioCtrl().IsAudioCaptureDeviceEnabled();
```

### 오디오 업스트림 시작/종료
이 API는 오디오 업스트림 시작/종료에 사용됩니다. 캡처 장치가 이미 시작된 경우, 캡처한 오디오 데이터를 전송합니다. 캡처 장치가 시작되지 않은 경우 여전히 무성입니다. 캡처 장치의 켜기/끄기는 API EnableAudioCaptureDevice를 참조하십시오.

#### 함수 프로토타입

```
ITMGAudioCtrl int EnableAudioSend(bool isEnabled)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| isEnabled    |bool     |오디오 업스트림을 시작해야 하는 경우 매개변수 TRUE를 가져오고, 오디오 업스트림을 종료해야 하는 경우 매개변수 FALSE를 가져옵니다.|

#### 샘플 코드  

```
ITMGContext.GetInstance().GetAudioCtrl().EnableAudioSend(true);
```

### 오디오 업스트림 상태 획득
이 API는 오디오 업스트림 상태 획득에 사용됩니다.
#### 함수 프로토타입  

```
ITMGAudioCtrl bool IsAudioSendEnabled()
```
#### 샘플 코드
```
bool IsAudioSend = ITMGContext.GetInstance().GetAudioCtrl().IsAudioSendEnabled();
```

### 마이크 실시간 음량 획득
이 API는 마이크 실시간 음량 획득에 사용되며 반환 값은 int 유형입니다.
#### 함수 프로토타입
```
ITMGAudioCtrl -(int)GetMicLevel
```
#### 샘플 코드
```
ITMGContext.GetInstance().GetAudioCtrl().GetMicLevel();
```

### 마이크 음량 설정
이 API는 마이크 음량 설정에 사용됩니다. 매개변수 volume은 마이크의 음량 설정에 사용되며, 값이 0이면 무성임을 의미하고, 값이 100이면 음량이 커지지도 작아지지도 않음을 의미합니다. 기본값은 100입니다.

#### 함수 프로토타입
```
ITMGAudioCtrl SetMicVolume(int volume)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| volume    |int        |음량 설정, 범위: 0 ~ 200|

#### 샘플 코드  

```
int micVol = (int)(value * 100);
ITMGContext.GetInstance().GetAudioCtrl().SetMicVolume (micVol);
```

### 마이크 음량 획득
이 API는 마이크 음량 획득에 사용됩니다. 반환 값은 int 유형이며 반환 값 101은 API SetMicVolume을 호출한 적이 없음을 의미합니다.

#### 함수 프로토타입
```
ITMGAudioCtrl GetMicVolume()
```
#### 샘플 코드
```
ITMGContext.GetInstance().GetAudioCtrl().GetMicVolume();
```

### 스피커 켜기/끄기
이 API는 스피커 켜기/끄기에 사용됩니다.
EnableSpeaker = EnableAudioPlayDevice +  EnableAudioRecv.
#### 함수 프로토타입  
```
ITMGAudioCtrl EnableSpeaker(bool isEnabled)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| isEnabled    |bool        |스피커를 꺼야 하는 경우 매개변수 FALSE를 가져오고, 스피커를 켜야 하는 경우 매개변수 TRUE를 가져옵니다.|

#### 샘플 코드  
```
스피커 켜기
ITMGContext.GetInstance().GetAudioCtrl().EnableSpeaker(true);
```


### 스피커 상태 획득
이 API는 스피커 상태 획득에 사용됩니다. 반환 값 0은 스피커가 꺼져 있음을 의미하며, 반환 값 1은 스피커가 켜져 있음을 의미하며, 반환 값 2는 스피커가 실행 중임을 의미합니다.
#### 함수 프로토타입  
```
ITMGAudioCtrl GetSpeakerState()
```

#### 샘플 코드  
```
speakerToggle.isOn = ITMGContext.GetInstance().GetAudioCtrl().GetSpeakerState();
```


### 재생 장치 켜기/끄기
이 API는 재생 장치 켜기/끄기에 사용됩니다.
#### 함수 프로토타입  

```
ITMGAudioCtrl EnableAudioPlayDevice(bool isEnabled)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| isEnabled    |bool        |재생 장치를 꺼야 하는 경우 매개변수 FALSE를 가져오고, 재생 장치를 켜는 경우 매개변수 TRUE를 가져옵니다.|

#### 샘플 코드

```
재생 장치 켜기
ITMGContext.GetInstance().GetAudioCtrl().EnableAudioPlayDevice(true);
```


### 재생 장치 상태 획득
이 API는 재생 장치 상태 획득에 사용됩니다.
#### 함수 프로토타입

```
ITMGAudioCtrl bool IsAudioPlayDeviceEnabled()
```
#### 샘플 코드  

```
bool IsAudioPlayDevice = ITMGContext.GetInstance().GetAudioCtrl().IsAudioPlayDeviceEnabled();
```

### 오디오 다운스트림 시작/종료
이 API는 오디오 다운스트림 시작/종료에 사용됩니다. 재생 장치가 이미 켜져 있는 경우 방의 다른 사람의 오디오 데이터를 재생합니다. 재생 장치가 켜지지 않은 경우 여전히 무성입니다. 재생 장치의 켜기/끄기는 API EnableAudioPlayDevice를 참조하십시오.

#### 함수 프로토타입  

```
ITMGAudioCtrl int EnableAudioRecv(bool isEnabled)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| isEnabled    |bool     |오디오 다운스트림을 시작해야 하는 경우 매개변수 TRUE를 가져오고, 오디오 다운스트림을 종료해야 하는 경우 매개변수 FALSE를 가져옵니다.|

#### 샘플 코드  

```
ITMGContext.GetInstance().GetAudioCtrl().EnableAudioRecv(true);
```

### 오디오 다운스트림 상태 획득
이 API는 오디오 다운스트림 상태 획득에 사용됩니다.
#### 함수 프로토타입  

```
ITMGAudioCtrl bool IsAudioRecvEnabled()
```
#### 샘플 코드  

```
bool IsAudioRecv = ITMGContext.GetInstance().GetAudioCtrl().IsAudioRecvEnabled();
```


### 스피커 실시간 음량 획득
이 API는 스피커 실시간 음량 획득에 사용됩니다. 반환 값은 int 유형이며, 스피커 실시간 음량을 나타냅니다.
#### 함수 프로토타입
```
ITMGAudioCtrl GetSpeakerLevel()
```

#### 샘플 코드
```
ITMGContext.GetInstance().GetAudioCtrl().GetSpeakerLevel();
```

### 스피커 음량 설정
이 API는 스피커의 음량 설정에 사용됩니다.
매개변수 volume은 스피커의 음량 설정에 사용되며, 값이 0이면 무성임을, 값이 100이면 음량이 커지지도 작아지지도 않음을 의미합니다. 기본값은 100입니다.

#### 함수 프로토타입  
```
ITMGAudioCtrl SetSpeakerVolume(int volume)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| volume    |int        |음량 설정, 범위: 0 ~ 200|

#### 샘플 코드

```
int speVol = (int)(value * 100);
ITMGContext.GetInstance().GetAudioCtrl().SetSpeakerVolume(speVol);
```

### 스피커 음량 획득
이 API는 스피커 음량 획득에 사용됩니다. 반환 값은 int 유형이며 스피커의 음량을 의미합니다. 반환 값 101은 API SetSpeakerVolume을 호출한 적이 없음을 의미합니다.
Level은 실시간 음량이고, Volume은 스피커의 음량으로 최종 음량은 Level * Volume%입니다. 예: 실시간 음량 값이 100이고, Volume 값이 60이라면 최종 음량 값은 역시 60이 됩니다.

#### 함수 프로토타입
```
ITMGAudioCtrl GetSpeakerVolume()
```
#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioCtrl().GetSpeakerVolume();
```


### 인이어 시동
이 API는 인이어 시동에 사용됩니다.
#### 함수 프로토타입  

```
ITMGContext GetAudioCtrl EnableLoopBack(bool enable)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| enable    |bool         |시동 여부 설정|

#### 샘플 코드  

```
ITMGContext.GetInstance().GetAudioCtrl().EnableLoopBack(true);
```

### 장치 점용과 릴리스 이벤트 콜백
방에서 장치를 점용하거나 릴리스할 경우 콜백하며, 위탁을 통해 이벤트 관련 정보를 전송합니다.

```
위탁 함수:
public delegate void QAVOnDeviceStateChangedEvent(int deviceType, string deviceId, bool openOrClose);
이벤트 함수:
public abstract event QAVOnDeviceStateChangedEvent OnDeviceStateChangedEvent;
```

|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| deviceType    	|int       	|1은 캡처 장치, 2는 재생 장치를 가리킴							|
| deviceId   	 	|string 	|장치 GUID, 장치를 표시하기 위해 사용하고 Windows와 Mac에서만 유효	|
| openOrClose    |bool  	|캡처 장치/ 재생 장치 점용 또는 릴리스							|


|매개변수    |값         |의미|
| ------------- |:-------------:|-------------|
| AUDIODEVICE_CAPTURE    	|1       	|캡처 장치를 가리킴|
| AUDIODEVICE_PLAYER   	 	|2 			|재생 장치를 가리킴|

#### 샘플 코드  

```
이벤트에 대해 수신:
ITMGContext.GetInstance().GetAudioCtrl().OnDeviceStateChangedEvent += new QAVAudioDeviceStateCallback(OnAudioDeviceStateChange);
수신후의 처리:
void QAVAudioDeviceStateCallback(){
    //장치 점용과 릴리스 이벤트 관련 콜백 처리
}
```


## 음성 채팅 반주 관련 API
|API     | API 의미   |
| ------------- |:-------------:|
|StartAccompany    				       |반주 재생 시작|
|StopAccompany    				   	|반주 재생 중지|
|IsAccompanyPlayEnd				|반주 재생 완료 여부|
|PauseAccompany    					|반주 재생 일시 중지|
|ResumeAccompany					|반주 다시 재생|
|SetAccompanyVolume 				|반주 음량 설정|
|GetAccompanyVolume				|반주 재생 음량 획득|
|SetAccompanyFileCurrentPlayedTimeByMs 				|재생 진도 설정|

### 반주 재생 시작
이 API를 호출하여 반주 재생을 시작합니다. M4A, WAV, MP3 모두 3가지 형식을 지원합니다. 이 API를 호출하면 음량이 리셋됩니다.
#### 함수 프로토타입
```
IQAVAudioEffectCtrl int StartAccompany(string filePath, bool loopBack, int loopCount, int duckerTimeMs)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| filePath    |string            |반주 재생 경로|
| loopBack    |bool          |믹스하여 발송 여부, 일반적으로 TRUE로 설정하며 다른 사람도 반주를 들을 수 있습니다.|
| loopCount    |int          |순환 횟수, -1은 무한 순환을 의미합니다.|
| duckerTimeMs    |int             |페이드아웃 시간, 일반적으로 0을 입력합니다|

#### 샘플 코드
```
ITMGContext.GetInstance().GetAudioEffectCtrl().StartAccompany(filePath,true,loopCount,duckerTimeMs);
```

### 반주 재생 콜백
반주 재생이 종료 시의 콜백, 위탁을 통해 정보를 전송합니다.
#### 함수 프로토타입  
```
위탁 함수:
public delegate void QAVAccompanyFileCompleteHandler(int code, string filepath);
이벤트 함수:
public abstract event QAVAccompanyFileCompleteHandler OnAccompanyFileCompleteHandler;
```
#### 샘플 코드
```
이벤트에 대해 수신：
ITMGContext.GetInstance().GetAudioEffectCtrl().OnAccompanyFileCompleteHandler += new QAVAccompanyFileCompleteHandler(OnAccomponyFileCompleteHandler);
수신후의 처리:
void OnAccomponyFileCompleteHandler(int code, string filepath){
    //반주 재생 이벤트 콜백
}
```

### 반주 재생 중지
이 API를 호출하여 반주 재생을 중지합니다.
#### 함수 프로토타입
```
IQAVAudioEffectCtrl int StopAccompany(int duckerTimeMs)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| duckerTimeMs    |int             |페이드아웃 시간|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().StopAccompany(duckerTimeMs);
```

### 반주 재생 완료 여부
재생이 완료되면 반환 값은 TRUE이고, 재생이 완료되지 않으면 반환 값은 FALSE입니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl bool IsAccompanyPlayEnd();
```
#### 샘플 코드
```
ITMGContext.GetInstance().GetAudioEffectCtrl().IsAccompanyPlayEnd();
```


### 반주 재생 일시 중지
이 API를 호출하여 반주 재생을 일시 중지합니다.
#### 함수 프로토타입
```
IQAAudioEffectCtrl int PauseAccompany()
```
#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().PauseAccompany();
```



### 반주 다시 재생
이 API는 반주를 다시 재생하는 데 사용됩니다.
#### 함수 프로토타입
```
IQAAudioEffectCtrl int ResumeAccompany()
```
#### 샘플 코드
```
ITMGContext.GetInstance().GetAudioEffectCtrl().ResumeAccompany();
```

### 자신이 반주를 들을 수 있는지 여부 설정
이 API는 자신이 반주를 들을 수 있는지 여부 설정에 사용됩니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl int EnableAccompanyPlay(bool enable)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| enable    |bool             |들을 수 있는지 여부|

#### 샘플 코드
```
ITMGContext.GetInstance().GetAudioEffectCtrl().EnableAccompanyPlay(true);
```

### 다른 사람이 반주를 들을 수 있는지 여부 설정
다른 사람이 반주를 들을 수 있는지 여부 설정합니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl int EnableAccompanyLoopBack(bool enable)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| enable    |bool             |들을 수 있는지 여부|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().EnableAccompanyLoopBack(true);
```


### 반주 음량 설정
반주 음량을 설정하는 데 사용되고 가능한 범위는 0 ~200입니다. 기본값은 100입니다. 100보다 큰 값은 "볼륨 높이기"를 의미하고 100보다 작은 값은 "볼륨 줄이기"를 의미합니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl int SetAccompanyVolume(int vol)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| vol    |int             |음량 값|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().SetAccompanyVolume(vol);
```

### 반주 재생 음량 획득
이 API는 반주 음량 획득에 사용됩니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl abstract int GetAccompanyVolume()
```
#### 샘플 코드  
```
string currentVol = "VOL: " + ITMGContext.GetInstance().GetAudioEffectCtrl().GetAccompanyVolume();
```

### 반주 재생 진도 획득
다음 2개의 API는 반주 재생 진도 획득에 사용됩니다. 주의사항: Current / Total = 현재 순환 횟수, Current % Total = 현재 순환 재생 위치입니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl abstract uint GetAccompanyFileTotalTimeByMs()
IQAAudioEffectCtrl abstract int GetAccompanyFileCurrentPlayedTimeByMs()
```
#### 샘플 코드  
```
Sstring current = "Current: " + ITMGContext.GetInstance().GetAudioEffectCtrl().GetAccompanyFileCurrentPlayedTimeByMs() + " ms";
string total = "Total: " + ITMGContext.GetInstance().GetAudioEffectCtrl().GetAccompanyFileTotalTimeByMs() + " ms";
```


### 재생 진도 설정
이 API는 재생 진도 설정에 사용됩니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl abstract uint SetAccompanyFileCurrentPlayedTimeByMs(uint time)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| time    |uint                |재생 진도, 단위: 밀리초|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().SetAccompanyFileCurrentPlayedTimeByMs(time);
```


## 음성 채팅 효과음 관련 API
|API     | API 의미   |
| ------------- |:-------------:|
|PlayEffect    		|효과음 재생|
|PauseEffect    	|효과음 재생 일시 중지|
|PauseAllEffects	|모든 효과음 일시 중지|
|ResumeEffect    	|효과음 다시 재생|
|ResumeAllEffects	|모든 효과음 다시 재생|
|StopEffect 		|효과음 재생 중지|
|StopAllEffects		|모든 효과음 재생 중지|
|SetVoiceType 		|변성 효과|
|SetKaraokeType 		|노래방 효과음|
|GetEffectsVolume	|효과음 재생 음량 획득|
|SetEffectsVolume 	|효과음 재생 음량 설정|

### 효과음 재생
이 API는 효과음 재생에 사용됩니다. 매개변수에서의 효과음 ID는 앱에서 관리해야 하며, ID는 1차의 독립 재생 이벤트를 의미합니다. 후속적으로 이 ID에 의해 이번 재생을 제어할 수 있습니다. 파일은 M4A, WAV, MP3 모두 3가지 형식을 지원합니다.
#### 함수 프로토타입  

```
IQAAudioEffectCtrl int PlayEffect(int soundId, string filePath, bool loop = false, double pitch = 1.0f, double pan = 0.0f, double gain = 1.0f)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| soundId    	|int      	|효과음 ID|
| filePath    	|string 	|효과음 경로|
| loop    		|bool   	|중복 재생 여부|
| pitch    	|double	|필드 보류|
| pan    		|double	|필드 보류|
| gain    		|double	|필드 보류|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().PlayEffect(soundId,filePath,true,1.0,0,1.0);
```

### 효과음 재생 일시 중지
이 API는 효과음 재생 일시 중지에 사용됩니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl int PauseEffect(int soundId)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| soundID    |int                    |효과음 ID|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().PauseEffect(soundId);
```

### 모든 효과음 일시 중지
이 API를 호출하여 모든 효과음을 일시 중지합니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl int PauseAllEffects()
```
#### 샘플 코드
```
ITMGContext.GetInstance().GetAudioEffectCtrl().PauseAllEffects();
```

### 효과음 다시 재생
이 API는 효과음 다시 재생에 사용됩니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl int ResumeEffect(int soundId)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| soundID    |int                    |효과음 ID|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().ResumeEffect(soundId);
```



### 모든 효과음 다시 재생
이 API를 호출하여 모든 효과음을 다시 재생합니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl int ResumeAllEffects()
```
#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().ResumeAllEffects();
```

### 효과음 재생 중지
이 API는 효과음 재생 중지에 사용됩니다.
#### 함수 프로토타입
```
IQAAudioEffectCtrl int StopEffect(int soundId)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| soundID    |int                    |효과음 ID|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().StopEffect(soundId);
```

### 모든 효과음 재생 중지
이 API를 호출하여 모든 효과음 재생을 중지합니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl int StopAllEffects()
```
#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().StopAllEffects();
```

### 변성 효과
이 API를 호출하여 변성 효과를 설정합니다.
#### 함수 프로토타입
```
IQAAudioEffectCtrl int setVoiceType(int type)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| type    |int                    |로컬 오디오 변성 유형|



|유형 매개변수     |매개변수 표시 값|의미|
| ------------- |-------------|------------- |
| ITMG_VOICE_TYPE_ORIGINAL_SOUND  		|0	|오리지날			|
| ITMG_VOICE_TYPE_LOLITA    				|1	|여자애			|
| ITMG_VOICE_TYPE_UNCLE  				|2	|아저씨			|
| ITMG_VOICE_TYPE_INTANGIBLE    			|3	|고유함			|
| ITMG_VOICE_TYPE_DEAD_FATBOY  			|4	|뚱뚱보			|
| ITMG_VOICE_TYPE_HEAVY_MENTA			|5	|헤비 멘탈			|
| ITMG_VOICE_TYPE_DIALECT 				|6	|외국인			|
| ITMG_VOICE_TYPE_INFLUENZA 				|7	|독감			|
| ITMG_VOICE_TYPE_CAGED_ANIMAL 			|8	|야수			|
| ITMG_VOICE_TYPE_HEAVY_MACHINE		|9	|무거운 기계			|
| ITMG_VOICE_TYPE_STRONG_CURRENT		|10	|강한 전류			|
| ITMG_VOICE_TYPE_KINDER_GARTEN			|11	|유치원생			|
| ITMG_VOICE_TYPE_HUANG 					|12	|미니언즈			|


#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().setVoiceType(0);
```

### 노래방 효과음
이 API를 호출하여 노래방 효과음을 설정합니다.
####  함수 프로토타입  
```
IQAAudioEffectCtrl int SetKaraokeType(int type)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| type    |int                    |로컬 오디오 변성 유형|


|유형 매개변수     |매개변수 표시 값|의미|
| ------------- |-------------|------------- |
|ITMG_KARAOKE_TYPE_ORIGINAL 		|0	|오리지날			|
|ITMG_KARAOKE_TYPE_POP 				|1	|팝			|
|ITMG_KARAOKE_TYPE_ROCK 			|2	|락			|
|ITMG_KARAOKE_TYPE_RB 				|3	|힙합			|
|ITMG_KARAOKE_TYPE_DANCE 			|4	|댄스곡			|
|ITMG_KARAOKE_TYPE_HEAVEN 			|5	|고유함			|
|ITMG_KARAOKE_TYPE_TTS 				|6	|음성 합성		|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().SetKaraokeType(0);
```




### 효과음 재생 음량 획득
효과음 재생 음량을 획득하는 데 사용되고 선형 음량입니다. 기본값은 100입니다. 100보다 큰 값은 "볼륨 높이기"를 의미하고 100보다 작은 값은 "볼륨 줄이기"를 의미합니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl  int GetEffectsVolume()
```
#### 샘플 코드  
```
ITMGContext.GetInstance().GetAudioEffectCtrl().GetEffectsVolume();
```


### 효과음 재생 음량 설정
이 API를 호출하여 효과음 재생 음량을 설정합니다.
#### 함수 프로토타입  
```
IQAAudioEffectCtrl  int SetEffectsVolume(int volume)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| volume    |int                    |음량 값|

#### 샘플 코드
```
ITMGContext.GetInstance().GetAudioEffectCtrl().SetEffectsVolume(volume);
```







## 오프라인 음성
오프라인 음성 및 문자 전환 기능을 사용하려면 우선 SDK를 초기화해야 합니다.

|API     | API 의미   |
| ------------- |:-------------:|
|ApplyPTTAuthbuffer    |인증 초기화	|
|SetMaxMessageLength    |최대 음성 메시지 시간 제한	|
|StartRecording		|녹음 시작		|
|StartRecordingWithStreamingRecognition		|스트리밍 녹음 시작		|
|StopRecording    	|녹음 중지		|
|CancelRecording	|녹음 취소		|
|GetMicLevel |오프라인 음성의 실시간 마이크 음량 획득	|
|GetSpeakerLevel | 오프라인 음성의 실시간 스피커 음량 획득	|
|UploadRecordedFile 	|음성 파일 업로드		|
|DownloadRecordedFile	|음성 파일 다운로드		|
|PlayRecordedFile 	|음성 재생		|
|StopPlayFile		|음성 재생 중지		|
|GetFileSize 		|음성 파일 크기		|
|GetVoiceFileDuration	|음성 파일 시간		|
|SpeechToText 		|음성을 문자로 인식			|

### 인증 초기화
SDK 초기화 후 인증 초기화를 호출하여 authBuffer를 획득하려면 상기 음성 채팅 인증 정보 API를 참조하십시오.
#### 함수 프로토타입  
```
ITMGPTT int ApplyPTTAuthbuffer (byte[] authBuffer)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| authBuffer    |byte[]                   |인증|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetPttCtrl().ApplyPTTAuthbuffer(authBuffer);
```

### 최대 음성 메시지 시간 제한
최대 음성 메시지 길이를 60초로 제한합니다.
#### 함수 프로토타입
```
ITMGPTT int SetMaxMessageLength(int msTime)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| msTime    |int                    |음성 시간, 단위: 밀리초|

#### 샘플 코드  

```
ITMGContext.GetInstance().GetPttCtrl().SetMaxMessageLength(60000);
```


### 녹음 시작
이 API는 녹음 시작에 사용됩니다. 녹음 파일을 업로드해야 음성을 문자로 전환 등 조작을 진행할 수 있습니다.
#### 함수 프로토타입  
```
ITMGPTT int StartRecording(string fileDir)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| fileDir    |string                      |음성 저장 경로|

#### 샘플 코드
```
string recordPath = Application.persistentDataPath + string.Format ("/{0}.silk", sUid++);
int ret = ITMGContext.GetInstance().GetPttCtrl().StartRecording(recordPath);
```

### 녹음 시작 콜백
녹음 완료 콜백, 위탁을 통해 정보를 전달합니다.
#### 함수 프로토타입  
```
위탁 함수:
public delegate void QAVRecordFileCompleteCallback(int code, string filepath);
이벤트 함수:
public abstract event QAVRecordFileCompleteCallback OnRecordFileComplete;
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| code    |string                      |code가 0일 경우 녹음 완료|
| filepath    |string                      |녹음 저장 주소|

#### 샘플 코드  
```
이벤트에 대해 수신:
ITMGContext.GetInstance().GetPttCtrl().OnRecordFileComplete += mInnerHandler;
수신후의 처리:
void mInnerHandler(int code, string filepath){
    //녹음 시작 콜백
}
```

### 스트리밍 음성 인식 시작
이 API는 스트리밍 음성 인식 시작에 사용되며, 동시에 콜백 과정에서 실시간으로 음성을 문자로 전환하여 반환할 수 있으며, 언어를 지정하여 인식할 수 있고 인식한 정보를 지정한 언어로 번역하여 반환할 수도 있습니다.

#### 함수 프로토타입  

```
ITMGPTT int StartRecordingWithStreamingRecognition(string filePath)
ITMGPTT int StartRecordingWithStreamingRecognition(string filePath, string speechLanguage,string translateLanguage)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| filePath    	|String	|음성 저장 경로	|
| speechLanguage    |String                     |String	|지정한 문자로 식별하는 언어 매개변수, 세부 정보는 [음성을 문자로 전환하는 언어 매개변수 참고리스트](https://cloud.tencent.com/document/product/607/30282)를 참조하십시오.|
| translateLanguage    |String                     |String	|지정한 문자로 번역하는 언어 매개변수, 세부 정보는 [음성을 문자로 전환하는 언어 매개변수 참고리스트](https://cloud.tencent.com/document/product/607/30282)를 참조하십시오. (이 매개변수는 잠시 사용 불가하고 speechLanguage와 동일한 매개변수를 입력해 주십시오)|

#### 샘플 코드  
```
string recordPath = Application.persistentDataPath + string.Format("/{0}.silk", sUid++);
int ret = ITMGContext.GetInstance().GetPttCtrl().StartRecordingWithStreamingRecognition(recordPath, "cmn-Hans-CN", "cmn-Hans-CN");
```

### 스트리밍 음성 인식 시작의 콜백
스트리밍 음성 인식을 시작 및 완료한 후의 콜백은 위탁을 통해 정보를 전달합니다.
```
위탁 함수:
public delegate void QAVStreamingRecognitionCallback(int code, string fileid, string filepath, string result);
이벤트 함수:
public abstract event QAVStreamingRecognitionCallback OnStreamingSpeechComplete;
```

|메시지 이름     | 의미         |
| ------------- |:-------------:|
| code    	|스트리밍 음성 인식 성공 여부를 판단하는 반환 코드			|
| result    		|음성을 문자로 전환하여 인식한 텍스트	|
| filepath 	|녹음 저장 로컬 주소		|
| fileid 		|녹음의 백 엔드 URL 주소	|

|오류 코드     | 의미         |처리 방식|
| ------------- |:-------------:|:-------------:|
|32775	|스트리밍 음성을 문자로 전환에 실패했지만, 녹음이 성공했습니다.	|UploadRecordedFile API를 호출하여 녹음을 업로드하고 SpeechToText API를 호출하여 음성을 문자로 전환합니다.
|32777	|스트리밍 음성을 문자로 전환에 실패했지만, 녹음이 성공했고, 업로드도 성공했습니다.	|반환한 정보에 업로드가 성공한 백 엔드 URL 주소가 있으며, SpeechToText API를 호출하여 음성을 문자로 전환합니다.

#### 샘플 코드  
```
이벤트에 대해 수신:
ITMGContext.GetInstance().GetPttCtrl().OnStreamingSpeechComplete += mInnerHandler;
수신후의 처리:
void mInnerHandler(int code, string fileid, string filepath, string result){
    //스트리밍 음성 인식 시작의 콜백
}

```
### 녹음 중지
이 API는 녹음을 중지하는 데 사용되며 비동기 API입니다. 녹음 후 콜백이 반환됩니다. 녹음이 완료되면 녹음 파일을 사용할 수 있습니다.
#### 함수 프로토타입  
```
ITMGPTT int StopRecording()
```
#### 샘플 코드
```
ITMGContext.GetInstance().GetPttCtrl().StopRecording();
```

### 녹음 취소
이 API를 호출하여 녹음을 취소합니다. 취소 후 콜백이 없습니다.
#### 함수 프로토타입  

```
IQAVPTT int CancelRecording()
```
#### 샘플 코드  

```
ITMGContext.GetInstance().GetPttCtrl().CancelRecording();
```

### 오프라인 음성 마이크 실시간 음량 획득
이 API는 마이크 실시간 음량 획득에 사용되며 반환 값은 int 유형입니다. 값 범위는 0 ~ 100입니다.

#### 함수 프로토타입  
```
IQAVPTT int GetMicLevel()
```
#### 샘플 코드  
```
ITMGContext.GetInstance().GetPttCtrl().GetMicLevel();
```


### 스피커 실시간 음량 획득
이 API는 스피커 실시간 음량 획득에 사용되며 반환 값은 int 유형입니다. 값 범위는 0 ~ 100입니다.

#### 함수 프로토타입  
```
IQAVPTT int GetSpeakerLevel()
```

#### 샘플 코드  
```
ITMGContext.GetInstance().GetPttCtrl().GetSpeakerLevel();
```




### 음성 파일 업로드
이 API는 음성 파일울 업로드하는 데 사용됩니다.
#### 함수 프로토타입  

```
IQAVPTT int UploadRecordedFile (string filePath)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| filePath    |string                      |음성 업로드 경로|

#### 샘플 코드

```
ITMGContext.GetInstance().GetPttCtrl().UploadRecordedFile(filePath);
```


### 음성 업로드 완료 콜백
음성 업로드 완료 콜백, 위탁을 통해 정보를 전달합니다.
#### 함수 프로토타입
```
위탁 함수:
public delegate void QAVUploadFileCompleteCallback(int code, string filepath, string fileid);
이벤트 함수:
public abstract event QAVUploadFileCompleteCallback OnUploadFileComplete;
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| code    |int                       |code가 0일 경우 녹음 완료|
| filepath    |string                      |녹음 저장 주소|
| fileid    |string                      |파일의 url 경로|

#### 샘플 코드
```
이벤트에 대해 수신:
ITMGContext.GetInstance().GetPttCtrl().OnUploadFileComplete += mInnerHandler;
수신후의 처리:
void mInnerHandler(int code, string filepath, string fileid){
    //음성 업로드 완료 콜백
}
```


### 음성 파일 다운로드
이 API는 음성 파일을 다운로드하는 데 사용됩니다.
#### 함수 프로토타입  

```
IQAVPTT DownloadRecordedFile (string fileID, string downloadFilePath)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| fileID    |string                       |파일의 url 경로|
| downloadFilePath    |string                       |파일의 로컬 저장 경로|

#### 샘플 코드

```
ITMGContext.GetInstance().GetPttCtrl().DownloadRecordedFile(fileId, filePath);
```


### 음성 파일의 다운로드가 완료되면 콜백이 실행됩니다.
음성 파일의 다운로드가 완료되면 대리 기능을 사용하여 콜백을 실행하여 메시지를 전달합니다.
#### 함수 프로토타입  
```
위탁 함수:
public delegate void QAVDownloadFileCompleteCallback(int code, string filepath, string fileid);
이벤트 함수:
public abstract event QAVDownloadFileCompleteCallback OnDownloadFileComplete;
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| code    |int                       |code가 0일 경우 녹음 완료|
| filepath    |string                      |녹음 저장 주소|
| fileid    |string                      |파일의 url 경로|

#### 샘플 코드  
```
이벤트에 대해 수신:
ITMGContext.GetInstance().GetPttCtrl().OnDownloadFileComplete += mInnerHandler;
수신후의 처리:
void mInnerHandler(int code, string filepath, string fileid){
    //음성 파일의 다운로드가 완료되면 콜백이 실행됩니다.
}
```



### 음성 재생
이 API는 음성을 재생하는 데 사용됩니다.
#### 함수 프로토타입
```
IQAVPTT PlayRecordedFile (string downloadFilePath)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| downloadFilePath    |string                       |파일의 경로|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetPttCtrl().PlayRecordedFile(filePath);
```


### 음성 재생 콜백
음성 재생 콜백, 위탁을 통해 정보를 전달합니다.
#### 함수 프로토타입  
```
위탁 함수:
public delegate void QAVPlayFileCompleteCallback(int code, string filepath);
이벤트 함수:
public abstract event QAVPlayFileCompleteCallback OnPlayFileComplete;
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| code    |int                       |code가 0일 경우 녹음 완료|
| filepath    |string                      |녹음 저장 주소|

#### 샘플 코드  
```
이벤트에 대해 수신:
ITMGContext.GetInstance().GetPttCtrl().OnPlayFileComplete += mInnerHandler;
수신후의 처리:
void mInnerHandler(int code, string filepath){
    //음성 재생 콜백
}
```




### 음성 재생 중지
이 API는 음성 재생 중지에 사용됩니다.
#### 함수 프로토타입  
```
IQAVPTT int StopPlayFile()
```

#### 샘플 코드  
```
ITMGContext.GetInstance().GetPttCtrl().StopPlayFile();
```



### 음성 파일 크기 획득
이 API를 통해 음성 파일 크기를 획득할 수 있습니다.
#### 함수 프로토타입  
```
IQAVPTT GetFileSize(string filePath)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| filePath    |string                      |음성 파일의 경로|

#### 샘플 코드  
```
int fileSize = ITMGContext.GetInstance().GetPttCtrl().GetFileSize(filepath);
```

### 음성 파일 시간 획득
이 API는 음성 파일 시간 획득에 사용되며 단위는 밀리초입니다.
#### 함수 프로토타입  
```
IQAVPTT int GetVoiceFileDuration(string filePath)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| filePath    |string                      |음성 파일의 경로|

#### 샘플 코드  
```
int fileDuration = ITMGContext.GetInstance().GetPttCtrl().GetVoiceFileDuration(filepath);
```


### 지정한 음성 파일을 문자로 식별
이 API는 지정한 음성 파일을 문자로 식별하는 데 사용됩니다.

#### 함수 프로토타입  

```
IQAVPTT int SpeechToText(String fileID)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| fileID    |string                      |음성 파일 URL|

#### 샘플 코드  
```
ITMGContext.GetInstance().GetPttCtrl().SpeechToText(fileID);
```

### 지정한 음성 파일을 문자로 번역(지정한 언어)
이 API는 지정한 음성 파일을 지정한 언어의 문자로 번역하는 데 사용됩니다.

####  함수 프로토타입  
```
IQAVPTT int SpeechToText(String fileID,String speechLanguage,String translateLanguage)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| fileID    |String                     |음성 파일 URL|
| speechLanguage    |String                     |지정한 문자로 식별하는 언어 매개변수, 세부 정보는 [음성을 문자로 전환하는 언어 매개변수 참고 리스트](https://cloud.tencent.com/document/product/607/30282)를 참조하십시오.|
| translateLanguage	|String	|지정한 문자로 번역하는 언어 매개변수, 세부 정보는 [음성을 문자로 전환하는 언어 매개변수 참고리스트](https://cloud.tencent.com/document/product/607/30282)를 참조하십시오(이 매개변수는 잠시 사용 불가하고 speechLanguage와 동일한 매개변수를 입력해 주십시오).|

####  샘플 코드  
```
ITMGContext.GetInstance().GetPttCtrl().SpeechToText(fileID,"cmn-Hans-CN","cmn-Hans-CN");
```



### 식별 콜백
지정 음성 파일을 문자로 식별하고 위탁을 통해 정보를 전송합니다.
#### 함수 프로토타입  
```
위탁 함수:
public delegate void QAVSpeechToTextCallback(int code, string fileid, string result);
이벤트 함수:
public abstract event QAVSpeechToTextCallback OnSpeechToTextComplete;
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| code    |int                       |code가 0일 경우 녹음 완료|
| fileid    |string                      |음성 파일 URL	|
| result    |string                      |전환한 텍스트 결과|

#### 샘플 코드
```
이벤트에 대해 수신:
ITMGContext.GetInstance().GetPttCtrl().OnSpeechToTextComplete += mInnerHandler;
수신후의 처리:
void mInnerHandler(int code, string fileid, string result){
    //식별 콜백
}
```
## 고급 API
### 버전 번호 획득
SDK 버전 번호 획득, 분석에 사용됩니다.
#### 함수 프로토타입
```
ITMGContext  abstract string GetSDKVersion()
```
#### 샘플 코드  
```
ITMGContext.GetInstance().GetSDKVersion();
```





### 로그 인쇄 등급 설정
로그 인쇄 등급 설정에 사용됩니다. 기본 등급 유지를 권장합니다.
#### 함수 프로토타입
```
ITMGContext  SetLogLevel(ITMG_LOG_LEVEL levelWrite, ITMG_LOG_LEVEL levelPrint)
```

#### 매개변수 의미

|매개변수|유형|의미|
|---|---|---|
|levelWrite|ITMG_LOG_LEVEL| 쓰기 로그의 등급 설정, TMG_LOG_LEVEL_NONE은 쓰지 않음을 의미합니다. 기본값은 TMG_LOG_LEVEL_INFO입니다|
|levelPrint|ITMG_LOG_LEVEL| 인쇄 로그의 등급 설정, TMG_LOG_LEVEL_NONE은 인쇄하지 않음을 의미합니다. 기본값은 TMG_LOG_LEVEL_ERROR입니다|



|ITMG_LOG_LEVEL|의미|
| -------------------------------|:-------------:|
|TMG_LOG_LEVEL_NONE=0		|로그 인쇄하지 않음			|
|TMG_LOG_LEVEL_ERROR=1		|오류 로그 인쇄(기본값)	|
|TMG_LOG_LEVEL_INFO=2			|알림 로그 인쇄		|
|TMG_LOG_LEVEL_DEBUG=3		|개발 디버깅 로그 인쇄	|
|TMG_LOG_LEVEL_VERBOSE=4		|고빈도 로그 인쇄		|

#### 샘플 코드  
```
ITMGContext.GetInstance().SetLogLevel(TMG_LOG_LEVEL_NONE,true,true);
```

### 로그 인쇄 경로 설정
로그 인쇄 경로 설정에 사용됩니다.
기본 경로:

|플랫폼     |경로        |
| ------------- |-------------|
|Windows 	|%appdata%\Tencent\GME\ProcessName|
|iOS    		|Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Documents|
|Android	|/sdcard/Android/data/xxx.xxx.xxx/files|
|Mac    		|/Users/username/Library/Containers/xxx.xxx.xxx/Data/Documents|

#### 함수 프로토타입
```
ITMGContext  SetLogPath(string logDir)
```

|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------
| logDir    		|NSString   		|경로|

#### 샘플 코드  
```
ITMGContext.GetInstance().SetLogPath(path);
```
### 진단 정보 획득
실시간 영상/음성 통화 품질 관련 정보를 획득합니다. 해당 API는 주로 실시간 통화 품질을 확인하고 문제점을 해결하는 데 사용되므로 비즈니스 측에서는 무시할 수 있습니다.
#### 함수 프로토타입  
```
IQAVRoom GetQualityTips()
```
#### 샘플 코드  
```
string tips = ITMGContext.GetInstance().GetRoom().GetQualityTips();
```

### 오디오 데이터 블랙리스트에 추가
어느 ID 하나를 오디오 데이터 블랙리스트에 추가합니다. 반환 값이 0이면 호출 성공을 의미합니다.
#### 함수 프로토타입  

```
ITMGContext ITMGAudioCtrl AddAudioBlackList(string openId)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| openId    |NSString      |블랙리스트에 추가할 ID|

#### 샘플 코드  

```
ITMGContext.GetInstance().GetAudioCtrl ().AddAudioBlackList (id);
```

### 오디오 데이터 블랙리스트에서 제거
어느 ID 하나를 오디오 데이터 블랙리스트에서 제거합니다. 반환 값이 0이면 호출 성공을 의미합니다.
#### 함수 프로토타입  

```
ITMGContext ITMGAudioCtrl RemoveAudioBlackList(string openId)
```
|매개변수     | 유형         |의미|
| ------------- |:-------------:|-------------|
| openId    |NSString      |블랙리스트에서 제거할 ID|

#### 샘플 코드  

```
ITMGContext.GetInstance().GetAudioCtrl ().RemoveAudioBlackList (id);
```
