## MMM-Hotword
MMM-Hotword is a simple snowboy hotword detector.
You can use this module to wake another voice assistant or to give a command to other module.

### Screenshot
This works in background, so there is no screenshot.

### Installation

1. Install pre-dependencies
```sh
sudo apt-get install libmagic-dev libatlas-base-dev
sudo apt-get install sox libsox-fmt-all
```
1. Install Module
```sh
git clone https://github.com/eouia/MMM-Hotword.git
cd MMM-Hotword
npm install
```
```sh
npm install --save snowboy   # Sometimes it could fail, retry again.
npm install --save-dev electron-rebuild
./node_modules/.bin/electron-rebuild   # It could takes dozens sec.
```

### Configuration
Below values are pre-set as default values. It means, you can put even nothing in config field.
```javascript
{
  module: 'MMM-Hotword',
  config: {
    snowboy: [
      {
				hotwords: "SMARTMIRROR", //this will be sended to other module for distinguishing which hotword is detected.
				file: "resources/models/smart_mirror.umdl",
				sensitivity: '0.5',
			},
			{
				hotwords: "SNOWBOY",
				file: "resources/models/snowboy.umdl",
				sensitivity: '0.5',
			},
			{
				file: 'resources/models/jarvis.umdl',
				sensitivity: '0.8,0.80',
				hotwords: ['jarvis','jarvis'] //Kitt.ai changed their Jarvis UMDL, it has 2 models in one file. So weird.
				//anyway, you can give different name for each. ['jarvis_1', 'jarvis_2']. Even though I think this is useless.
			}
    ],
    record: {
      sampleRate    : 16000,      // audio sample rate
      threshold     : 0.5,        // silence threshold (rec only)
      thresholdStart: null,       // silence threshold to start recording, overrides threshold (rec only)
      thresholdEnd  : null,       // silence threshold to end recording, overrides threshold (rec only)
      silence       : 1.0,        // seconds of silence before ending
      verbose       : false,      // log info to the console
      recordProgram : 'arecord',  // Defaults to 'arecord' - also supports 'rec' and 'sox'
      device        : null        // recording device (e.g.: 'plughw:1')
    },
    autostart: true              // if 'false', this module will wait for 'HOTWORD_RESUME' notification to start hotwords detection at the beginning.
    autorestart: false          // You can set this 'true' when you want this module to go back to listening mode automatically again after hotword is detected. But use this carefully when your other modules are using microphone or speaker.
  }
},

```

If you want to use default configuration, just use like this. and it's enough.
(With this empty configuration, you can use 'snowboy', 'smart mirror', 'jarvis' as hotwords.)

```
{
  module: "MMM-Hotword",
  config: {}
},
```

### Usage

#### 1. Commands
Other modules can order this module to start/stop by notifications.

|Notification| payload| description
|---|---|---|
|HOTWORD_RESUME | null | Let this module try to listen sounds until detecting hotwords or interrupted.
|HOTWORD_PAUSE | null | Let this module stop to listen sounds.

#### 2. Results
This module might broadcast some notification as results.

|Notification|payload|description
|---|---|---|
|HOTWORD_ERROR | error | When error is occurred.
|HOTWORD_LISTENING | null | When this module start listening. (normally instant answer for the 'HOTWORD_RESUME')
|HOTWORD_SLEEPING | null | When this module stop listening. (normally instant answer for the 'HOTWORD_PAUSE')
|HOTWORD_DETECTED | {index: `int`, hotword: `string`} | When a hotword is detected, this notification will be casted. <br> In default configuration values;<br>{1, 'SMARTMIRROR'}<br>{2, 'SNOWBOY'}<br>{3, 'JARVIS'}<br>{4, 'JARVIS'}

#### 3. Common usage flow
1. If `autostart` is set as 'true', Your mirror is ready to catch hotwords after the mirror is on. (See 5. for `autostart` is 'false')
2. Say the hotword. (e.g; 'Jarvis' or 'Smart mirror')
3. If detection is success, `HOTWORD_DETECTED`notification will be broadcasted with `{index:n, hotword:'something'}`.
4. Now, you can make your other modules receive that notification and do something.
5. After `HOTWORD_DETECTED` notification is sent, `MMM-Hotword` stops listening hotword(unless `autorestart` is `true`), so, your other modules should reactivate `MMM-HOTWORD` with `HOTWORD_RESUME` notification. (If your module uses microphone, should release mic for hotword listening before notification)
6. Don't set `autorestart` as 'true' when you combine this with other voice related modules. (But if you have 2 mics, It could be OK.)

### Tip.
- You can combine this module and [MMM-NotificationTrigger](https://github.com/eouia/MMM-NotificationTrigger) to make voice commander. (But you need so many .umdl or .pmdl to make commands)
- You can use this module with [MMM-AssistantMk2](https://github.com/eouia/MMM-AssistantMk2) to make embeded Google Assistant. 