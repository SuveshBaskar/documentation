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
##### Static Message
```js
app.sendTextMessage(
	"<speak>Hi I am still learning<break/>I can help you with your order status</speak>", 
	{ ...app.DEFAULT_VOICE_OPTIONS, "disconnect": false },
).then(() => resolve());
```

##### Store Message for Future Usage
```js
let responseMessage = `<speak>Dear Customer</speak>`;
await app.memory.set('response-message', responseMessage);
await app.memory.set('disconnect-on-silence', "true"); // This will Disconnect the call if there is no user response
await app.sendTextMessage(responseMessage, {...app.DEFAULT_VOICE_OPTIONS})
resolve();
```

#### Multilingual 
```js
const message = app.renderMessage('passport-status-003', {}, 'There was an error in checking your application status  Kindly try again later or visit our website  web.umang.gov.in');

await app.sendTextMessage(
    message,
    {
        ...app.VOICE_OPTIONS,
        capture_dtmf: true,
        disconnect: true,
    }
)
```
### Message Helper
```js
return new Promise((resolve) => {
    const { message, capture_dtmf = false, disconnect = false } = args;
    if (app.source == "voice") {
        app.publish([{
            message,
            ...app.VOICE_OPTIONS,
            capture_dtmf,
            disconnect,
        }])
    } else {
        app.sendTextMessage(message);
    }
    resolve();
});
```

### Invalid Count
```js
let invalidCount = 0;
try {
    invalidCount = await app.memory.get("validator-invalid-count-001");
} catch (error) {/* DO NOTHING */}

if (app.source == "voice") {

    let message = app.renderMessage('ivr-mobile-validator-001', {}, 'Sorry I was not able to understand that can you please repeat that, a little bit slowly this time'),
        disconnect = false;

    if (invalidCount > 1) {
        message = app.renderMessage('ivr-mobile-validator-001', {}, 'Sorry I am not able to help you right now. Please connect later or visit our website');
        disconnect = true;
    } else if (invalidCount == 1) {
        message = app.renderMessage('ivr-mobile-validator-002', {}, 'Apologies can you please use your dial pad to input the OTP');
    }
    app.executeFunction("messageHelper", { message, disconnect, capture_dtmf: true });
} else {
    if (invalidCount >= 2) {
        await app.sendTextMessage(app.renderMessage('web-otp-validator-003', {}, 'You <strong>exceeded</strong> the OTP verification. Kindly try again later or visit our website.'))
        return app.triggerIntent("default")
    }
    await app.sendTextMessage(app.renderMessage('web-otp-validator-004', {}, 'Apologies, Can you please enter a <strong>valid</strong> OTP?'))
}
await app.memory.set("validator-invalid-count-001", ++invalidCount);
```

### Remove SSML Tags
```js
const originalString = `
<speak>Apologies<break/> can you please use your dial pad <break/>to input your Date of birth</speak>
`
const strippedString = originalString.replace(/(<([^>]+)>)/ig,"");
console.log(strippedString)
```