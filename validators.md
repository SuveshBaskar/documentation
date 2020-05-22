### Parsing Helper
```js
try{
    typeof result.body === "string" ? JSON.parse(result.body) : result.body;
} catch(error){/* DO NOTHING */}
```

### Get Numbers from Predtiction
```js
const number = app.prediction.ordinals
    ? app.prediction.ordinals[0].value
    : app.prediction.numbers
        ? app.prediction.numbers[0].value
        : 0;
```

### Get Numbers only for OTP Validation - IVR
```js
const otpPattern = /\d{6}/;

const otp = message.match(otpPattern)
    ? message.match(otpPattern)[0]
    : message.match(/\d/g)
        ? message.match(/\d/g).join("").match(otpPattern)
            ? message.match(/\d/g).join("").match(otpPattern)[0]
            : 0
        : 0
```

### Invalid Count - For Handling Multiple Invalid Counts
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

### Get State and City

```shell
curl --location --request GET 'https://maps.googleapis.com/maps/api/geocode/json?address={{{MESSAGE}}}&key={{{API_KEY}}}&region=in'
```

```js
return new Promise(async (resolve) => {
    const { message } = args;
    const response = await app.executeApi("maps", { address: message })
    const body = typeof response.body === "string" ? JSON.parse(response.body) : response.body;
    const components = app._.get(body, "results[0].address_components", false)
    let city, state;
    if (components) {
        // City Check
        for (let value of components) {
            if (value.types && value.types.includes("administrative_area_level_2")) {
                city = value.long_name;
                break;
            }
        }
        // State Check
        for (let value of components) {
            if (value.types && value.types.includes("administrative_area_level_1")) {
                state = value.long_name;
                break;
            }
        }
    }
    resolve({city,state})
});
```


#### StopWords Helper
```js
return new Promise(resolve => {
    let { message } = args
    const stopWords =[
        'a', 'at', 'be', 'can', 'cant', 'could', 'couldnt', 'do', 'does', 'how', 'i', 'in', 'is', 'many', 'much', 'of',
        'on', 'or', 'should', 'shouldnt', 'so', 'such', 'the', 'them', 'they', 'to', 'us', 'we', 'what', 'who', 'why',
        'with', 'wont', 'would', 'wouldnt', 'you', 'my', 'name', 'is', 'you', 'can', 'call', 'me', 'I', 'am', 'yeah',
        'sure', 'check', 'blood', 'bank', 'availability', 'help', 'from'
    ];
    message =  message
                .replace(new RegExp('\\b(' + stopWords.join("|") + ')\\b', 'gi'), ' ')
                .replace(/\s{2,}/g, ' ')
                .trim();
    resolve(message);
});
```

#### Blood Type Helper
```js
return new Promise(resolve => {
    const bloods = ['AB', 'B', 'A', 'O'] // Add More Blood types if required
    const typesPattern = {
        '+Ve': ['\\+ve', '\\+', '\\s+\\+ve', '\\s+positive'],
        '-Ve': ['\\-ve', '\\-', '\\s+\\-ve', '\\s+negative'],
    }
    let bloodGroupPattern = {};
    bloods
        .forEach(blood => {
            let types = Object.keys(typesPattern);
            types
                .forEach(type => {
                    let bloodType = `${blood}${type}`
                    if (!bloodGroupPattern[bloodType]) {
                        bloodGroupPattern[bloodType] = {};
                    }
                    let patternArray = [];
                    typesPattern[type]
                        .forEach(value => {
                            patternArray.push(`${blood}${value}`.toLowerCase());
                        })
                    bloodGroupPattern[bloodType] = new RegExp(patternArray.join("|"), "gi");
                })
        })
    resolve(bloodGroupPattern);
});
```