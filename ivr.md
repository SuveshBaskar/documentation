## IVR

#### Default Voice Options
```js
app.CALL_FORWARD_DDI = "0028373644";
app.DEFAULT_VOICE_OPTIONS = {
	"tts_engine": "wavenet",
	"text_type": "ssml",
	"speed": 0.8,
	"disconnect": false, // Disconnects the Call - true
}
```

#### Publish Message
```js
app.publish([
	{ 
		"message": "Hello, I am transferring your call",  
		...app.DEFAULT_VOICE_OPTIONS,
		"forward_num": app.CALL_FORWARD_DDI, 
		"redirect": true,
	}
]);
```


#### Blank Message Handler

```js
if (!app.data.message || !app.data.message.trim()) {

    app.memory.get("disconnect-on-silence")
        .then(() => {
            app.memory.delete("disconnect-on-silence");
            app.publish([{ message: `Thank you for your valuble time.`, disconnect: true }])
            return Promise.resolve();
        })
        .catch(() => { /* DO NOTHING */ })

    app.memory.get('response-message')
        .then((message) => {
            app.memory.set("disconnect-on-silence", message);
            app.memory.delete('response-message');
            app.publish([{ message: `We are waiting for your response. ${message}`, disconnect: false }])
            return Promise.resolve();
        })
        .catch(() => {
            app.publish([{ message: "Can you kindly repeat that?", disconnect: false }])
            return Promise.resolve();
        })
} else {
    app.memory.delete("disconnect-on-silence");
}
```


#### Custom Bot Start Options

```js
app.BotStartOptions = {
    minConfidence: 0.9,
    excludeParamsForSwitching: ['step'],
}
```

#### Start Bot with context cleared

```js
app.StartBotDeleteContext = () => {
    return app.getContext().then(context => {
        app.clearContext();
        return app.start(app.BotStartOptions)

    }).catch(() => {
        return app.start(app.BotStartOptions)
    })
}
```

#### Start

```js
if (app.data.message && app.data.message == "welcome") {
 	return app.StartBotDeleteContext();
} else if (app.data.message && app.data.message == "not_answered") {
    app.publish([{ message: "<speak>All our lines are busy right now<break/>Please try again after some time<break/>Thank you for calling Domino's</speak>", "tts_engine": "wavenet", speed: 0.8, "text_type": "ssml", disconnect: true, "hangup_string": "normal" }])
    return Promise.resolve();
} else {
    return app.start(app.BotStartOptions)
}
```

### Send Text Message

```js
app.sendTextMessage(
	"<speak>Hi I am still learning<break/>I can help you with your order status</speak>", 
	{ ...app.DEFAULT_VOICE_OPTIONS, "disconnect": false },
).then(() => resolve());
```

```js
let responseMessage = `<speak>Dear Customer</speak>`;
await app.memory.set('response-message', responseMessage);
await app.memory.set('disconnect-on-silence', "true"); // This will Disconnect the call if there is no user response
await app.sendTextMessage(responseMessage, {...app.DEFAULT_VOICE_OPTIONS})
resolve();
```