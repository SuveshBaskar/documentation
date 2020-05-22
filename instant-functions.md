# Instant Functions

Direct copy paste functions that will work like magic

### Mobile Prompt - Validation

##### Mobile Prompt

```js
return new Promise(async (resolve) => {
    const { changemobile } = app.context.steps;
    let options = ["⏪ Back"];
    options = options.map(option => ({title: option, text: option}));
    if (changemobile) {
        await app.setStep('otp', undefined);
        await app.sendQuickReplies({
            title: "Can you please enter your primary contact number 📱 to verify the OTP?",
            options,
        });
        return resolve();
    } else if (app.source === "yellowmessenger") {
        await app.sendQuickReplies({
            title: 'We will be happy to serve you, Please enter your primary contact number📱 to continue?',
            options,
        });
        return resolve();
    } 
});
```


##### Mobile Validator
```js

app.MAX_MOBILE_LENGTH = 12;
app.MOBILE_PATTERN = /(\+91|91|0)?([6789]\d{9})/;

return new Promise(async (resolve) => {
    const { history, steps, intent } = app.context;
    const step = app.context.paramExpected;
    const { message } = app.data;

    const backPattern = /\b(back)\b/gi;
    if (message.match(backPattern)) {
        const previousSteps = history[history.length - 1].steps;
        delete previousSteps[step]
        return app.triggerIntent(intent, {}, {...previousSteps});
    }

    let mobile;
    if ((message && message.match(/\d/) && message.match(/\d/).join("").length <= app.MAX_MOBILE_LENGTH) && (message.match(app.MOBILE_PATTERN) && message.match(app.MOBILE_PATTERN)[2])) {
        mobile = message.match(app.MOBILE_PATTERN)[2];
    }

    if (mobile) {
        await app.setStep(step, mobile);
        resolve();
    } else {
        await app.sendTextMessage("Please enter a valid contact number to continue?");
        resolve({ success: false })
    }
});
```

##### OTP Prompt
```js
return new Promise(async (resolve) => {
    const { mobile } = app.context.steps;
    try {
        await app.sendOtp(mobile);
    } catch (error) {
        await app.sendTextMessage("There is a problem in reaching out to our Service. Can you kindly try again after some time?");
        return app.triggerIntent("default");
    }
    let options = ["Change Mobile Number","Resend OTP"];
    options = options.map(option => ({title: option, text: option}));
    app.sendQuickReplies({ 
        title: `Can you tell me the One-Time Password (OTP) sent on your mobile number <strong>${mobile}</strong>?`,
        options,
    });
    resolve();
});
```

##### OTP Validator
```js

app.OTP_INVALID_ATTEMPTS_THRESHOLD = 2; // n+1 attempts - 3 Attempts Max

return new Promise(async (resolve) => {
    const { message = "" } = app.data;

    const step = app.context.paramExpected;
    const { mobile } = app.context.steps;

    let authenticated = false;
    let invalidAttempts = {
        "resend_count": 0,
        "invalid_otp_count": 0,
        "change_mobile_count": 0
    };

    try {
        let rawInvalidAttempts = await app.memory.get('invalid-attempts');
        invalidAttempts = typeof rawInvalidAttempts === "string" ? JSON.parse(rawInvalidAttempts) : rawInvalidAttempts;
    } catch (error) { /* DO NOTHING */ }

    const invalidExceedHelper = async () => {
        try{
            await app.memory.delete('invalid-attempts');
        } catch(error){/* DO NOTHING */}
        await app.sendTextMessage("You have exceeded the invalid attempts. I am taking you to the main menu.");
        return app.triggerIntent('default');
    }

    const verifyResponse = await app.verifyOtp(message);
    
    if (verifyResponse && verifyResponse.success) {
        authenticated = true;
    }

    if (authenticated) {
        try{
            await app.memory.delete('invalid-attempts');
        } catch(error){/* DO NOTHING */}
        await app.setStep(step, "authenticated");
        return resolve();
    } else {
        if (message && message.match(/\b(resend otp|resend)\b/gi)) {
            ++invalidAttempts["resend_count"];
            if (invalidAttempts["resend_count"] > app.OTP_INVALID_ATTEMPTS_THRESHOLD) {
                invalidExceedHelper();
            } else {
                await app.memory.set('invalid-attempts', invalidAttempts);
                await app.executeFunction("otpPrompt")
                resolve({ success: false });
            }
        } else if (message && message.match(/\b(change mobile|change)\b/gi)) {
            ++invalidAttempts["change_mobile_count"];
            if (invalidAttempts["change_mobile_count"] > app.OTP_INVALID_ATTEMPTS_THRESHOLD) {
                invalidExceedHelper();
            } else {
                await app.memory.set('invalid-attempts', invalidAttempts);
                // setSteps instead of set Multiple steps - Some issue in setting steps undefinded
                await app.setStep(step, undefined); 
                await app.setStep("mobile", undefined);
                await app.setStep("changemobile", true)
                resolve();
            }
        } else {
            ++invalidAttempts["invalid_otp_count"];
            if (invalidAttempts["invalid_otp_count"] > app.OTP_INVALID_ATTEMPTS_THRESHOLD) {
                invalidExceedHelper();
            } else {
                await app.memory.set('invalid-attempts', invalidAttempts);
                await app.sendTextMessage("Can you please check the received OTP and enter again?");
                resolve({ success: false });
            }
        }
    }
});
```


### Extract details for a Sentence
```js
let data = "I need 5 bananas and 2kg of oranges, also add 500gms of Banana too";
const fruitPatternCollection = {
    "Banana": /\b(bananas|banana)\b/gi,
    "Orange": /\b(oranges|orange)\b/gi, 
}
const unitPatternCollection = {
    "KGs": /kg/gi,
    "Grams": /grams|gram|gms|gm|grm/gi,
}
const itemsArray = Object.keys(fruitPatternCollection)
const unitsArray = Object.keys(unitPatternCollection)
const productsArray = [
    ...Object.keys(fruitPatternCollection),
    ...Object.keys(unitPatternCollection),
]
const numberRegex = /[-]?[0-9]+[,.]?[0-9]*([\/][0-9]+[,.]?[0-9]*)*/g
const numberRegexString = "[-]?[0-9]+[,.]?[0-9]*([\/][0-9]+[,.]?[0-9]*)*"
const removePattern = new RegExp("^((?!("+ productsArray.join("|")+ "|" + numberRegexString +")).)*$","gmi");
const replaceInput = (data, object) => {
    for (let key in object) {
        data = data.replace(object[key], " " + key + " ").trim().replace(/\s{2,}/g," ")
    }
    return data;
}
data = replaceInput(data, fruitPatternCollection)
data = replaceInput(data, unitPatternCollection)
data = data
        .replace(/\s+/g,"\n")
        .replace(removePattern,"")
        .replace(/\n/g," ")
        .replace(/\s{2,}/g," ")
        .trim()
        .replace(new RegExp("("+itemsArray.join("|")+")","gi"),"$1 $#7884")
        .split("$#7884")
        .filter(Boolean)
        .map(value => value.trim());
data = data
        .map(value => {
            let result = {};
            result["product"] = value.match(new RegExp(itemsArray.join("|")))
                                ? value.match(new RegExp(itemsArray.join("|")))[0]
                                : null
            result["unit"] = value.match(new RegExp(unitsArray.join("|")))
                                ? value.match(new RegExp(unitsArray.join("|")))[0]
                                : null
            result["quantity"] = value.match(numberRegex)
                                    ? value.match(numberRegex)[0]
                                    : null
            return result;
        })
console.log(data);
```