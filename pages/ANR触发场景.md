## Service Timeout（服务的超时）
	- 对于前台服务，则超时为SERVICE_TIMEOUT = 20s；
	- 对于后台服务，则超时为SERVICE_BACKGROUND_TIMEOUT = 200s
- ## BroadcastQueue Timeout（广播的超时）
	- 对于**前台广播**，则超时为BROADCAST_FG_TIMEOUT =  ==**10s**==；
	- 对于后台广播，则超时为BROADCAST_BG_TIMEOUT = 60s;
- ## ContentProvider Timeout（内容提供者的超时）
	- ContentProvider超时为CONTENT_PROVIDER_PUBLISH_TIMEOUT = 10s;
- ## InputDispatching Timeout（输入事件的响应）
	- InputDispatching Timeout: 输入事件分发超[[#red]]==**时5s**==，包括按键和触摸事件。
	- >Input的超时机制与其他的不同，对于input来说即便某次事件执行时间超过timeout时长，只要用
	  户后续在没有再生成输入事件，则不会触发ANR